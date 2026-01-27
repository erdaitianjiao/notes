## linux电源管理 s4分析报告 (基于linux4.19)

### 一、简介

#### 1. 什么是s4

电源管理中的S4 挂载到磁盘(也称休眠)

睡眠状态会将机器状态保存到交换空间，就是将内存中的内容存入交换空间、并完全关闭机器电源。机器开机后，状态会恢复。在此之前，功耗为零

#### 2. ACPI (高级配置和电源接口)

ACPI首先可以理解为一个独立于体系结构的电源管理和配置框架，它在主机OS中形成一个子系统。该框架建立一个硬件寄存器集来定义电源状态(休眠、hibernate、唤醒等)。硬件寄存器集可以容纳专用硬件和通用硬件上的操作

标准ACPI框架和硬件寄存器集的主要目的是启用电源管理和系统配置，无需操作系统来直接调用固件。ACPI作为系统固件(BIOS]和OS之间的接口层	

##### 2.1 协同机制

Linux S4 流程并非纯粹的软件行为，而是一场由操作系统（OS）主导、平台固件（ACPI BIOS）配合的协同演习。内核通过 `struct platform_hibernation_ops` 这一抽象接口，在关键的时间节点调用平台特定的钩子函数，确保软件状态如内存镜像与硬件状态的一致性

##### 2.2 ACPI控制方法介绍

在acpi规范中，为了配合操作系统进行睡眠状态，定义了一些控制方法，这些方法在bios的acpi表中

| 阶段     | ACPI 方法 | Linux 对应钩子           | 说明                                 |
| -------- | --------- | ------------------------ | ------------------------------------ |
| 准备进入 | _PTS      | `platform_begin`         | Prepare To Sleep。通知固件即将睡眠。 |
| 快照前   | _GTS      | `platform_pre_snapshot`  | Going To Sleep。设置硬件唤醒使能。   |
| 下电前   | _S4       | `acpi_hibernation_enter` | 进入 S4 对象。执行硬件寄存器写入。   |
| 唤醒后   | _WAK      | `platform_finish`        | Wakeup。通知固件系统已回到正常状态   |

#### 3. 用户态接口

```bash
# 这里主要以reboot为例
echo reboot > /sys/power/disk
echo disk > /sys/power/status
```

### 二、代码流程分析

#### 1. 重要数据结构                                                                  

```c
 // 实例化
static const struct platform_hibernation_ops *hibernation_ops;
// 这个实例的初始化是在 drivers/acpi/sleep.c 中的 acpi_sleep_init 模块初始化中赋值的
hibernation_set_ops(old_suspend_ordering ?
             &acpi_hibernation_ops_old : &acpi_hibernation_ops);

// 定义
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

// ACPI 休眠操作实例定义
static const struct platform_hibernation_ops acpi_hibernation_ops = {
    // 准备阶段 通知 BIOS 即将开始休眠流程
    .begin = acpi_hibernation_begin, 
    // 结束阶段 整个休眠或恢复流程完成后的清理工作
    .end = acpi_pm_end, 
    // 快照前准备 关闭硬件事件响应，防止拍摄内存快照时受到干扰
    .pre_snapshot = acpi_pm_prepare, 
    // 流程终点 如果是恢复失败或正常唤醒，通知硬件回到工作状态
    .finish = acpi_pm_finish, 
    // 执行前准备 在正式断电前，最后一次确认硬件处于可关闭状态
    .prepare = acpi_pm_prepare, 
    // 正式休眠 向硬件寄存器写入指令，直接触发物理断电
    .enter = acpi_hibernation_enter, 
    // 离开休眠 硬件上电后，最初步的恢复处理
    .leave = acpi_hibernation_leave,
    // 镜像恢复前处理 在把磁盘数据读回内存前，先冻结硬件事务
    .pre_restore = acpi_pm_freeze, 
    // 镜像恢复后处理 数据加载完成后，解冻并恢复硬件正常运行
    .restore_cleanup = acpi_pm_thaw,
};
```



#### 2. 函数调用流程

##### 2.1 睡眠调用

整个睡眠调用大概可以分为三个阶段，第一个是准备阶段，第二个是拍摄镜像阶段，第三个是写入阶段，第四个是关机阶段

###### 1) 进入S4的核心链路

```c
hibernate() {
	...
	ksys_sync();						// 同步磁盘
	freeze_processes();					// 冻结用户线程
	create_basic_memory_bitmaps();		// 准备阶段
	...
	hibernation_snapshot();				// 拍摄快照
	...
	swsusp_write();						// 写入快照
	...
	power_down();						// 执行关机或者挂起命令
}
```

