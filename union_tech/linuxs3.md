                                                           
● 根据内核代码，我来整理 Linux S3 (suspend to RAM / mem) 的执行流程：                                                                                                                     
                                    
  Linux S3 (Suspend to RAM) 执行流程                                                                                                                                                      
                  
  用户空间: echo mem > /sys/power/state                                                                                                                                                   
                ↓                                                                                                                                                                         
            pm_suspend(PM_SUSPEND_MEM)          [kernel/power/suspend.c:636]                                                                                                              
                ↓                                                                                                                                                                         
            enter_state(state)                  [kernel/power/suspend.c:576]
                ↓

  1. 准备阶段

  ┌─────────────────────────────────────────────────────────────────┐
  │ 1.1 获取锁: mutex_lock(&system_transition_mutex)               │
  │                                                                 │
  │ 1.2 文件系统同步 (可选):                                       │
  │     pm_sleep_fs_sync() → ksys_sync()                           │
  │                                                                 │
  │ 1.3 suspend_prepare()                                         │
  │     ├─ pm_prepare_console()          切换到suspend控制台       │
  │     ├─ pm_notifier_call_chain()      通知订阅者准备suspend     │
  │     ├─ filesystems_freeze()          冻结文件系统             │
  │     └─ suspend_freeze_processes()    冻结进程                 │
  │         ├─ freeze_processes()        冻结用户空间进程         │
  │         │   ├─ __usermodehelper_disable(UMH_FREEZING)         │
  │         │   ├─ pm_freezing = true                              │
  │         │   └─ try_to_freeze_tasks(true) 遍历进程，调用freeze │
  │         └─ freeze_kernel_threads()    冻结内核线程            │
  │             └─ try_to_freeze_tasks(false)                     │
  └─────────────────────────────────────────────────────────────────┘

  2. 设备挂起阶段

  suspend_devices_and_enter()              [kernel/power/suspend.c:504]
      │
      ├─ platform_suspend_begin()         平台相关准备
      ├─ console_suspend_all()            暂停所有控制台
      │
      └─ dpm_suspend_start(PMSG_SUSPEND)  [drivers/base/power/main.c:2272]
          │
          ├─ dpm_prepare()                准备阶段
          │   └─ device_prepare() → 调用 dev->driver->pm->prepare()
          │
          └─ dpm_suspend()                挂起阶段
              └─ device_suspend() → 调用 dev->driver->pm->suspend()

  3. 深度挂起阶段

  suspend_enter()                         [kernel/power/suspend.c:419]
      │
      ├─ platform_suspend_prepare()       平台准备
      │
      ├─ dpm_suspend_late(PMSG_SUSPEND)   [drivers/base/power/main.c:1719]
      │   └─ device_suspend_late() → 调用 suspend_late 回调
      │       (此时运行时PM被禁用)
      │
      ├─ platform_suspend_prepare_late()  平台late准备
      │
      ├─ dpm_suspend_noirq(PMSG_SUSPEND)  [drivers/base/power/main.c:1588]
      │   ├─ suspend_device_irqs()        禁用所有设备中断
      │   └─ device_suspend_noirq() → 调用 suspend_noirq 回调
      │
      └─ platform_suspend_prepare_noirq() 平台noirq准备

  4. 进入S3 (核心)

      │
      ├─ pm_sleep_disable_secondary_cpus()  关闭非启动CPU
      │
      ├─ arch_suspend_disable_irqs()        禁用本地中断
      │
      ├─ system_state = SYSTEM_SUSPEND
      │
      ├─ syscore_suspend()                 [drivers/base/syscore.c:47]
      │   └─ 挂起系统核心组件 (timekeeping, 定时器等)
      │
      ├─ 检查wakeup: pm_wakeup_pending()
      │
      └─ suspend_ops->enter(state)         ⭐ 最终进入S3低功耗状态
              │                             调用ACPI或其他平台代码
              ▼                             执行HLT/WRT等指令进入睡眠
           ══════════════════════════════════════════════
                                                       (这里CPU停止，内存保持供电)
                                                       (等待唤醒事件...)
           ══════════════════════════════════════════════
              │                             唤醒后从这里继续执行

  5. 唤醒恢复 (逆序)

      │ (被唤醒事件中断)
      ▼
      syscore_resume()                     恢复系统核心
      ├─ arch_suspend_enable_irqs()        恢复中断
      ├─ pm_sleep_enable_secondary_cpus()  恢复其他CPU
      │
      ├─ dpm_resume_noirq()                恢复noirq设备
      ├─ dpm_resume_early()                恢复early设备
      ├─ dpm_resume()                      恢复所有设备
      │
      └─ suspend_finish()
          ├─ suspend_thaw_processes()      解冻进程
          ├─ filesystems_thaw()            解冻文件系统
          └─ pm_restore_console()          恢复控制台

  关键点总结

  ┌────────────────────┬────────────────────────────────────────┬──────────┐
  │        阶段        │                  作用                  │ 内存状态 │
  ├────────────────────┼────────────────────────────────────────┼──────────┤
  │ freeze_processes   │ 冻结所有用户空间进程和内核线程         │ 保持     │
  ├────────────────────┼────────────────────────────────────────┼──────────┤
  │ dpm_suspend        │ 调用设备suspend回调                    │ 保持     │
  ├────────────────────┼────────────────────────────────────────┼──────────┤
  │ dpm_suspend_late   │ 调用设备suspend_late回调，禁用运行时PM │ 保持     │
  ├────────────────────┼────────────────────────────────────────┼──────────┤
  │ dpm_suspend_noirq  │ 禁用设备中断，调用suspend_noirq        │ 保持     │
  ├────────────────────┼────────────────────────────────────────┼──────────┤
  │ syscore_suspend    │ 挂起时间管理等核心组件                 │ 保持     │
  ├────────────────────┼────────────────────────────────────────┼──────────┤
  │ suspend_ops->enter │ CPU进入低功耗，内存保持供电            │ 保持     │
  └────────────────────┴────────────────────────────────────────┴──────────┘

  S3的特点：内存内容保持，只需恢复CPU执行即可，恢复速度快。

