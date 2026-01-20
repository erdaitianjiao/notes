### 4.19 s4过程分析

**休眠中的acpi操作结构体**

```c
static const struct platform_hibernation_ops *hibernation_ops;
```

定义

```c
struct platform_hibernation_ops {
    int (*begin)(pm_message_t stage);
    void (*end)(void);
    int (*pre_snapshot)(void);
    void (*finish)(void);
    int (*prepare)(void);
    int (*enter)(void);
    void (*leave)(void);
    int (*pre_restore)(void);
    void (*restore_cleanup)(void);
    void (*recover)(void);
};
```

赋值

```C
static const struct platform_hibernation_ops acpi_hibernation_ops = { 
    .begin = acpi_hibernation_begin,
    .end = acpi_pm_end,
    .pre_snapshot = acpi_pm_prepare,
    .finish = acpi_pm_finish,
    .prepare = acpi_pm_prepare,
    .enter = acpi_hibernation_enter,
    .leave = acpi_hibernation_leave,
    .pre_restore = acpi_pm_freeze,
    .restore_cleanup = acpi_pm_thaw,
};
```

#### 函数调用过程 

- 内核入口函数

```c
static ssize_t state_store(struct kobject *kobj, struct kobj_attribute *attr,
               const char *buf, size_t n)
{
    suspend_state_t state;
    int error;

    error = pm_autosleep_lock();		// 检查电源管理的锁
    if (error)
        return error;

    if (pm_autosleep_state() > PM_SUSPEND_ON) {
        error = -EBUSY;
        goto out;
    }   

    state = decode_state(buf, n); 
    if (state < PM_SUSPEND_MAX) {
        if (state == PM_SUSPEND_MEM)
            state = mem_sleep_current;

        error = pm_suspend(state);
    } else if (state == PM_SUSPEND_MAX) {
        error = hibernate();				// s4的流程入口
    } else {
        error = -EINVAL;
    }   

 out:
    pm_autosleep_unlock();
    return error ? error : n;
}

```

s4 流程函数

```c
int hibernate(void)
{
    int error, nr_calls = 0; 
    bool snapshot_test = false;

    if (!hibernation_available()) {					// 检查是否支持睡眠
        pm_pr_dbg("Hibernation not available.\n");
        return -EPERM;
    }    

    lock_system_sleep();
    /* The snapshot device should not be opened while we're running */
    if (!atomic_add_unless(&snapshot_device_available, -1, 0)) {
        error = -EBUSY;
        goto Unlock;
    }    

    pr_info("hibernation entry\n");
    pm_prepare_console();							// 切换控制台
    error = __pm_notifier_call_chain(PM_HIBERNATION_PREPARE, -1, &nr_calls);
    if (error) {
        nr_calls--;
        goto Exit;
    }    

    pr_info("Syncing filesystems ... \n");
    ksys_sync();									// sync同步
    pr_info("done.\n");

    error = freeze_processes();						// 停止所有用户态线程和内核线程
    if (error)
        goto Exit;

    lock_device_hotplug();
    /* Allocate memory management structures */
    error = create_basic_memory_bitmaps();			// 分配内存页面位图，标记哪些内存是需要备份的
    if (error)
        goto Thaw;

    error = hibernation_snapshot(hibernation_mode == HIBERNATION_PLATFORM);		// 快照保存，
    if (error || freezer_test_done)
        goto Free_bitmaps;

    if (in_suspend) {
        unsigned int flags = 0; 

        if (hibernation_mode == HIBERNATION_PLATFORM)
            flags |= SF_PLATFORM_MODE;
        if (nocompress)
            flags |= SF_NOCOMPRESS_MODE;
        else 
                flags |= SF_CRC32_MODE;

        pm_pr_dbg("Writing image.\n");
        error = swsusp_write(flags);											// 将内存快照写入swap分区
        swsusp_free();
        if (!error) {
            if (hibernation_mode == HIBERNATION_TEST_RESUME)
                snapshot_test = true;
            else 
                power_down();								// 镜像已经安全存入磁盘后最从切断电源
        }    
        in_suspend = 0; 
        pm_restore_gfp_mask();
    } else {
        pm_pr_dbg("Image restored successfully.\n");
    }    

 Free_bitmaps:
    free_basic_memory_bitmaps();
 Thaw:
    unlock_device_hotplug();
    if (snapshot_test) {
        pm_pr_dbg("Checking hibernation image\n");
        error = swsusp_check();
        if (!error)
            error = load_image_and_restore();
    }    
    thaw_processes();										// 解冻所有进程，让程序恢复运行

    /* Don't bother checking whether freezer_test_done is true */
    freezer_test_done = false;
 Exit:
    __pm_notifier_call_chain(PM_POST_HIBERNATION, nr_calls, NULL);
    pm_restore_console();
    atomic_inc(&snapshot_device_available);
 Unlock:                            
    unlock_system_sleep();
    pr_info("hibernation exit\n");

    return error;
}

```

power_down

```c
static void power_down(void)
{
#ifdef CONFIG_SUSPEND
    int error;

    if (hibernation_mode == HIBERNATION_SUSPEND) {
        error = suspend_devices_and_enter(PM_SUSPEND_MEM);			
        if (error) {
            hibernation_mode = hibernation_ops ?
                        HIBERNATION_PLATFORM : 
                        HIBERNATION_SHUTDOWN;
        } else {
            /* Restore swap signature. */
            error = swsusp_unmark();
            if (error)
                pr_err("Swap will be unusable! Try swapon -a.\n");

            return;
        }    
    } 
#endif

    switch (hibernation_mode) {
    case HIBERNATION_REBOOT:
        kernel_restart(NULL);
        break;
    case HIBERNATION_PLATFORM:
        hibernation_platform_enter();		// 进入S4
        /* Fall through */
    case HIBERNATION_SHUTDOWN:
        if (pm_power_off)
            kernel_power_off();
        break;
    }    
    kernel_halt();
    /*   
     * Valid image is on the disk, if we continue we risk serious data
     * corruption after resume.
     */
    pr_crit("Power down manually\n");
    while (1)
        cpu_relax();
}
```

