### 设备管理
#### 计算机设备的连接和通信
##### 总线(bus)
设备一般通过bus和cpu通信，常见的总线有AMBA，PCIe等，设备和bus的工频率一般低于cpu
1. AMBA总线
    arm上的片上总线是级微控制器总线结构(AdvancedMicro-controller-Bus-Archiecture , MBA)规范，该规范定义了ARM架构片上系统(System-on-chip Soc)的通信标准，AMBA可以将arm处理器和其他ip盒进行集成，比如arm和fpga

    AMBA包括三组总线
    - 高级高性能总线（Advanced High-Performance Bus, AHB）：用于连接其他高性能 IP 核、片上和片外内存以及中断控制器等高性能模块
    - 高级系统总线（Advanced System Bus, ASB）：用于某些不必使用 AHB 但同时又需要高性能特性的芯片中，能起到一部分降低功耗的作用
    - 高级设备总线（Advanced Peripheral Bus, APB）：用于连接低速的设备，作为低功耗的精简接口总线
2. PCI总线
   组件互连（Peripheral Component Interconnect， PCI）标准，通过pci插槽连接，每个pci设备有一个唯一识别编号，包括总线号(bus number)，设备号(device number)，功能号(function number)

   当数据传输速率过高时，PCI 所采用的并行线路间会相互干扰。PCIe（PCI Express） 使用了基于数据包的串行连接协议，其相比于 PCI 总线来说带宽更高，同时保持了 PCI 设备驱动的前向兼容性

##### 可编程io
cpu通常以读写寄存器的方式和设备进行通信，一个设备有多个寄存器，可以在内存地址或者io地址间访问
- 设备寄存器有三个类型
    1. 控制寄存器: 接受驱动程序的命令
    2. 状态寄存器: 反馈设备的工作状态
    3. 输入/输出寄存器: 用于驱动和设备之间的数据交互
- 设备访问寄存器有两种方式
    1. 内存映射 MMIO(Memory-Mapped I/O)，将设备直接映射到内存空间，有独立的内存地址，读写内存指令即可控制设备
    2. 端口映射 PMIO(Port-Mapped I/O)，通过端口操作指令和设备交互
    - 这两个统称 (PIO Programm I/O)

##### DMA
直接访问内存(Direct Memory Access, DMA)是一种在设备和内存之间高效传输的方式，和PIO不同，PIO需要cpu参与主动搬运数据，但是DMA允许设备绕过cpu直接读写内存

DMA的发起者可以是cpu也可以是设备，设备发起dma的过程
  1. 设备驱动在内存中申请一段DMA缓冲区
  2. 发起DMA请求
  3. 设备通过dma将数据拷贝到DMA缓冲区
  4. 拷贝完成设备触发中断通知cpu对缓冲区的数据进行处理

###### DMA控制器
在某些架构中，DMA的发起还需要DMA控制器的参与，DMA控制器和cpu和内存在同一系统总线上，有DMA控制器发起DMA的过程
  1. cpu向控制器发送DMA缓冲区的位置和长度，以及数据传输的方向，之后放弃总线的控制权
  2. 控制器获得总线权限，然后将设备的数据拷贝到内存
  3. 完成后控制器向cpu发送中断，cpu重新获得总线控制权
  - 如果是设备发起DMA那就是先通知控制器

![DMA控制器的DMA过程](./img/dma控制器工作流程.png)  

有些架构可以让设备直接获得总线的控制权然后进行操作
    
###### iommu
大多数情况下，cpu上执行的操作系统代码在进行内存访问时使用的是虚拟地址(virtualaddress)，并通过 MMU 翻译成物理地址(physical address）)，设备进行 DMA 时访问的内存地址是总线地址(bus address)，其不同于虚拟地址和物理地址，当操作系统向 DMA 控制器注册 DMA 内存缓冲区时，需要填写的是总线地址，设备和内存之间的iommu(Input-Output Memory Management Unit，IOMMU)会负责将总线地址翻译成物理地址

