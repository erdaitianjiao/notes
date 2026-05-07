# VFIO讲解

## 结构体
### 结构体定义
```c
// VFIO设备
struct vfio_device {
    struct device *dev;                    // 关联的物理设备
    const struct vfio_device_ops *ops;     // 设备操作回调
    const struct vfio_migration_ops *mig_ops;  // 迁移操作
    const struct vfio_log_ops *log_ops;    // 日志操作

    struct vfio_group *group;              // 所属的组
    struct list_head group_next;           // 组内设备链表
    struct list_head iommu_entry;          // IOMMU设备链表

    struct vfio_device_set *dev_set;       // 所属设备集合
    struct list_head dev_set_list;         // 集合内链表

    unsigned int open_count;               // 打开计数
    struct iommufd_access *iommufd_access; // IOMMUFD访问
    struct iommufd_device *iommufd_device; // IOMMUFD设备
    struct kvm *kvm;                       // 关联的KVM
    // ...
};

// 设备集合
struct vfio_device_set {
    void *set_id;                         // 集合ID
    struct mutex lock;
    struct list_head device_list;
    unsigned int device_count;
};

// VFIO组
struct vfio_group {
    struct device dev;
    struct cdev cdev;
    refcount_t drivers;                 // 驱动引用计数
    unsigned int container_users;       // 容器用户数
    struct iommu_group *iommu_group;    // 关联的IOMMU组
    struct vfio_container *container;   // 所属容器
    struct list_head device_list;       // 组内设备列表
    struct mutex device_lock;
    struct list_head vfio_next;         // 全局组列表
    enum vfio_group_type type;          // 组类型
    struct mutex group_lock;
    struct kvm *kvm;                    // 关联的KVM
    struct iommufd_ctx *iommufd;        // IOMMUFD上下文
    // ...
};

enum vfio_group_type type {
    VFIO_IOMMU;             // 物理设备，有IOMMU支持
    VFIO_EMULATED_IOMMU;    // 虚拟设备，软件模拟IOMMU
    VFIO_NO_IOMMU;          // 物理设备但无IOMMU（危险模式）
};

struct vfio_container {
    struct kref kref;
    struct list_head group_list;             // 容器内的组列表
    struct rw_semaphore group_lock;
    struct vfio_iommu_driver *iommu_driver;  // IOMMU驱动
    void *iommu_data;                        // IOMMU驱动私有数据 一般指向
    bool noiommu;
};

struct vfio_iommu {
    struct list_head domain_list;       // IOMMU域列表
    struct list_head iova_list;         // IOVA区间列表
    struct mutex lock;
    struct rb_root dma_list;            // DMA映射红黑树 
    struct list_head device_list;       // 设备列表
    unsigned int dma_avail;
    uint64_t pgsize_bitmap;             // 支持的页大小
    bool v2;                            // API版本
    bool dirty_page_tracking;           // 脏页跟踪
};

struct vfio_dma {
    struct rb_node node;                // 红黑树节点
    dma_addr_t iova;                    // IOVA (设备地址)
    unsigned long vaddr;                // 进程虚拟地址
    size_t size;                        // 映射大小
    int prot;                           // 保护标志(IOMMU_READ/WRITE)
    bool iommu_mapped;                  // 是否已建立IOMMU映射
    struct task_struct *task;           // 所属任务
    struct rb_root pfn_list;            // PFN列表
    struct mm_struct *mm;               // 内存描述符
};

struct vfio_pfn {
    struct rb_node node;
    dma_addr_t iova;        // IOVA
    unsigned long pfn;      // Host PFN (HPA)
    unsigned int ref_count; // 引用计数
};

struct vfio_domain {
    struct iommu_domain *domain;    // IOMMU域
    struct list_head next;          // 域列表
    struct list_head group_list;    // 域内的组
    bool enforce_cache_coherency;
};

```

### 各结构体作用

#### 核心结构体

**vfio_device** - 单个物理设备的VFIO抽象
- 每个PCI设备对应一个vfio_device
- 包含设备操作回调、所属group、关联的KVM等

**vfio_group** - iommu_group的VFIO封装
- 由硬件物理拓扑决定，无法改变
- 一个group可以包含多个device（如同一个PCIe设备下的多个Function）

