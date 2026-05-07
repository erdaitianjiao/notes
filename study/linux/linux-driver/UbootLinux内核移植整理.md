## Uboot启动流程

启动各个阶段

```
上电复位 → ROM Bootloader → SPL / 初期引导 → U-Boot 主体 → 装载 Linux 内核 + 设备树 → 跳转到内核入口
```

下面逐步拆解：



### 1. 上电 / ROM Bootloader 阶段

- SoC 上电 / 复位后执行固化在片上的 BootROM（iROM / Mask ROM），这是芯片厂商写好的程序
- BootROM 根据引脚或 fuse（e.g. boot mode pins）判断从哪种介质启动（SD、eMMC、NAND、SPI 等）
- BootROM 负责把 SPL（Secondary Program Loader）或最初的一段引导代码从外存加载到片上 SRAM / 内部 RAM，作为启动入口



### 2. SPL / 初期引导（如果 SoC 有 SPL 或类似方案）

- SPL 是一个精简版的 U-Boot，引导初期硬件环境（如 DDR 初始化、时钟、功耗等），因为在 BootROM 能访问的资源很少
- 完成 DRAM 初始化后，SPL 将 U-Boot 主程序加载到 DRAM 中，并跳转运行它
- 这样做是为了让完整版 U-Boot 能在 DRAM 上运行，摆脱地址限制



### 3. U-Boot 主体初始化阶段

这个阶段可以拆成两个子阶段：**前期初始化 + 代码重定位 + 后期初始化**

#### 3.1 前期初始化（board_init_f）

- 进入 `board_init_f`（在 `board_f.c`）前，需要设置好早期环境，如串口、定时器、内存布局、全局数据（global_data）等
- 计算 U-Boot 将被 **重定位** 的目标地址（`gd->relocaddr`）以及重定位后堆栈地址等
- 这个阶段不能使用 DRAM 上的高级功能，因为还没完全初始化

#### 3.2 代码重定位（relocate阶段）

- 在 `relocate.S` 或相关汇编代码中，将 U-Boot 自己从当前加载地址（可能是在 Flash 或 SRAM）拷贝到 DRAM 的重定位地址
- 拷贝后还要修正符号引用、全局变量、静态变量的地址（即处理 relocations / GOT / .rel.dyn 段等）
- 此外要调整中断向量表的位置（`relocate_vectors.S`），把向量表移到新的地址，并配置 VBAR（向量基址寄存器）
- 拷贝完毕后，U-Boot 从新的地址继续执行

