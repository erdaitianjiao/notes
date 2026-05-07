# gpu-io-and-vfio

## 整理

### 1. 威胁模型

完全的信任硬件和对于软件的0信任，假设hypSec TCB可以被信任 iommu可以被信任
攻击者的目的是破坏和复制guest os里面的数据，包括内存，磁盘，vpcu的信息
排除硬件层面的攻击，假设云厂商是值得信任的

- 需要确认的一个问题
攻击者是只能拿到host os的或者是el1里面的Hostvisor的权限，是不能拿到vm的权限的吧

### 2. gpu-io

需要确定的点
- 在gpu中攻击者的威胁模型是什么，他可以访问到哪些内容
- 想问一下关于之前说的io加密，磁盘加密就不用说了，网络加密中，对传输的内容进行加密的目的就是

1. [NVIDIA Confidential Computing](https://www.nvidia.com/en-us/data-center/solutions/confidential-computing) 结合cpu tee vm

note
1. tee技术 intel tdx amd-sev armcc cpu sgx intel cca  
trust zone

提出ppcie，在数据通过PCIe总线从 CPU 内存传输到 GPU 显存的过程中，数据始终保持加密状态
数据进入 GPU 内部的特定硬件单元时才进行解密。即使攻击者在物理层截获 PCIe 传输的数据包也看不到内容

1. 个人方案的解决方案就是mmio加访问限制，但是这个confidential computing好像已经做了，我最近准备去研究一下这个

### 3. vfio
将GPU的寄存器空间（MMIO）映射给用户空间（QEMU/KVM），从而实现设备直通

将物理 GPU 从宿主机原生驱动解绑，并绑定到 vfio-pci 驱动，从而将设备控制权移交给用户态（QEMU/KVM）

IOMMU保护 利用 IOMMU 技术实现 DMA 重映射和中断重映射。这允许设备直接访问虚拟机内存，严格限制了设备的访问权限，确保其只能访问分配的内存且中断不会越界，从而实现安全隔离

### 4. Memory protection in Android using KVM

1. 为什么需要pkvm
在安全世界（Secure World）运行DRM、加密等敏感任务
但EL2（管理程序层）基本处于闲置状态。这导致了一个核心问题：Host os缺乏内存保护能力

2. 方案
基于ARMv8-A架构的nVHE实现
使用stage-2地址转换