###### 2) 拍摄快照阶段调用

拍摄快照的时候需要停止一切活动，防止对内存的再次修改，但是在停止活动之前要预先分配内存才存放这个镜像，提前把存储快照所需的空间预留出来，不然一旦系统静止后再发现内存不足，将没有任何机制能释放空间，导致逻辑死锁。

```c
hibernation_snapshot() {
	platform_begin();					// 与ACPI固件的交互 _PTS调用
	hibernate_preallocate_memory();		// 预申请内存
	freeze_kernel_threads();			// 冻结内核线程
	...									// 创建快照的准备部分
	create_image();						// 执行拷贝动作
	...
	platform_end();						// 恢复系统的读写能力 为写入磁盘做准备
	return;
}
```

###### 3) 创建快照调用

在create_image内部，系统经历了一个从静止到运行的转折

先是继续为拍摄镜像做一个静止的环境 在拍摄完快照之后就恢复运行状态，因为接下来需要执行写入镜像和执行关机或者休眠，所以得回到运行状态才能正常写入和休眠

```c
create_image() {
	platform_pre_snapshot();		// 调用ACPI的 _GTS方法 
	disable_nonboot_cpus();			// 关闭CPU0以外的所有多核
	local_irq_disable();			// 关闭中断
    syscore_suspend();				// 挂起系统内核的核心设备
    save_processor_state();			// 保存特殊寄存器的状态
    ...
	swsusp_arch_suspend(); 			// 拍摄快照 恢复状态的时候会从这里返回
    ...
    restore_processor_state();		// 恢复寄存器	
    platform_leave();				
    syscore_resume();				// 重新激活中断控制器
    local_irq_enable();				// 开启中断
    enable_nonboot_cpus();			// 唤醒其他cpu
    platform_finish();				// 后续可继续访问cpu核心 
}
```

###### 4) 关机调用

这里面有三个模式，reboot模式，休眠模式，shutdown模式

1. platform模式

   在断电前触发ACPI的\_PTS睡眠\_GTS等方法 调用hibernation_ops的方法

   在这种模式下开机时 BIOS 会通过特定的标志位更早地识别出需要从 Swap 恢复

2. shutdown模式

   直接走关机流程，下次开机时，系统会像普通冷启动一样经历完整的 BIOS 自检，直到内核识别到 Swap 分区里的休眠签名才会开始恢复

3. reboot模式

   直接走重启流程

```c
powerdown() {
    switch (hibernation_mode)
        case HIBERNATION_REBOOT:
            kernel_restart() {
                kernel_restart_prepare();       // 通知驱动子系统即将重启，触发底层设备的关闭回调
                migrate_to_reboot_cpu();        // 强制切换至引导核心（CPU0），确保重启指令发出的环境稳定
                machine_restart(cmd) {
                    native_machine_emergency_restart() {
                        acpi_reboot();          // 绕过标准清理流程，通过 ACPI 控制寄存器直接触发硬件重置
                    }
                }
            }
        case HIBERNATION_PLATFORM:
            hibernation_platform_enter() {
                hibernation_ops->begin();       // 调用 ACPI 规范中的 _PTS 方法，通知硬件即将进入 S4 状态
                hibernation_ops->prepare();     // 调用 ACPI 规范中的 _GTS 方法，进行最后的物理电平切换准备
                disable_nonboot_cpus();         // 遵循 ACPI 协议规范，必须在单核心环境下执行状态切换
                hibernation_ops->enter();       // 最终向 PM1_CNT 寄存器写入 S4 编码，由硬件触发断电
            }
        case HIBERNATION_SHUTDOWN:
            kernel_power_off() {
                machine_power_off() {
                    native_machine_power_off() {
                        // 调用在 acpi_sleep_init 中绑定的 acpi_power_off 钩子
                        pm_power_off() -> acpi_power_off(); // 执行标准的 S5 (Soft Off) 物理关机流程
                    }
                }
            }
}
```



