### linux电源管理 s4分析报告 (基于linux4.19)

#### 一、简介

##### 1. 什么是s4

电源管理中的S4 挂载到磁盘(也称休眠)

睡眠状态会将机器状态保存到交换空间，就是将内存中的内容存入交换空间、并完全关闭机器电源。机器开机后，状态会恢复。在此之前，功耗为零

##### 2. ACPI (高级配置和电源接口)

ACPI首先可以理解为一个独立于体系结构的电源管理和配置框架，它在主机OS中形成一个子系统。该框架建立一个硬件寄存器集来定义电源状态(休眠、hibernate、唤醒等)。硬件寄存器集可以容纳专用硬件和通用硬件上的操作

标准ACPI框架和硬件寄存器集的主要目的是启用电源管理和系统配置，无需操作系统来直接调用固件。ACPI作为系统固件(BIOS]和OS之间的接口层

###### 2.1 协同机制

Linux S4 流程并非纯粹的软件行为，而是一场由操作系统（OS）主导、平台固件（ACPI BIOS）配合的协同演习。内核通过 `struct platform_hibernation_ops` 这一抽象接口，在关键的时间节点调用平台特定的钩子函数，确保软件状态（内存镜像）与硬件状态（寄存器、总线功率、中断环境）的一致性

###### 2.2 核心钩子函数功能对照

>.begin
>
>.prepare
>
>.enter
>
>.end
>
>.

##### 3. 用户态接口

```bash
# 这里主要以reboot为例
echo reboot > /sys/power/disk
echo disk > /sys/power/status
```

#### 二、代码流程分析

##### 1. 重要数据结构                                                                  

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

// 新的apci ops结构体
static const struct platform_hibernation_ops acpi_hibernation_ops = {
    .begin = acpi_hibernation_begin, // -> acpi_pm_start - Start system PM transition   
    .end = acpi_pm_end, // -> acpi_pm_end - Finish up system PM transition.
    .pre_snapshot = acpi_pm_prepare, // -> acpi_pm_prepare - Prepare the platform to enter the target sleep state and disable the GPEs.
    .finish = acpi_pm_finish, // -> acpi_pm_finish - Instruct the platform to leave a sleep state / This is called after we wake back up (or if entering the sleep state failed).
    .prepare = acpi_pm_prepare, // -> acpi_pm_prepare - Prepare the platform to enter the target sleep state and disable the GPEs.
    .enter = acpi_hibernation_enter, // -> acpi_enter_sleep_state 进入睡眠
    .leave = acpi_hibernation_leave,
    .pre_restore = acpi_pm_freeze, // -> acpi_pm_freeze - Disable the GPEs and suspend EC transactions
    .restore_cleanup = acpi_pm_thaw,
};
```



##### 2. 函数调用流程

###### 2.1 睡眠调用

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

###### 2.2 恢复代码调用

```
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



##### 3. 核心代码分析

##### 4. 重点逻辑分析

reboot的过程是先sync然后冻结所有线程

#### 三、实验验证分析

使用qumu启动debian进行hibernation的测试

##### 1. 整体流程分析

从日志中可以看出这个流程，但是可以注意到这个日志关机部分和开机部分有一部分是重叠的 

```
[   25.284901] PM: freeze of devices complete after 41.165 msecs
[   25.286311] PM: late freeze of devices complete after 1.297 msecs
[   25.288994] PM: noirq freeze of devices complete after 1.998 msecs
[   25.289127] Disabling non-boot CPUs ...
[   25.303495] smpboot: CPU 1 is now offline
[   25.313526] smpboot: CPU 2 is now offline
[   25.324817] smpboot: CPU 3 is now offline
[   25.325747] PM: Creating hibernation image:
[   25.325747] PM: Need to copy 39548 pages
```

整个的流程就是<br>先sync文件系统，然后先冻结用户线程，然后创建bitmap，冻结内核线程，冻结硬件设备，关闭中断，禁用其他cpu，拍摄其他快照，创建镜像，启动其他cpu，开启中断，写入镜像，进入关机流程

