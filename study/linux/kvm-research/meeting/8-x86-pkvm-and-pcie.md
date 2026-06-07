# 8 meetings

## 上次遗留

bar 和 bar空间和配置空间
ace 的 device id table 存放 and bar
把bar mmap到自己空间 是 bar本身还是 bar映射的空间

pv 半虚拟化
ept and pv mmu
pkvm保护的IOMMU 的对象
pkvm设备直通的具体实现

host 访问 vpcu状态
host 访问 guest内存
host 访问 直通的设备(安全性 如果是host创建的一个virt device?)

### bar和bar空间

设备在系统的PCI地址空间里申请一段来用，所申请的空间基址和大小保存在BAR基址寄存器里
*BAR里的只是PCI域的地址空间，需要映射到IO地址空间里或者内存空间里之后软件才能使用*

1. 从设备角度，bar是内部寄存器的编号，设备只知道访问了第几个位置，然后就把数据交付出去
2. 从系统角度来说，bar是系统给设备分配的门牌号，系统启动后在物理地址里面给每个设备都分配一个地址范围，把起始地址放到写进bar，CPU访问这段地址，硬件会自动把请求路由到对应设备上
3.  CPU访问 驱动先通过 ioremap把物理地址映射成虚拟地址，然后在代码里读写这个虚拟地址，MMU把它转成物理地址，硬件发现这个地址落在 MMIO 区

### ACS 规范
ACS 是 PCIe 规范里面的一个可选功能，解决 PCIe switch下游的设备之间不能p2p通信

- acs的 device id table
pcie switch/root port 内部会有一个路由控制器，核心作用是

如果siwtch收到一个tlp(PCIe 事务包) 需要判断
1. 这个包是向上游走还是下游，到另一个设备
2. 如果是向下游，acs会判断这个p2p是不是被允许的

acs的控制位包括
1. ACS Source Validation：检查请求的发起者(Requester ID = BDF)是否合法
2. ACS Translation Blocking：阻止未经过地址翻译的 DMA 请求 (必须走 IOMM)
3. ACS P2P Request Redirect：把设备间的 P2P 请求重定向到上游(Root Port)，强制经过IOMMU 检查
4. ACS Upstream Forwarding：控制上游转发行为

```eg
switch(none acs)
 -> devivce a
 -> siwtch(acs)
     -> device c
```

那么a和c是不是一个device

### qemu映射

qemu映射的是BAR指向的MMIO空间（设备寄存器/存储），不是BAR寄存器本身（配置空间里的那个32位值）

*kernel 函数 vfio_pci_core.c*
```c
static unsigned long vma_to_pfn(struct vm_area_struct *vma)
{
    struct vfio_pci_core_device *vdev = vma->vm_private_data;
    int index = vma->vm_pgoff >> (VFIO_PCI_OFFSET_SHIFT - PAGE_SHIFT);
    u64 pgoff;
    pgoff = vma->vm_pgoff & ((1U << (VFIO_PCI_OFFSET_SHIFT - PAGE_SHIFT)) - 1);
    // pci_resource_start 返回的是 BAR 对应的 MMIO 物理地址（如 0xD0000000）
    // 不是配置空间里 BAR 寄存器的地址（配置空间偏移 0x10~0x24）
    return (pci_resource_start(vdev->pdev, index) >> PAGE_SHIFT) + pgoff;
}
```

*kernel 函数 vfio_pci_core.c*
```c
int vfio_pci_core_mmap(struct vfio_device *core_vdev, struct vm_area_struct *vma)
{
    index = vma->vm_pgoff >> (VFIO_PCI_OFFSET_SHIFT - PAGE_SHIFT);
    // 只接受 BAR index（0~5），config region index 直接返回 EINVAL
    if (index >= VFIO_PCI_ROM_REGION_INDEX)
        return -EINVAL;
    // pci_resource_len 返回的是 BAR 的 MMIO 空间大小（如 2MB），不是配置空间大小
    phys_len = PAGE_ALIGN(pci_resource_len(pdev, index));
    ...
}
```

QEMU侧两条不同的路径
```c
vfio_pci_config_setup() {
    vfio_pci_config_space_read();

    vfio_bars_register() {
        vfio_bar_register() {
            vfio_region_mmap() {
                mmap(fd, MAP_SHARED, ...);  // mmap 映射的是 BAR MMIO 空间
            }
        }
    }
}
```

### 半虚拟化(PV)

需要修改客户端虚拟机的源代码就是半虚拟化(para-virtualization)

半虚拟化需要对客户操作系统与虚拟机监视器进行协调设计，一方面，hypervisor为虚拟机提供超级调用(hypercall)，hypercall和系统调用类似，涵盖了调度，内存，IO多方面的功能，另一方面需要修改客户操作系统的代码，替换不可虚拟化的指令，改为hypercall