```c
state_store() {
	hibernate() {
        ksys_sync(); 			// sync
        freeze_processes();		// 冻结用户态线程
        create_basic+memory_bitmaps();	// 分配内存页面位图，标记哪些内存是需要备份的
        hibernation_snapshot() {		// 快照保存
        	hibernation_ops->begin();
            freeze_kernel_threads();	// 冻结内核线程
            create_image() {
                platform_pre_snapshot():hibernation_ops->pre_snapshot();
                disable_nonboot_cpus();	// 关闭其他cpu
                local_irq_disable(); 	// 关闭中断
                swsusp_arch_suspend();	// <------- 拍摄快照 开机的时候加载完swap会从这里开始执行
                platform_leave():hibernation_ops->leave();
                local_irq_enable();		// 开中断
                enable_nonboot_cpus();	// 开启其他cpu
                platform_finish():hibernation_ops->finish();
            }
        }
        swsusp_write();			// 将内存快照写进swap分区
        power_down() {			
            kernel_restart() {		
            	machine_restart() {
                    native_machine_restart() {
                        native_machine_emergency_restart() {
                            acpi_reboot() {
                            	switch (rr->space_id) {		// 写入复位信号
                                	case ACPI_ADR_SPACE_PCI_CONFIG:		// 复位寄存器是pci设备
                                        pci_bus_write_config_byte();	// 复位寄存器位于普通的内存映射IO或系统I/O端口
                                    case ACPI_ADR_SPACE_SYSTEM_MEMORY:
                                    case ACPI_ADR_SPACE_SYSTEM_IO: 
                                       	acpi_reset() {
                                        	acpi_os_write_port(); 	// 处理io端口
                                            else acpi_hw_write();	// 处理非io端口
                                        }
                                }
                            }
                        }
                    } 
                }
            }
        }
    }
}
```

##### 2.2 恢复代码调用

###### 1)  恢复代码总体调用栈

```c
software_resume() {
	load_image_and_restore() {		// 加载镜像
        create_basic_memory_bitmaps();
        swsusp_read();
        hibernation_restore() {
            resume_target_kernel() {
                platform_pre_restore();
                hibernate_resume_nonboot_cpu_disable();
                local_irq_disable(); 
				swsusp_arch_resume() { 		// 执行完这里将会回到拍摄快照的时候的那个位置继续执行
                	restore_image();
                    restore_registers(); 	// 在这里修改in_suspend
                }
            }
        }
    }	
}
```



#### 3. 核心代码分析

##### hibernation_snapshot

```c
int hibernation_snapshot(int platform_mode)
{
	...
    pm_suspend_clear_flags();
    error = platform_begin(platform_mode);	// 通知 BIOS 准备进入休眠
    error = hibernate_preallocate_memory();	// 预先分配镜像所需要的内存
    error = freeze_kernel_threads();		// 冻结内核线程
    error = dpm_prepare(PMSG_FREEZE);		// 驱动程序休眠准备
    pm_restrict_gfp_mask();					// 限制内存分配掩码
    error = dpm_suspend(PMSG_FREEZE);		// 挂起设备 调用驱动的钩子函数，关闭大部分设备
    
    if (error || hibernation_test(TEST_DEVICES))
        platform_recover(platform_mode);	// 挂起失败使用acpi尝试恢复
    else 
        error = create_image(platform_mode);	// 执行创建img过程
    
    // 恢复的时候从这里开始执行 然后in_suspend这个变量会有区别 用于区分休眠动
    if (error || !in_suspend)	// 如果是恢复状态 就直接释放预分配的内存 因为不需要
        swsusp_free();
	// 确定现在状态
    msg = in_suspend ? (error ? PMSG_RECOVER : PMSG_THAW) : PMSG_RESTORE;
    dpm_resume(msg);			// 唤醒驱动程序

    if (error || !in_suspend)	// 允许自由分配内存
        pm_restore_gfp_mask();

    resume_console();			// 重启控制台
    dpm_complete(msg);			// 完成驱动设备唤醒

 Close:
    platform_end(platform_mode);	// 
    return error;

 Thaw:
    thaw_kernel_threads();
 Cleanup:
    swsusp_free();
    goto Close;
}

```



#### 4. 重点逻辑分析

reboot的过程是先sync然后冻结所有线程

因为s4的关机是将之前的内存镜像加载重新执行，所以加载完镜像之后会继续执行hibernate这个函数，区分这两个过程的标志是变量in_suspend

逻辑 

```c
// kernel/power/bibernate.c 
int hibernate()
...
if (in_suspend) {
    ...
	power_down();
} else {
	pm_pr_dbg("Image restored successfully.\n");                                                       
}
... 
```

所以是如果in_suspend为1 表示是休眠部分，调用powerdown执行关机命令，如果是0则是恢复部分，后续将执行其他恢复操作

在恢复部分对in_suspend变量的操作是在 

```assembly
; arch/x86/power/hibernate_asm_64.S
ENTRY(restore_registers)
	... 
    movq    %rax, in_suspend(%rip)
    ret 
ENDPROC(restore_registers) 
```

这里将in_suspend设置为0后，回去将能顺利执行恢复部分的代码

#### 5. acpi交互