**vfio_container** - 用户空间的IOMMU上下文
- 用户空间打开`/dev/vfio/vfio`时创建
- 多个group可以加入同一个container，共享同一个IOVA地址空间

#### DMA映射相关结构体

**vfio_iommu** - VFIO层的IOMMU管理
- 由container的`iommu_data`指向
- 管理DMA映射：维护`dma_list`红黑树
- 管理IOVA地址空间
- 管理多个`vfio_domain`（多IOMMU硬件场景）

**vfio_dma** - 单段IOVA→HPA的DMA映射区域
- `iova`: Guest物理地址（设备看到的地址）
- `vaddr`: QEMU进程虚拟地址
- `pfn_list`: 该区域内各页的IOVA→PFN映射

**vfio_pfn** - 单页的IOVA→HPA映射
- `iova`: IOVA地址
- `pfn`: Host物理页帧号(HPA)
- `ref_count`: 引用计数

**vfio_domain** - 硬件IOMMU域封装
- 封装Linux的`iommu_domain`
- 直接与硬件IOMMU交互，建立/删除硬件页表映射

#### 设备集合结构体

**vfio_device_set** - 协同操作的设备集合
- 管理需要协同操作的设备组（如SR-IOV的PF/VF）
- 核心机制：提供全局锁保护open/close操作

### 结构体关系

#### 整体架构

VFIO分为三层抽象：Container（容器）→ Group（组）→ Device（设备）

```
Container (1) ──包含──> Group (1-N) ──包含──> Device (1-N)
     │                    │                     │
     │                    │                     └─► 物理设备的VFIO抽象
     │                    └─► iommu_group封装，由硬件拓扑决定
     └─► 用户空间的IOMMU上下文，提供IOVA地址空间
```

#### 串联关系图

```
vfio_container
    ├─► group_list ──► vfio_group ──► device_list ──► vfio_device
    │                                           └─► dev_set ──► vfio_device_set
    │
    └─► iommu_data ──► vfio_iommu
                          ├──► domain_list ──► vfio_domain ──► iommu_domain
                          │                                    (1对1封装)
                          └──► dma_list ──► vfio_dma ──► pfn_list ──► vfio_pfn
```

#### 关键关系表

| 关系 | 类型 | 谁决定 | 能否改变 |
|------|------|--------|----------|
| device → group | 多对1 | 硬件拓扑(PCIe ACS、SR-IOV等) | 不能 |
| group → container | 多对1 | 用户空间(ioctl) | 可以 |
| vfio_iommu → vfio_domain | 1对多 | 硬件IOMMU实例数量 | - |
| iommu_domain → iommu_group | 1对多 | 软件选择 | 可以共享 |
| vfio_domain → iommu_domain | 1对1 | 封装关系 | - |

#### domain 的本质

**iommu_domain = 一套IOVA页表 + 一个IOVA地址空间 + 一个IOMMU硬件槽位**

- 一个group使用一个domain = 使用这套页表进行地址翻译
- 多个group共享同一个domain = 共享同一套页表、同一个IOVA地址空间
- container中domain的数量取决于group所属的IOMMU硬件是否兼容

#### 层次职责划分

| 层次 | 结构体 | 职责 |
|------|--------|------|
| 用户接口层 | vfio_container | 提供/dev/vfio/vfio接口，管理group列表 |
| 设备管理层 | vfio_group/vfio_device | 设备隔离、设备生命周期管理 |
| 软件管理层 | vfio_iommu/vfio_dma | 维护IOVA→HPA映射元数据 |
| 硬件操作层 | vfio_domain/iommu_domain | 操作硬件IOMMU页表 |

## 流程

### 设备注册（内核探测时）

```c
vfio_pci_probe(pci_dev)
{
    // 分配并初始化 vfio_device
    vfio_device = vfio_alloc_device(...);

    // 注册设备，创建 vfio_group
    vfio_register_group_dev(vfio_device) {
        // 获取设备的 iommu_group（由硬件拓扑决定）
        iommu_group = iommu_group_get(dev);

        // 创建或获取 vfio_group
        vfio_group = vfio_group_get_from_iommu(iommu_group);

        // 创建设备节点 /dev/vfio/$GROUP
        // 创建设备节点 /dev/vfio/devices/$DEVICE
    }
}
```

### 用户空间初始化（QEMU启动时）