xen就是半虚拟化的代表之一

半虚拟化的优势
1. 更高的性能
2. 缓解语义鸿沟问题，半虚拟化技术可以获取虚拟机内部的状态，提高资源分配效率

### ept and pv mmu
#### ept 
ept是 Intel 硬件提供的二层页表，用于虚拟化场景。

普通系统只有一层地址翻译：
虚拟地址 -> (MMU) -> 物理地址

虚拟化下有两层：
guest 虚拟地址 (GVA)
    -> guest 页表 (由 guest OS 维护)
    -> guest 物理地址 (GPA)
    -> ept (由 Hypervisor 维护)
    -> Host 物理地址 (HPA)

ept 就是 GPA -> HPA 这一层映射。 它是硬件自动完成的，不需要 Hypervisor 介入(除非ept miss 触发 vmexit)

有了 EPT：
guest 随意改自己的页表，不需要 vmexit
硬件自动做两次翻译：GVA -> GPA -> HPA
只有 ept miss（GPA 没有映射到 HPA）时才 vmexit

- pKVM 中的 ept

pKVM 维护了两个 EPT：

host EPT：映射 Host 能访问的物理内存
    -> guest 受保护的内存被标记为 NOPAGE（Host 看不到）

guest pgstate_pgt：映射 guest 直通设备的 IOMMU 页表
    -> 设备 DMA 只能访问 guest 允许的地址

#### PV MMU

guest 主动告诉 hypervisor 自己在干什么：

guest 要修改页表 -> 先 hypercall 通知 hypervisor -> hypervisor 提前处理

具体包括：

1. PV TLB 刷新
guest通过hypercall告诉 hypervisor刷新的位置，hypervisor 只刷新真正需要的核，减少IPI广播
1. PV 页表更新
guest 通过 hypercall 主动通知 hypervisor 切换了页表，hypervisor 提前更新 ept 缓存，减少后续 ept miss
1. PV 内存回收提示
guest 主动告诉 hypervisor "这几页我不用了，你可以回收"


### pkvm保护的IOMMU和设备直通


#### 威胁模型

pKVM 的威胁模型是 Host 不可信，保护的是 Guest(Protected VM) 的内存不被 Host 通过设备 DMA 窃取或篡改

```
Host OS（不可信）
  -> 能写 IOMMU 寄存器、配置设备、发起 DMA
  -> IOMMU -> PCIe 设备 -> 访问物理内存

没有 pKVM 时：Host 随便配 IOMMU 页表，设备 DMA 就能读到 Guest 的内存
```

#### pKVM IOMMU 三层保护

1. IOMMU MMIO 拦截 (iommu.c)
Host 对 IOMMU 寄存器的访问被拦截，Host 看到的是虚拟 IOMMU(viommu)
```c
case DMAR_GCMD_REG:     // 全局命令寄存器
    handle_global_cmd(iommu, val);   // pKVM 代替 Host 执行
case DMAR_RTADDR_REG:   // Root Table 地址
    viommu->vreg.rta = val;          // 只记录到虚拟寄存器
case DMAR_IQT_REG:      // Queue Invalidation
    handle_qi_invalidation(iommu, val); // pKVM 审计后再执行
```

2. Shadow IOMMU 页表 (iommu.c)
Host 维护自己的 IOMMU 页表(vpgt)，但 pKVM 维护独立的 shadow 页表(pgt)，真正写入硬件的是 shadow 页表
```c
struct pkvm_ptdev {
    struct pkvm_pgtable vpgt;   // Host 维护的（不可信）
    struct pkvm_pgtable *pgt;   // pKVM 维护的 shadow（真正生效）
};
```

审计核心 (iommu_audit_did)：不同 VM 的设备不能用同一个 domain ID，防止 Host 把 Guest 设备的 domain 指向 Host 的 IOMMU 页表

3. Host EPT 内存隔离 (mem_protect.h)
```c
enum pkvm_page_state {
    PKVM_NOPAGE,              // Host 看不到（Guest 私有内存）
    PKVM_PAGE_OWNED,          // Host 独占
    PKVM_PAGE_SHARED_OWNED,   // Host 和 Guest 共享
    PKVM_PAGE_SHARED_BORROWED,// Guest 借用的共享页
};
```

Guest 私有内存通过 `__pkvm_host_donate_guest()` 从 Host EPT 中取消映射

#### 设备直通流程