#### 6. 控制权交互

1. OS 阶段 linux准备好镜像并保存到磁盘

2. 交接点

   OS 调用 acpi_enter_sleep_state(ACPI_STATE_S4)

   写入 PM 寄存器，主板切断主要电源

   此时，所有权属于硬件/固件电源管理模块

3. 唤醒触发

   用户按下电源键

   **固件 (BIOS/UEFI) 取得控制权**：执行内存自检（POST），发现硬件曾处于 S4 标志位

4. grub

   引导程序加载内核镜像

   此时内核被当作冷启动加载，直到执行 software_resume

5. 控制权回归打回到OS

   内核检查内核命令行参数 `resume=/dev/sdx`

   内核从磁盘读取数据覆盖当前内存

    调用 restore_processor_state()，代码从 swsusp_arch_suspend 的返回点继续执行

   


### 三、实验测试

使用qumu启动debian进行hibernation的测试

#### 1. 整体流程分析

整个的流程就是<br>先sync文件系统，然后先冻结用户线程，关闭OOM killer，然后创建bitmap 记录需要保存的物理页，冻结内核线程，冻结硬件设备，关闭中断，禁用其他cpu，拍摄快照[快照]，创建镜像，启动其他cpu，开启中断，写入镜像，进入关机流程

开机的时候<br>冻结用户线程，冻结内核线程，关闭OOMkiller，创建基本bitmap，加载镜像[进入快照]，启动其他cpu，开启中断，开启外设，释放bitmap内存，恢复其他线程，然后退出hibernation

#### 2. 日志结果

##### 2.1 关机日志

```
[   23.566995] PM: Hibernation mode set to 'reboot'
[   23.567748] PM: hibernation entry
[   23.575401] PM: Syncing filesystems ... 
[   23.616652] PM: done.
[   23.616887] Freezing user space processes ... (elapsed 0.002 seconds) done.
[   23.619881] OOM killer disabled.
[   23.620861] PM: Marking nosave pages: [mem 0x00000000-0x00000fff]
[   23.621147] PM: Marking nosave pages: [mem 0x0009f000-0x000fffff]
[   23.621328] PM: Basic memory bitmaps created
[   23.621635] PM: Preallocating image memory... done (allocated 41634 pages)
[   23.770731] PM: Allocated 166536 kbytes in 0.14 seconds (1189.54 MB/s)
[   23.770893] Freezing remaining freezable tasks ... (elapsed 0.001 seconds) done.
[   23.817087] PM: freeze of devices complete after 43.056 msecs
[   23.818889] PM: late freeze of devices complete after 1.571 msecs
[   23.821630] PM: noirq freeze of devices complete after 1.983 msecs
[   23.822018] Disabling non-boot CPUs ...
[   23.838222] smpboot: CPU 1 is now offline
[   23.850525] smpboot: CPU 2 is now offline
[   23.859356] smpboot: CPU 3 is now offline
[   23.860475] PM: Creating hibernation image:
[   23.860475] PM: Need to copy 40759 pages
[   23.860475] PM: Normal pages needed: 40759 + 1024, available pages: 221228
[   23.860475] PM: Hibernation image created (40759 pages copied)
[   23.860475] PM: Timekeeping suspended for 0.016 seconds
[   23.861703] Enabling non-boot CPUs ...
[   23.861974] x86: Booting SMP configuration:
[   23.862067] smpboot: Booting Node 0 Processor 1 APIC 0x1
[   23.863003]  cache: parent cpu1 should not be sleeping
[   23.865729] CPU1 is up
[   23.865912] smpboot: Booting Node 0 Processor 2 APIC 0x2
[   23.866820]  cache: parent cpu2 should not be sleeping
[   23.868600] CPU2 is up
[   23.868817] smpboot: Booting Node 0 Processor 3 APIC 0x3
[   23.869979]  cache: parent cpu3 should not be sleeping
[   23.871933] CPU3 is up
[   23.875432] PM: noirq thaw of devices complete after 2.855 msecs
[   23.877200] PM: early thaw of devices complete after 1.288 msecs
[   23.903684] PM: thaw of devices complete after 26.266 msecs
[   23.904548] PM: Writing image.
[   24.038334] ata2.01: NODEV after polling detection
[   24.046073] PM: Using 3 thread(s) for compression
[   24.046252] PM: Compressing and saving image data (40839 pages)...
[   24.046699] PM: Image saving progress:   0%
[   24.245191] PM: Image saving progress:  10%
[   24.370119] PM: Image saving progress:  20%
[   24.505850] PM: Image saving progress:  30%
[   24.670568] PM: Image saving progress:  40%
[   24.832371] PM: Image saving progress:  50%
[   25.001559] PM: Image saving progress:  60%
[   25.089819] PM: Image saving progress:  70%
[   25.156418] PM: Image saving progress:  80%
[   25.300744] PM: Image saving progress:  90%
[   25.387305] PM: Image saving progress: 100%
[   25.388528] PM: Image saving done
[   25.388610] PM: Wrote 163356 kbytes in 1.34 seconds (121.90 MB/s)
[   25.389384] PM: S|
[   25.932885] e1000: enp0s3 NIC Link is Up 1000 Mbps Full Duplex, Flow Control: RX
[   26.430319] sd 0:0:1:0: [sdb] Synchronizing SCSI cache
[   26.478773] sd 0:0:0:0: [sda] Synchronizing SCSI cache
[   26.708630] reboot: Restarting system
[   26.708794] reboot: machine restart
```