开机的时候<br>冻结用户线程，关闭OOMkiller，读取镜像，启动其他cpu，开启中断，开启外设，释放bitmap内存，恢复其他线程，然后推出这个hibernation

##### 2. 日志结果

1. 关机日志

```
[   25.007741] Adding 2097148k swap on /dev/sdb.  Priority:-2 extents:1 across:2097148k 
[   25.021742] PM: Hibernation mode set to 'reboot'
[   25.024514] PM: hibernation entry
[   25.030407] PM: Syncing filesystems ... 
[   25.074595] PM: done.
[   25.075035] Freezing user space processes ... (elapsed 0.002 seconds) done.
[   25.077579] OOM killer disabled.
[   25.078859] PM: Marking nosave pages: [mem 0x00000000-0x00000fff]
[   25.079098] PM: Marking nosave pages: [mem 0x0009f000-0x000fffff]
[   25.079266] PM: Basic memory bitmaps created
[   25.079406] PM: Preallocating image memory... done (allocated 40638 pages)
[   25.240380] PM: Allocated 162552 kbytes in 0.16 seconds (1015.95 MB/s)
[   25.240590] Freezing remaining freezable tasks ... (elapsed 0.001 seconds) done.
[   25.243343] Suspending console(s) (use no_console_suspend to debug)
[   25.284901] PM: freeze of devices complete after 41.165 msecs
[   25.286311] PM: late freeze of devices complete after 1.297 msecs
[   25.288994] PM: noirq freeze of devices complete after 1.998 msecs
[   25.289127] Disabling non-boot CPUs ...
[   25.303495] smpboot: CPU 1 is now offline
[   25.313526] smpboot: CPU 2 is now offline
[   25.324817] smpboot: CPU 3 is now offline
[   25.325747] PM: Creating hibernation image:
[   25.325747] PM: Need to copy 39548 pages
[   25.325747] PM: Normal pages needed: 39548 + 1024, available pages: 222443
[   25.325747] PM: Hibernation image created (39548 pages copied)
[   25.327115] Enabling non-boot CPUs ...
[   25.327329] x86: Booting SMP configuration:
[   25.327342] smpboot: Booting Node 0 Processor 1 APIC 0x1
[   25.328701]  cache: parent cpu1 should not be sleeping
[   25.331420] CPU1 is up
[   25.331508] smpboot: Booting Node 0 Processor 2 APIC 0x2
[   25.332197]  cache: parent cpu2 should not be sleeping
[   25.333618] CPU2 is up
[   25.333696] smpboot: Booting Node 0 Processor 3 APIC 0x3
[   25.334406]  cache: parent cpu3 should not be sleeping
[   25.335952] CPU3 is up
[   25.339177] PM: noirq thaw of devices complete after 2.814 msecs
[   25.340576] PM: early thaw of devices complete after 1.123 msecs
[   25.368380] PM: thaw of devices complete after 27.740 msecs
[   25.371921] PM: Writing image.
[   25.502957] ata2.01: NODEV after polling detection
[   25.509375] PM: Using 3 thread(s) for compression
[   25.509512] PM: Compressing and saving image data (39626 pages)...
[   25.510094] PM: Image saving progress:   0%
[   25.657480] PM: Image saving progress:  10%
[   25.791578] PM: Image saving progress:  20%
[   25.935055] PM: Image saving progress:  30%
[   26.139024] PM: Image saving progress:  40%
[   26.324067] PM: Image saving progress:  50%
[   26.444644] PM: Image saving progress:  60%
[   26.585831] PM: Image saving progress:  70%
[   26.700523] PM: Image saving progress:  80%
[   26.837912] PM: Image saving progress:  90%
[   26.915645] PM: Image saving progress: 100%
[   26.917854] PM: Image saving done
[   26.917962] PM: Wrote 158504 kbytes in 1.40 seconds (113.21 MB/s)
[   26.919159] PM: S|
[   27.430262] e1000: enp0s3 NIC Link is Up 1000 Mbps Full Duplex, Flow Control: RX
[   27.955290] sd 0:0:1:0: [sdb] Synchronizing SCSI cache
[   27.986493] sd 0:0:0:0: [sda] Synchronizing SCSI cache
[   28.150839] reboot: Restarting system
[   28.150989] reboot: machine restart
```