```
1. host(KVM high) 发起 hypercall -> vmcall -> pkvm

2. pkvm_add_ptdev(shadow_vm_handle, bdf, pasid)
   -> pkvm_alloc_ptdev(bdf, pasid)
   -> pkvm_attach_ptdev(bdf, pasid, vm)

3. pkvm_attach_ptdev() 具体做
   -> pkvm_ptdev_cache_bar()            // 读取并缓存 6 个 BAR 值
   -> ptdev->pgt = &vm->pgstate_pgt     // 切换 iommu 页表到 VM 的页表
   -> pkvm_iommu_sync(bdf, pasid)       // 同步 shadow iommu 到硬件

4. 之后该设备的所有 DMA 走 pgstate_pgt
   pgstate_pgt 只映射了该 VM 拥有的内存
   -> 设备 DMA 无法访问其他 VM 或 host 的内存
```

配置空间保护 (pci.c)：Host 不能修改已直通设备的 BAR 地址
```c
static bool host_vpci_cfg_data_allow_write(ptdev, offset, size, value) {
    if (offset >= 0x10 && offset < 0x28) {  // BAR 区域
        return value == ptdev->bars[index];  // 只允许写回原始值
    }
}
```

#### 整体保护图

```
Host写IOMMU -> pKVM拦截审计 -> shadow生效 -> 硬件执行
Host直接写  -> EPT trap     -> hyp接管   -> Host以为生效了但硬件没动

```

### Host 访问 vCPU 状态

pKVM 用嵌套虚拟化(Nested VMX)模型。Host(L1) 通过 VMREAD/VMWRITE 操作 VMCS 访问 Guest(L2) 的 vCPU 状态，pKVM 拦截并过滤

```
pkvm(L0, 真正的 VMX root)
  -> 维护 vmcs02(真正写入硬件的VMCS)
  -> host 看到的是 vmcs12(缓存的虚拟VMCS)

host(L1, VMX non-root)
  -> VMREAD/VMWRITE -> vmexit -> pkvm 拦截
  -> Protected VM(L2, Guest)
```

VMCS 字段三级分类 (pkvm_nested_vmcs_fields.h)：

1. Emulated Fields：pKVM 完全控制
   - EPT_POINTER、IO_BITMAP、MSR_BITMAP、VM_EXIT_CONTROLS 等控制类字段
   - Host 写这些只记录到 cached_vmcs12，pKVM 在 vmresume 前翻译成真正写入 vmcs02 的值
   - 比如 Host 写 EPT_POINTER -> pKVM 替换成 shadow EPT

2. Shadow Fields：直接透传给硬件
   - GUEST_RIP、GUEST_RSP、GUEST_RFLAGS、GUEST_CR0/CR3/CR4 等寄存器
   - 硬件自动在 vmcs02 中 shadow，pKVM 不干预

3. Host Fields：Host 自己的状态，自由读写

安全边界：Host 能读写 Guest 的寄存器（RIP/RSP 等），但不能修改控制类字段（EPT/IO bitmap 等）

### Host 访问 Guest 内存

通过 Host EPT 做内存隔离，每个物理页有四种状态：
- NOPAGE：Host 看不到（Host EPT 中无映射）
- PAGE_OWNED：Host 独占
- PAGE_SHARED_OWNED：Host 和 Guest 共享
- PAGE_SHARED_BORROWED：Guest 借用的共享页

页面所有权转移：
```
初始：host 拥有所有内存
  -> __pkvm_host_donate_guest() -> guest 私有(host 看不到)
     -> __pkvm_host_undonate_guest() -> host 收回
  -> __pkvm_host_share_guest() -> host 和 guest 共享
     -> __pkvm_host_unshare_guest() -> host 收回
  -> host 独占(默认)
```

Host 访问 Guest 私有内存 -> Host EPT miss -> vmexit -> pKVM 拦截 -> 拒绝

### Host 访问直通的设备（安全性）

#### 当前保护
1. 配置空间审计：Host 不能修改直通设备的 BAR
2. IOMMU shadow 页表：设备 DMA 受 pKVM 控制
3. DMA 内存隔离：Guest 私有内存不会被映射到 IOMMU 域

#### 已知安全缺口 (FIXME/TODO)

缺口1 ptdev.c:160
pKVM 无法独立发现直通设备，依赖 Host 通过 ADD_PTDEV hypercall 上报。Host 可以隐瞒设备或伪造虚拟设备(virt device)
```c
/*
 * FIXME:
 * The passthrough devices attached to the protected VM is relying on KVM
 * high to send vmcall so that pKVM can know which device should be isolated.
 * But if KVM high has created a passthrough device for a protected VM without
 * using this vmcall to notify pKVM, pKVM should still be able to isolate this
 * passthrough device. To guarantee this, either needs pKVM to know the
 * passthrough devices information to isolate them independently or needs
 * protected VM to check with pKVM about its passthrough device info through
 * some vmcall. Currently neither way is available.
 */
```
攻击场景：
```
1. Host 创建一个 virtio (设备（软件模拟)
2. 告诉 Guest "这是一个直通设备"
3. Guest 把敏感数据写到设备的 buffer
4. Host 可以直接读取 buffer 内容
5. 这个 "设备" 没有走 ADD_PTDEV，pKVM 不知道它的存在
```