> 补充一点：U-Boot 的 `global_data` 结构中含有 `relocaddr`、`reloc_off`、`start_addr_sp` 等字段，用于记录重定位相关信息([docs.u-boot.org](https://docs.u-boot.org/en/latest/develop/global_data.html?utm_source=chatgpt.com))

#### 3.3 后期初始化（board_init_r）

- 在 `board_init_r` 阶段，U-Boot 初始化剩余设备（如 MMC、网络、环境变量、USB、GPIO 等）
- 初始化环境变量、控制台、缓存、存储设备等
- 最终进入主循环 (`main_loop`)：倒计时启动 + CLI 命令行界面等



### 4. 启动 Linux 内核（bootz / booti 等命令）

当用户或自动执行 `bootz` / `booti` 命令时：

1. **解析命令参数**

   - 获取内核镜像地址（zImage / Image）、设备树 (FDT) 地址 / initrd 地址
   - 如果是 zImage 格式，则 U-Boot 会调用 `bootz`；如果是 Image (非压缩) 则用 `booti`([Stack Overflow](https://stackoverflow.com/questions/65054120/booting-linux-from-u-boot-raspberry-pi-3-b-plus-u-boot-bootz-command-not-pre?utm_source=chatgpt.com))

2. **验证内核镜像**

   - 检查 zImage 的头部魔数、压缩格式等，确保镜像合法
   - 对齐镜像地址（有些平台要求 alignment）([Welcome to the Mike’s homepage!](https://krinkinmu.github.io/2023/08/21/how-u-boot-loads-linux-kernel.html?utm_source=chatgpt.com))

3. **准备参数 / 传参**

   - 根据是否使用设备树（FDT）决定传递方式：如果用设备树，R2 就是设备树地址；若不使用，则使用 ATAG

   - 同时把 `bootargs`（如 console、root 等参数）写入设备树的 `/chosen` 节点

   - 最后内核入口函数原型通常是：

     ```c
     kernel_entry(r0 = 0, r1 = machine_id, r2 = fdt_addr_or_atag)
     ```

4. **跳转至内核入口**

   - U-Boot 从当前上下文跳转到内核入口点 (images->ep)，将控制权交给内核
   - 此后 U-Boot 不再执行（除非 kernel 返回，通常不会）



### 补充 

- U-Boot 必须在重定位之前保证不能引用 DRAM 中还未初始化的地址
- 重定位内存拷贝时要考虑 overlapping、对齐、cache 同步等
- 设备树如果有更新（例如 reserve 内存、修改属性等），U-Boot 有可能 **重新构建 / 重新复制 / 重定位设备树 blob**([Welcome to the Mike’s homepage!](https://krinkinmu.github.io/2023/08/21/how-u-boot-loads-linux-kernel.html?utm_source=chatgpt.com))
- 在加载内核 / DTB / initrd 时要确保它们在内存中不重叠
- `CONFIG_CMD_BOOTZ` 要在 U-Boot 功能配置中启用，否则 `bootz` 命令不可用([Stack Overflow](https://stackoverflow.com/questions/65054120/booting-linux-from-u-boot-raspberry-pi-3-b-plus-u-boot-bootz-command-not-pre?utm_source=chatgpt.com))
- `mkimage` 工具可以把 zImage 包装成 uImage，加上 header，适配老版本 U-Boot([Unix & Linux Stack Exchange](https://unix.stackexchange.com/questions/122526/how-to-convert-a-zimage-into-uimage-for-booting-with-u-boot?utm_source=chatgpt.com))
- U-Boot 的 `global_data` 结构在运行时存储很多关键状态（重定位地址、Stack 指针、环境变量、内存基址等）([docs.u-boot.org](https://docs.u-boot.org/en/latest/develop/global_data.html?utm_source=chatgpt.com))
- 有些平台允许 U-Boot 跳过重定位步骤（如果编译时设置 `SYS_TEXT_BASE` 等），但很多情况下跳过会引发数据访问错误或异常([e2e.ti.com](https://e2e.ti.com/support/processors-group/processors/f/processors-forum/409970/u-boot-relocate_code-problem?utm_source=chatgpt.com))

## Uboot移植过程


### 一、总体流程概述

U-Boot 移植的核心是：

> 基于官方评估板（EVK）的源码，针对目标硬件修改配置文件、驱动和启动参数，使其能在自定义平台上正确启动并引导 Linux



### **U-Boot 移植总体流程**

```
获取官方源码 → 编译测试官方配置 → 创建新板配置 → 驱动适配 → 配置启动参数 → 启动验证
```



### 二、第一阶段：获取与验证官方源码

1. **获取源码**
    从 NXP 官网下载适配 i.MX6ULL 的 U-Boot 源码（如 `uboot-imx-rel_imx_4.1.15_2.1.0_ga.tar.bz2`）

2. **编译测试官方板**

   ```bash
   make distclean
   make mx6ull_14x14_evk_emmc_defconfig
   make -j16
   ```

   烧录生成的 `u-boot.bin`，确认开发环境和交叉编译器工作正常



### 三、第二阶段：创建自定义开发板配置

1. **添加配置文件**

   ```bash
   cd configs
   cp mx6ull_14x14_evk_emmc_defconfig mx6ull_alientek_emmc_defconfig
   ```

   修改配置宏：

   ```bash
   CONFIG_TARGET_MX6ULL_ALIENTEK_EMMC=y
   CONFIG_SYS_EXTRA_OPTIONS="IMX_CONFIG=board/freescale/mx6ull_alientek_emmc/imximage.cfg,MX6ULL_EVK_EMMC_REWORK"
   ```

2. **创建板级头文件**

   ```bash
   cp include/configs/mx6ullevk.h include/configs/mx6ull_alientek_emmc.h
   ```

   修改 DDR、Flash、环境变量存储、驱动相关配置宏

3. **创建板级目录**

   ```bash
   cd board/freescale
   cp -r mx6ullevk mx6ull_alientek_emmc
   ```

   更新：

   - **Makefile**：修改目标名
   - **imximage.cfg**：修改插件路径
   - **Kconfig** / **MAINTAINERS**：修改板名和维护信息

4. **注册新板支持**
    在 `arch/arm/cpu/armv7/mx6/Kconfig` 添加：

   ```bash
   config TARGET_MX6ULL_ALIENTEK_EMMC
       bool "Support mx6ull_alientek_emmc"
       select MX6ULL
       select DM
       select DM_THERMAL
   source "board/freescale/mx6ull_alientek_emmc/Kconfig"
   ```



### 四、第三阶段：驱动适配

#### 网络驱动修改（PHY 更换）

- 修改 PHY 地址与类型：

  ```c
  #define CONFIG_FEC_MXC_PHYADDR 0x1
  #define CONFIG_PHY_REALTEK
  ```

- 删除与官方扩展芯片（74LV595）相关的代码

- 添加 PHY 复位 GPIO 控制：

  ```c
  #define ENET1_RESET IMX_GPIO_NR(5, 7)
  #define ENET2_RESET IMX_GPIO_NR(5, 8)
  
  gpio_direction_output(ENET1_RESET, 1);
  gpio_set_value(ENET1_RESET, 0);
  mdelay(20);
  gpio_set_value(ENET1_RESET, 1);
  mdelay(150);
  ```

#### LCD 驱动修改

- 在 `mx6ull_alientek_emmc.c` 中修改显示参数：

  ```c
  .mode = {
      .name = "TFT7016",
      .xres = 1024, .yres = 600,
      .pixclock = 19531,
      .left_margin = 140, .right_margin = 160,
      .upper_margin = 20, .lower_margin = 12,
      .hsync_len = 20, .vsync_len = 3,
  }
  ```

- 修改默认环境变量：

  ```bash
  "panel=TFT7016\0"
  ```

- 若无显示：
   在命令行手动设置：

  ```bash
  setenv panel TFT7016
  saveenv
  ```



### 五、第四阶段：配置启动命令与环境变量

#### 设置启动命令（bootcmd）

- **从 eMMC 启动：**

  ```bash
  setenv bootcmd 'mmc dev 1; fatload mmc 1:1 80800000 zImage; fatload mmc 1:1 83000000 imx6ull-alientek-emmc.dtb; bootz 80800000 - 83000000;'
  ```

- **从网络启动（调试用）：**

  ```bash
  setenv bootcmd 'tftp 80800000 zImage; tftp 83000000 imx6ull-alientek-emmc.dtb; bootz 80800000 - 83000000;'
  ```

#### 设置内核启动参数（bootargs）

```bash
setenv bootargs 'console=ttymxc0,115200 root=/dev/mmcblk1p2 rootwait rw'
```

参数说明：

- `console=ttymxc0,115200`：串口0为控制台
- `root=/dev/mmcblk1p2`：根文件系统分区
- `rootwait rw`：等待设备就绪并读写挂载

### 网络参数设置

```bash
setenv ipaddr 192.168.1.50
setenv serverip 192.168.1.250
setenv gatewayip 192.168.1.1
setenv netmask 255.255.255.0
setenv ethaddr 00:04:9f:04:d2:35
saveenv
```



### 六、第五阶段：编译与启动测试

1. **自动化编译脚本**

   ```bash
   #!/bin/bash
   make distclean
   make mx6ull_alientek_emmc_defconfig
   make -j16
   ```

2. **烧录测试**

   - 烧写 `u-boot.bin` 到 SD / eMMC；
   - 串口监视启动日志；
   - 使用 `ping` 测试网络；
   - 执行 `boot` 命令验证 Linux 启动

3. **调试与排查**

   - 使用 `printenv` 检查环境变量；
   - 用 `mmc read` / `tftp` 分步测试；
   - 确认 zImage 与 dtb 地址无重叠；
   - 若卡在 “Starting kernel...”，检查 bootargs 与设备树配置



### 七、总结与经验

移植的核心是：

- **配置文件迁移**
- **硬件驱动适配**
- **启动参数调整**

常见问题：

- PHY 地址错误 → 网络不通；
- LCD 参数不匹配 → 无显示；
- bootcmd / bootargs 错误 → 启动卡死；
- 环境变量未保存 → 重启后失效





##  Linux启动流程

启动流程

> **Bootloader → 内核汇编入口 → C语言核心初始化 → 内核线程 → Init进程 → 用户空间**



### 一、Bootloader 加载内核

- Bootloader（如 U-Boot）完成以下工作：
  - 将内核镜像 (zImage/uImage) 和设备树（DTB）加载到内存
  - 设置寄存器：
    - r0 = 0
    - r1 = 机器ID
    - r2 = ATAGS 或 DTB 指针
  - 跳转到内核入口点 `stext`

### 二、内核汇编入口（stext，head.S）

- CPU 初始化：
  - 确保 SVC 模式，关闭中断
  - 检查 CPU ID (__lookup_processor_type)
- 页表创建：
  - 创建初始页表 (__create_page_tables)
  - 为 MMU 开启做准备
- 启用 MMU (__enable_mmu)
- 跳转至 `__mmap_switched` 进入 C 环境

### 三、切换至 C 语言环境 (__mmap_switched)

- 初始化：
  - 清空 BSS 段
  - 初始化数据段
  - 保存 CPU ID、机器类型、ATAGS/DTB 指针
- 调用 `start_kernel`（架构无关 C 函数）



### 四、内核核心初始化 (start_kernel)

- 初始化子系统：
  - **中断与陷阱**：trap_init(), init_IRQ()
  - **内存管理**：mm_init(), bootmem_init(), build_all_zonelists()
  - **调度器**：sched_init()
  - **定时器**：init_timers(), time_init()
  - **控制台**：console_init()
  - **解析启动参数**：parse_early_param(), parse_args()
  - **架构特定初始化**：setup_arch(&command_line)
- 初始化 RCU、Slab 分配器
- 创建内核初始化线程：调用 `rest_init`



### 五、创建核心内核线程 (rest_init)

- 创建两个关键线程：
  1. `kernel_init` (PID=1)：启动用户空间
  2. `kthreadd` (PID=2)：管理内核线程
- 原始进程 (PID=0) → idle 进程



### 六、内核初始化线程 (kernel_init)

- 执行 `kernel_init_freeable`：
  - 初始化驱动模型，调用所有内核模块的 __init 函数
  - 打开 `/dev/console`（标准输入/输出/错误）
  - 释放初始化内存 (free_initmem)
  - 挂载根文件系统
  - 执行第一个用户空间 init 程序：
    - `rdinit=` → `init=` → `/sbin/init` → `/etc/init` → `/bin/init` → `/bin/sh`
  - 找不到则内核 panic



### 七、用户空间启动

- Init 程序启动（如 systemd、sysvinit、busybox init）
- 读取配置文件（/etc/inittab 或 systemd unit）
- 启动系统服务，建立终端，最终完成系统启动



### 八、流程图（简化）

```text
Bootloader (U-Boot)
        |
        V
stext (汇编)
  CPU初始化 → 页表 → 启用MMU
        |
        V
__mmap_switched (汇编)
  清BSS → 初始化数据段 → 调用 start_kernel
        |
        V
start_kernel (C)
  内核子系统初始化
  setup_arch → 调用 rest_init
        |
        V
rest_init
  kernel_init(PID=1)
  kthreadd(PID=2)
  idle(PID=0)
        |
        V
kernel_init
  驱动初始化 → 挂载根FS → 打开控制台 → 执行 /sbin/init
        |
        V
用户空间 Init
  启动服务 → 完成系统启动
```



### 记忆小技巧

- **口诀**：**Boot → 汇编入口 → C 核心 → 内核线程 → Init → 用户空间**
- **关键词**：
  - **Bootloader**：加载镜像、传 r0/r1/r2
  - **汇编**：stext, __mmap_switched, MMU, BSS
  - **核心初始化**：start_kernel → 中断、内存、调度、定时器、控制台
  - **线程**：rest_init → kernel_init (用户空间) / kthreadd / idle
  - **用户空间**：init → systemd/sysvinit → 系统服务



## Linux 内核移植流程

### 一、准备工作

1. **获取源码**
   - 下载 NXP 官方内核：`linux-imx-rel_imx_4.1.15_2.1.0_ga.tar.bz2`
   - 解压并重命名（如 `linux-imx-rel_imx_4.1.15_2.1.0_ga_alientek`）
2. **配置开发环境**
   - 安装交叉编译工具链：`arm-linux-gnueabihf-`
   - 可选：VSCode 工程配置，方便代码编辑



### 二、编译与测试官方内核

1. **配置 Makefile**

   ```text
   ARCH = arm
   CROSS_COMPILE = arm-linux-gnueabihf-
   ```

2. **编译内核**

   ```bash
   make clean
   make imx_v7_mfg_defconfig
   make -j16
   ```

   - 输出：`zImage`（内核镜像）+ `imx6ull-14x14-evk.dtb`（设备树）

3. **启动测试**

   ```bash
   tftp 80800000 zImage
   tftp 83000000 imx6ull-14x14-evk.dtb
   bootz 80800000 - 83000000
   ```

   - 若出现 `VFS: Unable to mount root fs`，需配置根文件系统



### 三、添加自定义开发板支持

1. **创建自定义配置**

   ```bash
   cp arch/arm/configs/imx_v7_mfg_defconfig arch/arm/configs/imx_alientek_emmc_defconfig
   ```

   - 屏蔽 `CONFIG_ARCH_MULTI_V6=y`（I.MX6ULL 为 ARMv7）

2. **添加设备树**

   ```bash
   cp arch/arm/boot/dts/imx6ull-14x14-evk.dts arch/arm/boot/dts/imx6ull-alientek-emmc.dts
   ```

   - 修改 `arch/arm/boot/dts/Makefile`：

     ```text
     dtb-$(CONFIG_SOC_IMX6ULL) += imx6ull-alientek-emmc.dtb
     ```

3. **编译自定义内核**

   - 编写脚本 `imx6ull_alientek_emmc.sh`：

     ```bash
     make distclean
     make imx_alientek_emmc_defconfig
     make menuconfig  # 可选
     make -j16
     ```

   - 测试启动自定义内核（新 `zImage` + `.dtb`）



### 四、关键驱动修改

1. **CPU 主频**

   - 默认动态调频（ondemand），198MHz

   - 高性能模式：

     ```text
     # CONFIG_CPU_FREQ_DEFAULT_GOV_ONDEMAND is not set
     CONFIG_CPU_FREQ_GOV_PERFORMANCE=y
     ```

   - 超频可修改 DTS 的 operating-points（测试用）

2. **EMMC 驱动**

   - DTS 启用 8 线模式：

     ```dts
     &usdhc2 {
       pinctrl-0 = <&pinctrl_usdhc2_8bit>;
       bus-width = <8>;
       non-removable;
       no-1-8-v;  // 禁用1.8V
       status = "okay";
     };
     ```

3. **网络驱动**

   - **V2.4 以前（PHY LAN8720A）**
     - DTS 配置复位引脚、PHY 地址
     - 驱动修改 fec_main.c、smsc.c
   - **V2.4 及以后（PHY SR8201F）**
     - DTS 配置 PHY 地址及复位延时 200ms
     - 驱动 fec_main.c 添加复位延时

4. **保存配置**

   - 图形化界面保存到 `imx_alientek_emmc_defconfig`



### 五、根文件系统支持

- 通过 `bootargs` 指定根文件系统：

  ```text
  root=/dev/mmcblk1p2
  ```

- 启动缺失根文件系统会报错：

  ```
  Kernel panic - not syncing: VFS: Unable to mount root fs
  ```



### 六、总结步骤

1. 获取并解压 NXP 官方内核
2. 编译测试官方配置，确保基础功能
3. 创建自定义配置文件与设备树
4. 修改关键驱动（CPU、EMMC、网络 PHY）
5. 编译测试自定义内核
6. 配置根文件系统完成最终启动



**记忆小技巧：**

> **源码 → 编译 → 自定义配置 → 驱动修改 → 编译测试 → 根文件系统**

这个流程可以快速覆盖内核移植核心环节：设备树修改、驱动适配和根文件系统配置