```c
QEMU 初始化
{
    // 1. 打开容器，创建 vfio_container
    container_fd = open("/dev/vfio/vfio");

    // 2. 设置 IOMMU 类型，创建 vfio_iommu
    ioctl(container_fd, VFIO_SET_IOMMU, VFIO_TYPE1_IOMMU) {
        // 分配 vfio_iommu
        // 初始化 dma_list 红黑树
        // 初始化 domain_list 链表
    }

    // 3. 打开组，获取 vfio_group
    group_fd = open("/dev/vfio/26");

    // 4. 将组加入容器
    ioctl(group_fd, VFIO_GROUP_SET_CONTAINER, container_fd) {
        // group->container = container
        // 将 group attach 到 vfio_iommu 的 domain
    }

    // 5. 获取设备 fd
    device_fd = ioctl(group_fd, VFIO_GROUP_GET_DEVICE_FD, "0000:00:02.0");

    // 6. mmap 映射 BAR 空间到用户空间
    mmap(NULL, size, PROT_READ|PROT_WRITE, MAP_SHARED, device_fd, region_offset);
}
```

### DMA映射流程

```c
用户空间调用 VFIO_IOMMU_MAP_DMA
{
    // 参数准备
    map.iova  = 0x10000000;      // Guest 物理地址（设备看到的地址）
    map.vaddr = guest_memory_va;  // QEMU 进程的虚拟地址（Guest内存）
    map.size  = 1MB;

    ioctl(container_fd, VFIO_IOMMU_MAP_DMA, &map) {

        // 1. 创建 vfio_dma，记录映射区域
        vfio_dma_do_map() {
            dma->iova  = 0x10000000;
            dma->vaddr = guest_memory_va;
            dma->size  = 1MB;

            // 2. pin 用户内存页，获取 HPA
            vfio_pin_pages_remote(dma) {
                // 根据 vaddr 获取物理页
                // pin_user_pages_remote()
                // 返回 HPA，如 0xABC000
            }

            // 3. 将 IOVA→PFN 记录到 pfn_list
            vfio_add_to_pfn_list(dma) {
                vfio_pfn->iova = 0x10000000;
                vfio_pfn->pfn  = 0xABC;
                // 加入 dma->pfn_list 红黑树
            }

            // 4. 在硬件 IOMMU 中建立映射
            vfio_iommu_map() {
                iommu_map(domain, iova=0x10000000, paddr=0xABC000, size=1MB) {
                    // 写入硬件 IOMMU 页表
                    // IOVA 0x10000000 → HPA 0xABC000
                    // 刷新 IOTLB
                }
            }

            // 5. 将 vfio_dma 加入索引
            vfio_link_dma(iommu, dma) {
                // 加入 vfio_iommu->dma_list 红黑树
            }
        }
    }
}
```

### 设备使用流程

```c
Guest OS 访问设备
{
    // 1. 写设备寄存器（通过 BAR）
    //    直接访问 QEMU mmap 的 BAR 内存空间
    write_bar_register() {
        // QEMU 的虚拟地址
        // 直接对应设备 MMIO
        // 到达物理设备寄存器
    }

    // 2. 设备发起 DMA
    device_dma_transfer() {
        // 设备使用 IOVA 作为 DMA 地址
        dma_addr = 0x10000000;  // GPA

        // 经过硬件 IOMMU
        // IOMMU 查页表: 0x10000000 → 0xABC000

        // 直接访问 Host 物理内存
        // 最终写入 HPA 0xABC000
    }

    // 3. 设备中断
    device_trigger_interrupt() {
        // VFIO 驱动收到中断
        // eventfd_signal() 通知 KVM
        // KVM 注入中断到 Guest
        // Guest 处理中断
    }
}
```

### DMA取消映射流程

```c
用户空间调用 VFIO_IOMMU_UNMAP_DMA
{
    unmap.iova = 0x10000000;
    unmap.size = 1MB;

    ioctl(container_fd, VFIO_IOMMU_UNMAP_DMA, &unmap) {

        vfio_dma_do_unmap() {
            // 1. 从 dma_list 查找对应的 vfio_dma
            dma = vfio_find_dma(iommu, iova=0x10000000, size=1MB);

            // 2. 在硬件 IOMMU 中删除映射
            iommu_unmap(domain, dma->iova, dma->size) {
                // 删除硬件页表项
                // 刷新 IOTLB
            }

            // 3. unpin 内存页，释放引用
            vfio_unpin_pages_remote(dma) {
                // 遍历 pfn_list
                // unpin_user_page()
                // 释放 vfio_pfn
            }

            // 4. 释放 vfio_dma
            vfio_unlink_dma(iommu, dma);
            kfree(dma);
        }
    }
}
```