缺口2 (pkvm_hyp_types.h:120)
所有 vCPU 共享同一个 shadow EPT，如果一个 vCPU 进入 SMM 而其他没有，SMM 不同的内存访问权限可能导致内存泄漏
```c
/*
 * VM's shadow EPT. All vCPU shares one mapping.
 * FIXME: a potential security issue if some vCPUs are
 * in SMM but the others are not.
 */
struct shadow_ept_desc sept_desc;
```

缺口3 (ept.c:391)
Host EPT violation 时 pKVM 会映射 MMIO 范围，但没检查这段 MMIO 是否属于已直通给 Protected VM 的设备，Host 可能通过此途径操作 Guest 的设备
```c
/*
 * TODO:
 * check if this MMIO belongs to a secure VM pass-through device.
 */
```

缺口4 (pkvm.c:195)
pKVM 只控制了 primary vCPU 入口，secondary vCPU 的启动没有限制，Host 可以在 Guest 没准备好时让 secondary vCPU 运行
```c
/*
 * TODO: also need to prevent running secondary vCPUs
 * until the VM itself allows it (probably via a hypercall
 * to pKVM) at the moment when it boots a secondary vCPU.
 */
```

缺口5 (lapic.c:117)
x2APIC 模式下可以通过拦截 MSR 防止 APIC ID 被修改，但 xAPIC 模式下 MMIO 写入的 APIC ID 修改没拦截，可能导致中断路由错误
```c
/*
 * APIC_ID reg is writable for primary VM so it is
 * possible for primary VM to change the APIC_ID.
 * So pkvm should have a way to intercept the APIC_ID
 * changing. For x2apic mode, this can be done through
 * intercepting the APIC_ID msr write.
 *
 * TODO: handling the APIC_ID changing for xapic mode.
 */
```

缺口6 (iommu.c:2181)
bdf_pasid_to_iommu() 只在已同步设备列表里查找，从未同步过的设备找不到 IOMMU，后续隔离操作会失败
```c
/*
 * TODO:
 * Currently assume that the bdf/pasid has ever been synced
 * so that the IOMMU can be found. If not synced, then cannot
 * get a valid IOMMU by calling this function.
 *
 * To handle this case, pKVM IOMMU driver needs to check the
 * DMAR to know which IOMMU should be used for this bdf/pasid.
 */
```

缺口7 (commits 5afd02899447, 5e111cb61f60)
Protected VM 的内存页通过粗暴的 pin-all 方式防止被 Host 回收，存在重复计数和回收问题，正确方案应类似 TDX 的 file-based memory
```
REVERTME: HACK: kvm: pin all the pages used by protected VM

This is a hack implementation to pin every mappings created for a
protected VM. It is probably that one page can be counted multiple
times if KVM is switching EPT root during runtime. All these pinned
pages will be unpinned after this protected VM is destroyed.

For the proper solution, we would like to leverage the file based memory
solution used by TDX[1], which the discussion is still on-going at the
mailing list.
```

### v6.18commit 分类

代码结构变化：
```
v6.12:  arch/x86/kvm/vmx/pkvm/hyp/    (全部在hyp目录下)
v6.18:  arch/x86/kvm/pkvm/            (独立目录)
        arch/x86/kvm/pkvm/vmx/        (独立vmx)
        drivers/iommu/intel/pkvm/     (IOMMU从hyp拆出为独立驱动)
```

- 一、架构重构（pVMX 替代 Nested VMX）
Nested VMX 架构被替换为 pVMX + pkvm_x86_ops

```
5a1b9041bc41 pKVM: x86: Introduce pkvm_x86_ops
e1e8360d2372 pKVM: VMX: Implement pkvm_x86_ops
c4fd27f285d3 KVM: pVMX: Add initial pKVM host interface
9a930d9454c8 pKVM: x86: Add global initialization operation in pkvm_init_ops
83a88dcd0a47 pKVM: VMX: Implement hardware_setup operation
6cdd8df0e852 pKVM: VMX: Implement hyp_global_init operation
8f357dcc0ed5 pKVM: VMX: Implement vm_init operation
1582caab9b42 pKVM: VMX: Reprivilege host cpus if pkvm init fails
170d6bc851c3 x86/pkvm: Add dependency with CONFIG_PARAVIRT
```

- 二、IOMMU（PV IOMMU 架构，替代 v6.12 的 shadow IOMMU）
IOMMU 从 shadow 改为 PV ，ptdev 被移除，设备直通改走 PV IOMMU domain 管理