2. 开机日志

```
[    2.898539] PM: resume from hibernation
[    2.904409] Freezing user space processes ... (elapsed 0.000 seconds) done.
[    2.905480] OOM killer disabled.
[    2.905632] Freezing remaining freezable tasks ... (elapsed 0.001 seconds) done.
[    2.926167] PM: Using 3 thread(s) for decompression
[    2.926332] PM: Loading and decompressing image data (39626 pages)...
[    3.421389] input: ImExPS/2 Generic Explorer Mouse as /devices/platform/i8042/serio1/input/input3
[    3.427356] PM: Image loading progress:   0%
[    3.924620] PM: Image loading progress:  10%
[    4.011360] PM: Image loading progress:  20%
[    4.103217] PM: Image loading progress:  30%
[    4.208277] PM: Image loading progress:  40%
[    4.280453] PM: Image loading progress:  50%
[    4.358423] PM: Image loading progress:  60%
[    4.426480] PM: Image loading progress:  70%
[    4.494721] PM: Image loading progress:  80%
[    4.567551] PM: Image loading progress:  90%
[    4.634056] PM: Image loading progress: 100%
[    4.635052] PM: Image loading done
[    4.635390] PM: Read 158504 kbytes in 1.70 seconds (93.23 MB/s)
[    4.639900] Suspending console(s) (use no_console_suspend to debug)
[   25.284901] PM: freeze of devices complete after 41.165 msecs
[   25.286311] PM: late freeze of devices complete after 1.297 msecs
[   25.288994] PM: noirq freeze of devices complete after 1.998 msecs
[   25.289127] Disabling non-boot CPUs ...
[   25.303495] smpboot: CPU 1 is now offline
[   25.313526] smpboot: CPU 2 is now offline
[   25.324817] smpboot: CPU 3 is now offline
[   25.325747] PM: Creating hibernation image:
[   25.325747] PM: Need to copy 39548 pages
[   25.325747] PM: Normal pages needed: 39548 + 1024, available pages: 222443
[   25.325747] PM: Timekeeping suspended for 12.000 seconds
[   25.345272] Enabling non-boot CPUs ...
[   25.352589] x86: Booting SMP configuration:
[   25.352641] smpboot: Booting Node 0 Processor 1 APIC 0x1
[   25.366868]  cache: parent cpu1 should not be sleeping
[   25.394680] CPU1 is up
[   25.395264] smpboot: Booting Node 0 Processor 2 APIC 0x2
[   25.396142]  cache: parent cpu2 should not be sleeping
[   25.397584] CPU2 is up
[   25.397694] smpboot: Booting Node 0 Processor 3 APIC 0x3
[   25.398464]  cache: parent cpu3 should not be sleeping
[   25.403158] CPU3 is up
[   25.410674] PM: noirq restore of devices complete after 5.914 msecs
[   25.412461] PM: early restore of devices complete after 1.040 msecs
[   25.457623] sd 0:0:0:0: [sda] Starting disk
[   25.457631] sd 0:0:1:0: [sdb] Starting disk
[   25.618408] ata2.01: NODEV after polling detection
[   25.632249] PM: restore of devices complete after 183.638 msecs
[   25.640549] PM: Image restored successfully.
[   25.641470] PM: Basic memory bitmaps freed
[   25.641866] OOM killer enabled.
[   25.642010] Restarting tasks ... done.
[   25.664937] PM: hibernation exit
```