### IOVA页表维护

```c
IOVA → HPA 的查找过程
{
    // 设备发起 DMA，使用 IOVA 0x10000000

    // 硬件 IOMMU 查页表（硬件自动完成）
    // 硬件: 0x10000000 → 0xABC000 → 直接访问内存

    // ===== 软件维护的数据结构 =====

    // 1. vfio_iommu->dma_list（红黑树）
    //    按 iova 索引所有 vfio_dma
    dma = vfio_find_dma(iommu, iova=0x10001000, size=4096) {
        // 在红黑树中查找包含该 iova 的 vfio_dma
        // 返回: dma->iova=0x10000000, dma->size=1MB
    }

    // 2. vfio_dma->pfn_list（红黑树）
    //    按 iova 索引该区域内的每一页
    vpfn = vfio_find_vpfn(dma, iova=0x10001000) {
        // 在 pfn_list 中查找该页
        // 返回: vpfn->iova=0x10001000, vpfn->pfn=0xABD
    }

    // 3. pfn → HPA
    //    HPA = vpfn->pfn << PAGE_SHIFT
    hpa = 0xABD << 12;  // = 0xABD000
}
```

### Q: VFIO如何让Host OS放弃设备，让Guest OS使用？

#### 设备控制权转移原理

```
Host OS 正常状态
├── PCI 设备绑定到原生驱动 (如 e1000e, igb)
└── Host OS 控制设备

用户手动操作
├── 1. 解绑原生驱动
│   echo "0000:00:02.0" > /sys/bus/pci/devices/0000:00:02.0/driver/unbind
│
└── 2. 绑定到 vfio-pci
    echo "vfio-pci" > /sys/bus/pci/devices/0000:00:02.0/driver_override
    echo "0000:00:02.0" > /sys/bus/pci/drivers_probe

Guest OS 使用状态
├── PCI 设备绑定到 vfio-pci 驱动
├── vfio-pci 不控制设备，只提供透传接口
└── Guest OS 驱动直接控制设备
```

#### vfio-pci 的特殊之处

```c
vfio-pci 驱动
{
    // 不做任何实际的设备控制
    // 只是提供接口让用户空间访问设备

    open()  { /* 允许用户打开设备 */ }

    mmap()  { /* 将 BAR 空间映射到用户空间 */ }

    ioctl() { /* 提供设备配置、中断等接口 */ }

    // 注意：vfio-pci 不会初始化设备、不会启动设备
    //       这些都由 Guest 驱动完成
}
```

#### 控制权对比

| 状态 | 绑定驱动 | 谁控制设备 |
|------|----------|-----------|
| Host正常使用 | 原生驱动 (e1000e等) | Host OS |
| 绑定vfio-pci后 | vfio-pci | **无人控制**（仅透传） |
| Guest使用时 | vfio-pci + Guest驱动 | Guest OS |

#### Guest 如何获得控制权

```c
Guest OS 启动
{
    // Guest 驱动探测到 PCI 设备（通过 QEMU 模拟的 PCI 根总线）

    // Guest 驱动初始化设备
    guest_driver_init() {
        // 写 PCI 配置空间
        write_pci_config() {
            // 直接访问 BAR，到达物理设备
        }

        // 初始化设备寄存器
        write_device_register() {
            // 通过 vfio mmap 的 BAR 空间
        }

        // 申请中断
        request_irq() {
            // 通过 VFIO eventfd 机制
        }
    }

    // Guest 驱动完全控制设备
}
```

#### 核心机制

1. **Host OS放弃**: 将设备从原生驱动解绑，绑定到vfio-pci
2. **vfio-pci不控制**: 只是透传接口，不干预设备操作
3. **Guest完全控制**: Guest驱动通过VFIO直接操作物理设备




todo 
1. vfio_group的字段container_next
2. device的bar空间，PCI config space
3. iommu和domain的关系
4. gpu io-device的架构
5. 