- 基础框架
pKVM IOMMU驱动桩代码、延迟初始化到pKVM启动后、domain管理框架搭建
```
2044deeec66e iommu/vt-d: pKVM: Driver stub and vendor hooks for IOMMU
551ba8854746 iommu/vt-d: pKVM: Delay IOMMU initialization to after pKVM init
047384cf336f iommu/vt-d: pKVM: IOMMU initialization during pKVM init
dd1cead55aef iommu/vt-d: pKVM: Introduce IOMMU domain framework
9eab585380c1 iommu/vt-d: pKVM: Disable pkvm unsupported features in host driver
250f73842bfe iommu/vt-d: pKVM: Force enable IOMMU if pKVM is enabled
017f6ff3fe4e iommu/vt-d: pKVM: Do not disable IOMMU on igfx_off
```

- MMIO 访问
host通过hypercall读写iommu寄存器，pKVM拦截审计GCMD等控制命令后执行
```
b7d3c3982776 iommu/vt-d: pKVM: Hypercall for IOMMU MMIO access
3a1cc23c2cec iommu/vt-d: pKVM: Hypercall wrappers for dmar_read/write
755477ea53e6 iommu/vt-d: pKVM: Unmap IOMMU MMIO space from host
c3468ac4d04f iommu/vt-d: pKVM: Framework for handling GCMD register access
a1d4b1facca2 iommu/vt-d: pKVM: Handle DMA_GCMD_SRTP
920d3cd88902 iommu/vt-d: pKVM: Handle DMA_GCMD_TE
bec143d4d3c9 pKVM: VMX: Do not handle ept violation on IOMMU MMIO access
```

- QI(Queue Invalidation)
将iommu队列失效机制移植到pkvm，Host通过hypercall提交失效描述符
```
7c0ba1255a61 iommu/vt-d: pKVM: Initialize Queued Invalidation (QI)
0d97f7729406 iommu/vt-d: pKVM: Statically allocate desc_status in qi_inval
4d6d18de2904 iommu/vt-d: pKVM: Port QI functionality to pKVM
1be5dacfef30 iommu/vt-d: pKVM: iommu_flush callbacks for pKVM
5aa7b22d26bb iommu/vt-d: pKVM: Hypercall for submitting QI descriptors
a095aa7f338c iommu/vt-d: pKVM: Replace qi_submit hypercall with iec_flush
```

- Context/PASID 表管理
Legacy/Scalable模式上下文表和PASID表更新hypercall，根表写保护
```
436ff3fc07bd iommu/vt-d: pKVM: Port legacy mode context table helpers to pKVM
edc3db0f8b44 iommu/vt-d: pKVM: Legacy mode context update hypercalls
112ac25afe12 iommu/vt-d: pKVM: Port scalable mode context table helpers to pKVM
55fa9d0e1751 iommu/vt-d: pKVM: Scalable mode context update hypercalls
7254d0321c27 iommu/vt-d: pKVM: Port pasid table update helpers to pKVM.
fde0edcd148c iommu/vt-d: pKVM: Pasid table hypercall for first level translation
fc5e826f2af8 iommu/vt-d: pKVM: Pasid table hypercall for second level translation
015652bb04bf iommu/vt-d: pKVM: Pasid table entry teardown hypercall
6b328edbcddc iommu/vt-d: pKVM: Write protect the Root Table Page
fbeeb833d1cc iommu/vt-d: pKVM: Prevent clearing context entry with active PASIDs
```

- Domain 页表管理
IOMMU domain页表创建/映射/释放hypercall，用bit63标记已映射PTE
```
21a07055c0bc iommu/vt-d: pKVM: Port domain page table handling code to pKVM
e1bb7d519f38 iommu/vt-d: pKVM: Hypercalls for iommu domain management
119a4e6a8273 iommu/vt-d: pKVM: Hypercalls for domain pagetable management
ea646ccd5379 iommu/vt-d: pKVM: Release the pagetable pages when freeing domain
a661d6091da5 iommu/vt-d: pKVM: Check for mappings when switching to superpage
35ce34774372 iommu/vt-d: pKVM: Use hardware ignored bit 63 to represent mapped PTE
```

- Cache 管理
IOTLB和context cache刷新管理，host侧避免不必要的flush，cache tag移植到pkvm
```
6e261a4619ed iommu/vt-d: pKVM: Port cache tag management logic to pkvm.
d2ca28a44e5d iommu/vt-d: pKVM: Move cache tag assign/unassign to pKVM
828a7e5ad3db iommu/vt-d: Adapt to pKVM handling of cache flushing.
c376f142184d iommu/vt-d: pKVM: Flag to denote if iotlb sync needed after map
f6132f0a8604 iommu/vt-d: Avoid iotlb flush when pKVM enabled
e62dbd6d804d iommu/vt-d: pKVM: Flush did FLPT_DEFAULT_DID for host ept flush
```

