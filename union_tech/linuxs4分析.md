### 4.19 s4过程分析

#### 简介

##### 什么是s4

- 电源管理中的S4 挂载到磁盘(也称休眠)

睡眠状态会将机器状态保存到交换空间，并完全关闭机器电源。机器开机后，状态会恢复。在此之前，功耗为零

##### acpi

休眠中的acpi操作结构体

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

##### 用户态接口

```bash
echo reboot > /sys/power/disk
echo disk > /sys/power/status
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
                platform_finish(platform_mode):hibernation_ops->finish();
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