##### 2.2 开机日志

```
[    2.539535] PM: Checking hibernation image partition /dev/sdb
[    2.539921] cfg80211: failed to load regulatory.db
[    2.540000] PM: Hibernation image partition 8:16 present
[    2.540129] PM: Looking for hibernation image.
[    2.541920] PM: Image signature found, resuming
[    2.542086] PM: resume from hibernation
[    2.547354] PM: Preparing processes for restore.
[    2.547679] Freezing user space processes ... (elapsed 0.000 seconds) done.
[    2.548497] OOM killer disabled.
[    2.548631] Freezing remaining freezable tasks ... (elapsed 0.001 seconds) done.
[    2.550616] PM: Loading hibernation image.
[    2.551977] PM: Marking nosave pages: [mem 0x00000000-0x00000fff]
[    2.552259] PM: Marking nosave pages: [mem 0x0009f000-0x000fffff]
[    2.552770] PM: Basic memory bitmaps created
[    2.566348] PM: Using 3 thread(s) for decompression
[    2.566465] PM: Loading and decompressing image data (40839 pages)...
[    2.953088] input: ImExPS/2 Generic Explorer Mouse as /devices/platform/i8042/serio1/input/input3
[    2.953872] PM: Image loading progress:   0%
[    4.314496] PM: Image loading progress:  10%
[    4.411758] PM: Image loading progress:  20%
[    4.539809] PM: Image loading progress:  30%
[    4.652579] PM: Image loading progress:  40%
[    4.725588] PM: Image loading progress:  50%
[    4.802332] PM: Image loading progress:  60%
[    4.884944] PM: Image loading progress:  70%
[    4.952024] PM: Image loading progress:  80%
[    5.027977] PM: Image loading progress:  90%
[    5.110200] PM: Image loading progress: 100%
[    5.110915] PM: Image loading done
[    5.111198] PM: Read 163356 kbytes in 2.54 seconds (64.31 MB/s)
[    5.114713] PM: Image successfully loaded
[    5.129918] PM: quiesce of devices complete after 13.853 msecs
[    5.132019] PM: late quiesce of devices complete after 1.887 msecs
[    5.134512] PM: noirq quiesce of devices complete after 1.503 msecs
[    5.134819] Disabling non-boot CPUs ...
[   23.860475] PM: Timekeeping suspended for 11.016 seconds
[   23.877857] Enabling non-boot CPUs ...
[   23.884796] x86: Booting SMP configuration:
[   23.884914] smpboot: Booting Node 0 Processor 1 APIC 0x1
[   23.898182]  cache: parent cpu1 should not be sleeping
[   23.922386] CPU1 is up
[   23.923219] smpboot: Booting Node 0 Processor 2 APIC 0x2
[   23.924188]  cache: parent cpu2 should not be sleeping
[   23.925914] CPU2 is up
[   23.926088] smpboot: Booting Node 0 Processor 3 APIC 0x3
[   23.927018]  cache: parent cpu3 should not be sleeping
[   23.930357] CPU3 is up
[   23.940036] PM: noirq restore of devices complete after 7.004 msecs
[   23.947919] PM: early restore of devices complete after 7.161 msecs
[   23.991699] sd 0:0:1:0: [sdb] Starting disk
[   23.991707] sd 0:0:0:0: [sda] Starting disk
[   24.154510] ata2.01: NODEV after polling detection
[   24.167581] PM: restore of devices complete after 182.923 msecs
[   24.178294] PM: Image restored successfully.
[   24.178974] PM: Basic memory bitmaps freed
[   24.179269] OOM killer enabled.
[   24.179399] Restarting tasks ... done.
[   24.206010] PM: hibernation exit
```