- 安全增强
阻止host捐赠DMA页、ATS设备白名单(SATC扫描)、host EPT页大小对齐约束
```
cb51d2d27ca8 iommu/vt-d: pKVM: Prevent host from donating DMA pages
66e88187ed6a pKVM: x86: Add API to pin/unpin host owned pages for DMA
62c6749c2088 iommu/vt-d: pKVM: Scan SATC for supporting devices using ATS
2302899e6ffa iommu/vt-d: pKVM: Advertise ATS only for devices in SATC
c144d392d28e pKVM: VMX: Consider iommu supported page size and level for host EPT
```

- 三、PCI 配置空间：通过hypercall拦截PCI MMCFG访问，防止Host直接操作设备配置空间

```
05804102b367 x86/paravirt: Extend pv_mmio_ops to support pci mmcfg read and write
a285b036489e x86/pkvm: Use hypercall to access the pci mmcfg
```

- 四、APIC 保护（v6.18 新增）
APIC 保护全新加入，拦截x2APIC MSR、保护APIC page、强制TPR和中断注入审计

```
e091271f1090 pKVM: VMX: Require the APICv feature
74a122c01466 pKVM: x86: Protect apic page for pVMs
d5daeb320418 pKVM: x86: Enforce the TPR to 0x10 for the protected APIC
37c33782ff1a pKVM: x86: Set the pVM's lapic in x2apic mode by default
8607a3e96c72 pKVM: x86: Expose kvm_apic_set_base to the pKVM
5b68c7f5a7f8 pKVM: x86: Emulate the MSR_IA32_APICBASE for pVM in the pKVM
a576b3a6f5e7 pKVM: VMX: Intercept MSR reading for protected apic
3a74d6532c78 pKVM: x86: Remove protected x2apic MSRs from the host emulated MSR list
2c8428ac076c pKVM: x86: Introduce shared_lapic_regs in pkvm_vcpu structure
18f9b0ffd002 pKVM: VMX: Handle EXIT_REASON_APIC_WRITE by sharing self-IPI/ICR regs
97f3e69da878 pKVM: x86: Sync PIR to IRR before entering guest in the pKVM
c364ad7f3429 pKVM: x86: Expose two IRR related functions to the pKVM
4af52fd8e9a0 pKVM: VMX: Align vmx_sync_pir_to_irr with the host KVM implementation
10f2e968339d pKVM: x86: Remove __pkvm__sync_pir_to_irr PV interface
ccd1138a4545 pKVM: x86: Add protected_apic_has_interrupt PV interface
e101b5775849 pKVM: x86: Update apicv in the pKVM
8748b091c028 pKVM: VMX: Respect the host KVM's enable_apicv/enable_ipiv params
fa3c2ecdcb38 pKVM: x86: Make __pkvm__set_virtual_apic_mode for the npVM only
dd4bedaed2a4 pKVM: VMX: Remove the apic mode check when set virtual apic mode
0216f905ca70 pKVM: x86: Make __pkvm__hwapic_isr_update for non-protected APIC only
59e69e05b041 pKVM: x86: Fix max_isr truncation bypass in hwapic_isr_update
6ce95b86e76b pKVM: x86: Expose the kvm_lapic_reset to the pKVM
8a724a4bd0e7 pKVM: x86: Enable lapic reset in the pKVM
14b06d159c37 SQUASHME: pKVM: x86: Disallow the host to inject software interrupts to a pVM
```

- 五、硬件特性隔离（v6.18 新增）  
硬件特性隔离全新加入（Intel PT/LBR/MPX/CET）

防止 Host 通过 CPU 调试/追踪特性窥探 Guest