在一些场景下设备使用的总线地址和物理地址相同。由于 DMA 在进行过程中会向连续的地址写入数据，因此操作系统需要分配连续的物理内存作为 DMA 缓冲区。IOMMU 的存在免除了 DMA 缓冲区各内存页必须物理连续的限制，同时 IOMMU 还限定了设备可以访问到的物理内存范围

- IOMMU 的另一个重要用途是为操作系统提供内存访问保护机制，用于防止恶意设备以及驱动对物理内存的非法访问。此外，IOMMU 还可用于在虚拟化环境中防止恶意虚拟机通过设备对其他虚拟机和虚拟机监控器内存进行非法访问

![iommu地址关系](./img/iommu地址关系.png)


#### vfio
##### 基础概念
- vfio是虚拟化硬件加速方案中的其中一个方案
- vfio是一个设备直通(pass-through)技术，用户态可以直接用过使用vfio驱动直接访问硬件，这个过程是基于IOMMU保护的，iommu可以把设备io，中断，dma等能力呈现给用户态

##### 原理
vfio使用DMA remapping和Interrupt remapping，DMA remapping使用iommu限制设备的io访问，Interrupt remapping使用中断重映射做中断隔离

##### iommu
iommu是一种内存管理单元（MMU），它将具有直接存储器访问能力（可以DMA）的I/O总线连接至主内存，iommu将设备可见的虚拟地址（在此上下文中也称设备地址或I/O地址）映射到物理地址。部分单元还提供内存保护功能，防止故障或恶意的设备
iommu使用iommu guest os可以直接访问gpa(guest physical address)就不访问hpa(host physical address)，iommu维护gpa或者iova到hpa的映射

##### group
iommu和vfio都有group的概念，两者几乎是一样的
iommu对dma的隔离的单位是group，group可以只有一个或多个devices，如果一个设备可以绕过iommmu和其他设备通信，比如p2p(peer to peer)，就不能独立成一个group

##### container
多个group可以绑定到一个vfio container，共享iommu的上下文，比如iova到hva的映射

![img](./img/iommu和device.png)

- vfio使用样例
```c
int container, group, device;
struct vfio_group_status group_status = { .argsz = sizeof(group_status) };
struct vfio_iommu_type1_info iommu_info = { .argsz = sizeof(iommu_info) };
struct vfio_iommu_type1_dma_map dma_map = { .argsz = sizeof(dma_map) };
struct vfio_device_info device_info = { .argsz = sizeof(device_info) };

container = open("/dev/vfio/vfio", O_RDWR);
group = open("/dev/vfio/18", O_RDWR);
ioctl(group, VFIO_GROUP_SET_CONTAINER, &container);
ioctl(container, VFIO_SET_IOMMU, VFIO_TYPE1_IOMMU);
ioctl(container, VFIO_IOMMU_GET_INFO, &iommu_info);

// 将group加入到container中
ioctl(group, VFIO_GROUP_SET_CONTAINER, &container);
ioctl(container, VFIO_SET_IOMMU, VFIO_TYPE1_IOMMU);
ioctl(container, VFIO_IOMMU_GET_INFO, &iommu_info);

// 建立iova到hpa的映射
dma_map.vaddr = mmap(0, 1024 * 1024, PROT_READ | PROT_WRITE,
                     MAP_PRIVATE | MAP_ANONYMOUS, 0, 0);
dma_map.size = 1024 * 1024;
dma_map.iova = 0;
dma_map.flags = VFIO_DMA_MAP_FLAG_READ | VFIO_DMA_MAP_FLAG_WRITE;
ioctl(container, VFIO_IOMMU_MAP_DMA, &dma_map);
```

##### vfio驱动
上面样例代码中的container group device分别对应vfio里面的 vfio_container vfio_group vfio_device

- 调用栈
```
ioctl()系统调用
vfio_fops_unl_ioctl
vfio_iommu_type1_ioctl
vfio_iommu_type1_map_dma
vfio_pin_map_dma
vfio_iommu_map
iommu_map
```

![vfio主要结构体](./img/vfio主要结构体.png)