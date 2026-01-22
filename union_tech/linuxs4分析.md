### 4.19 s4过程分析

#### 简介

##### 什么是s4

电源管理中的S4 挂载到磁盘(也称休眠)

睡眠状态会将机器状态保存到交换空间，就是将内存中的内容存入交换空间、并完全关闭机器电源。机器开机后，状态会恢复。在此之前，功耗为零

##### ACPI (高级配置和电源接口)

ACPI首先可以理解为一个独立于体系结构的电源管理和配置框架，它在主机OS中形成一个子系统。该框架建立一个硬件寄存器集来定义电源状态(休眠、hibernate、唤醒等)。硬件寄存器集可以容纳专用硬件和通用硬件上的操作

标准ACPI框架和硬件寄存器集的主要目的是启用电源管理和系统配置，无需操作系统来直接调用固件。ACPI作为系统固件(BIOS]和OS之间的接口层

##### 用户态接口

```bash
# 这里主要以reboot为例
echo reboot > /sys/power/disk
echo disk > /sys/power/status
```

#### 重要数据结构

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

#### 函数调用流程 

reboot的过程是先sync然后冻结所有线程

##### 重启调用 (关机部分)

echo reboot > /sys/power/disk
echo disk > /sys/power/status

~~~c
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
~~~

##### 重启调用 (开机部分)

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

#### 阶段流程分析

##### 总体流程

关机流程

sync同步 -> 冻结用户线程->  冻结内核线程 ->  关闭中断关闭其他cpu -> 拍摄快照[快照] -> 恢复其他cpu开启中断 -> 进入关机流程 -> 发送重启指令

开机流程