```
5e15003f85c1 pKVM: VMX: Enforce Intel PT isolation
f778d64713a1 pKVM: VMX: Enforce architectural LBR isolation
b4739a795e34 pKVM: VMX: Enforce MPX isolation
53839abd9e30 pKVM: VMX: Enforce supervisor mode CET isolation
5ffe84fbdf82 pKVM: VMX: Introduce a fixup mechanism for the host vCPU
314138903e37 SQUASHME: pKVM: VMX: Do fixup before switching spec ctrl
93dd649e305f pKVM: VMX: Switch the spec ctrl MSR when switching to/from the host
1c8f63f7bf7d SQUASHME: pKVM: VMX: Set VM_ENTRY_LOAD_BNDCFGS to satisfy VMCS pair checks

- 六、内存管理
Guest内存共享、页钉住(pin)、pvmfw内存区域保护和页引用泄漏修复

```
8b3ab39c8952 x86/pkvm: Add pkvm guest memory share support
66e88187ed6a pKVM: x86: Add API to pin/unpin host owned pages for DMA
621b8f64ce56 pKVM: x86: Protect pvmfw memory region from the host
c6321af4e11b KVM: x86/mmu: pKVM: Fix page reference leak in pkvm_page_fault()
c6f219bc0d4e pKVM: x86: Clamp negative mmu_age error codes to zero for host
c144d392d28e pKVM: VMX: Consider iommu supported page size and level for host EPT
```

- 七、vCPU 和 TLB 管理
vCPU模式设置、TLB刷新等待机制、CPU索引优化和use-after-free修复

```
2196684199c2 pKVM: x86: Create helpers to set vcpu mode
7101e8baec54 pKVM: VMX: Fix setting for the host vcpu->mode
5ab1030e97b0 pKVM: x86: Add helper to wait vCPU kicked out
c5a77a405244 pKVM: VMX: Wait guest vCPU kicked out after flushing TLB
e4d5560331c5 pKVM: VMX: Wait host vCPU kicked out after flushing TLB
c1a901671fde pKVM: x86: Index host_vcpus and pcpus via consecutive number
d5122a4a3306 pKVM: x86: Add for_each_pkvm_initialized_cpu macro
fde78d6833a9 pKVM: x86: Add helper to check if CPU is initialized or not
c7bbf8de3166 pKVM: VMX: Do not flush host EPT if CPU is not initialized
9a7f23f37826 pKVM: x86: Introduce macro for_each_pkvm_guest_vcpu
a46caf84e739 pKVM: x86: Re-check pending requests before vmenter guest
ea384e5f5423 pKVM: x86: Handle the KVM_REQ_TLB_FLUSH_GUEST request
987a8203c0c6 pKVM: x86: Fix use-after-free race in pkvm_put_vcpu()
```

- 八、Panic/调试基础设施（v6.18 新增）
Panic/调试基础设施全新加入

```
119d69b3f855 pKVM: x86: Add hypervisor panic and reset infrastructure
6843ce75a5c3 pKVM: x86: Implement hyp panic logging to ramoops console
e9494f2c7d00 pKVM: x86: Dynamically discover ramoops from platform bus
e29523019549 pKVM: x86: Map ramoops memory into hypervisor page table
37709eb09a5e pKVM: x86: Hook fatal hardware exceptions to hypervisor panic
0faf749b44d1 pKVM: x86: Implement lightweight scnprintf for hyp diagnostics
c088cf66eefa pKVM: x86: Implement pkvm_udelay using hardware TSC
eb1358a9df26 pKVM: x86: Enforce calibrated TSC during initialization
9f0d7b717c64 pKVM: x86: trace: Register per guest VM vmexit trace debug file
73c7dc6ca479 pKVM: x86: Enlarge unmapped vmemmap hole to catch huge struct dereferences
```

- 九、pvmfw（Protected VM Firmware）
pVM固件安全加载、入口点强制、secondary vCPU安全启动和内存保护

```
aa55aab317f4 pKVM: x86: Parse pvmfw memory region info from the bootloader
18ee4fa7df66 pKVM: x86: Enforce pvmfw as the pVM entry point
94b2ecc1f121 pKVM: x86: Enforce secure pvmfw bootstrap
5b6f64c6ae31 pKVM: x86: Implement secure startup of secondary vCPUs
621b8f64ce56 pKVM: x86: Protect pvmfw memory region from the host
```

- 十、PV 接口（Host 半虚拟化）
Host通过PV接口调用vcpu_run、mmu_pgd等操作，pKVM审计并执行

```
d1a400ff2017 pKVM: x86: Add vcpu_run PV interface
01d1296d3072 pKVM: x86: Implement vm_finalize PV interface
fc1a12c3866e pKVM: x86: Add load_mmu_pgd PV interface
969a62812770 pKVM: x86: Validate segment index in segment PV interfaces
5e2c2c261f21 pKVM: x86: Audit PV interfaces for protected VMs
436538eea9ac pKVM: x86: Add has_wbinvd_exit PV interface
95974250b2c6 pKVM: x86: Add complete_emulated_msr PV interface
0bfaaa55db81 x86/pkvm: Use hypercall to start CPUs
```

### v6.18 总结

- 主要成果
v6.18 做了三个大的变化：
1. iommu从shadow页表改为PV架构(host通过hypercall操作iommu，pkvm审计执行)，代码从hyp/拆成独立驱动drivers/iommu/intel/pkvm/
2. Nested VMX替换为pVMX + pkvm_x86_ops
3. 新加入APIC保护(~25 commits)、硬件特性隔离(Intel PT/LBR/CET)、Panic调试基础设施

- 已知缺口
v6.12 的 ptdev.c/h、pci.c/h、nested.c/h、vmexit.c/h、io_emulate.c/h 全部被删除，但部分功能没有等价替代：

1. 运行时设备分配不支持 (mmu.c:1769)，对应 v6.12 缺口 1：v6.12 至少有 ADD_PTDEV hypercall 可以追踪设备，只是 Host 可以选择不调用；v6.18 直接删了 ptdev 整套机制，连 "这个设备属于哪个 VM" 的追踪都没有了，Host 伪造 virt device 骗 Guest 的攻击场景依然完全可行
```c
/*
 * not tracked in pkvm's vmemmap and thus have no refcount. This is ok
 * as long as pKVM doesn't support device assignment, so it never
 * donates MMIO pages at runtime.
 *
 * FIXME: this assumes that any such reserved memory regions are
 * in holes between memory memblocks (so they have been already
 * mapped in the host MMU with PKVM_PAGE_OWNED by pkvm_host_mmu_init())
 * which is generally not guaranteed.
 * To fix this cleanly, we could e.g. rework pKVM to set PKVM_ID_HOST
 * (instead of PKVM_ID_HYP) as the initial owner for those MMIO pages
 * that are allowed to be used by the host but haven't been lazily
 * mapped in the host MMU yet (while still keep their state as
 * PKVM_PAGE_NONE until they are lazily mapped), to let pKVM easily
 * distinguish between them and those MMIO pages that are not allowed
 * to be mapped for the host (e.g. IOMMU MMIO).
 */
```

2. PCI 配置空间保护不足：v6.12 有完整的 BAR 写审计（host_vpci_cfg_data_allow_write），v6.18 只剩 2 个 MMCFG hypercall，缺少直通设备 BAR 保护

3. IOMMU domain 硬编码 (domain.c:12)
```c
/*
 * TODO: Make this a dynamic value.
 */
#define MAX_IOMMU_DOMAIN_NUM	128
```

4. 安全 TSC 未保护 (pkvm.c:898)
```c
/*
 * TODO: As the pVM can use another secure time source, the
 * guest TSC is allowed for the host to emulate and access. To
 * support the pVM with secure TSC, add protection for TSC
 * related PV interfaces.
 *	__pkvm__write_tsc_offset
 *	__pkvm__write_tsc_multiplier
 */
```

5. 大页合并不支持 (pgtable.c:249)
```c
/* TODO: Support merging small entries into a huge entry. */
```

6. Host MMIO 映射页不足无回收 (ept.c:453)
```c
/*
 * TODO: In case the host mmu free pages are not
 * enough, -ENOMEM will be returned. This could be
 * handled by unmaping some other MMIO mapped for the
 * host VM to reclaim some mmu pages and try again.
 */
```

7. SATC 物理功能未验证 (iommu_hc.c:49)
```c
/*
 * Device is in SATC and optimistically assuming that a well crafted SATC
 * would contain only physical functions, its safe to set pfsid = bdf.
 * TODO: We should probably be verifying SATC for existence of only
 * physical functions during pkvm initialization.
 */
```

8. 用户态 VMM MSR 处理 (pkvm.c:1012)
```c
/*
 * TODO: The user space VMM from the host side (e.g.,
 * crosvm) may still try to set these MSRs which are
 * protected by the pKVM hypervisor for a pVM. Ignore
 * writings to these MSRs and return 0 to make such
 * user space VMM happy, meanwhile doesn't really modify
 * these MSRs. This eventually will be fixed in the user
 * space VMM to avoid doing so for a pVM. Once this is
 * implemented, these can be removed.
 */
```

9. MMIO 捐赠初始化假设 (mmu.c:1200)
```c
/*
 * NOTE: another hacky assumption is that this API should only be used during
 * pKVM initialization, not at runtime. Otherwise it doesn't ensure protection
 * of these MMIO pages from the host DMA (see pkvm_host_use_dma()).
 * TODO: clean this mess.
 */
```

10. 链接脚本对齐未确认 (pkvm.lds.S:8)
```c
/*
 * FIXME:
 * According to arch/x86/kernel/vmlinux.lds.S, the section
 * ".text..__x86.rethunk_untrain" starts at 2M aligned address,
 * followed by ".text..__x86.rethunk_safe" section. Since those two
 * sections for pKVM are merged in .pkvm.text below, need to check
 * whether the same alignment should be enforced.
 */
```

11. XSAVE 无 CPU 场景未验证 (host_vmx.c:241)
```c
/*
 * TODO: CPUID.01H:ECX.XSAVE[bit 26] == 0 will also result in
 * #UD but all modern Intel CPUs have XSAVE. If the pKVM runs
 * on such CPU without XSAVE, verify if this #UD also has
 * priority over the interception.
 */
```

12. 栈保护器未实现 (Makefile:5)
```makefile
# TODO: make the stack protector work with the pKVM hypervisor code
```

13. 10 个 SQUASHME commit：表明多个功能仍为临时实现，未最终定稿
