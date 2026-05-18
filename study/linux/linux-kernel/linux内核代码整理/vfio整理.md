# vfio

## 一、结构体
```c
struct vfio_devices {
    struct device *dev;
    const struct vfio_device_ops *ops;

};
struct 


// 复位相关
struct vfio_device_set {};  

// pcie相关结构体
struct vfio_pci_core_device {
    struct vfio_device   vdev;         
    struct pci_dev       *pdev;        // PCI 设备指针
    void __iomem         *barmap[6];   // BAR 映射
    u8                   *vconfig;     // 虚拟配置空间
    u8                   msix_bar;     // MSI-X BAR 编号
    u16                  msix_size;    // MSI-X 表大小
    bool                 has_vga:1;    // 是否是 VGA 设备
    // ...
};
```

## 二、流程

### 初始化函数
内核初始化vfio驱动的时候会地用这个函数
```c
int __init vfio_pci_init(void)
{
    int ret;
    is_diable_vga = disable_vga

    // 设置了几个参数，应该是关于 vga 和 idle的
    vfio_pci_core_set_params(nointxmask, is_disable_vga, disable_idle_d3);

    // 注册 PCI 驱动，触发 probe 所有匹配的设备
    ret = pci_register_driver(&vfio_pci_driver);

    vfio_pci_fill_ids();

    return ret;
}

// pci结构体定义
static struct pci_driver vfio_pci_driver = {
    .name           = "vfio-pci",
    .id_table       = vfio_pci_table,       // 匹配的pci的表
    .probe          = vfio_pci_probe,       // Probe函数，有新的pci的设备后会调用
    .remove         = vfio_pci_remove,
    .sriov_configure    = vfio_pci_sriov_configure,
    .err_handler        = &vfio_pci_core_err_handlers,
    .driver_managed_dma = true,
};
```

### probe函数
设备被vfio-pci绑定的时候调用

```c
static int vfio_pci_probe(struct pci_dev *pdev, const struct pci_device_id *id)
{
    struct vfio_pci_core_device *vdev;
    int ret;

    // 检查是否在拒绝列表中
    if (vfio_pci_is_denylisted(pdev))
        return -EINVAL;

    // 分配一个 vfio_device
    vdev = vfio_alloc_device(vfio_pci_core_device, vdev, &pdev->dev,
                 &vfio_pci_ops);

    dev_set_drvdata(&pdev->dev, vdev);
    vdev->pci_ops = &vfio_pci_dev_ops;

    // 注册设备到vfio框架
    ret = vfio_pci_core_register_device(vdev);
    if (ret)
        goto out_put_vdev;
    return 0;

out_put_vdev:
    vfio_put_device(&vdev->vdev);
    return ret;
}

```

### 核心注册
```c
int vfio_pci_core_register_device(struct vfio_pci_core_device *vdev)
{
    struct pci_dev *pdev = vdev->pdev;
    struct device *dev = &pdev->dev;
    int ret;

    // 分配设备集 (用于复位管理)
    if (pci_is_root_bus(pdev->bus) || pdev->is_virtfn) {
        // 在 root上 或者是 sr-iov 中的 vf
        ret = vfio_assign_device_set(&vdev->vdev, vdev);  
    } else if (!pci_probe_reset_slot(pdev->slot)) {
        // 如果在某个slot上
        ret = vfio_assign_device_set(&vdev->vdev, pdev->slot);  
    } else {
        // 没有slot就整个pcie bus
        ret = vfio_assign_device_set(&vdev->vdev, pdev->bus);  
    }

    // 初始化 VF 和 VGA
    ret = vfio_pci_vf_init(vdev);  
    ret = vfio_pci_vga_init(vdev);  

    //  注册到 VFIO group
    ret = vfio_register_group_dev(&vdev->vdev);  
    if (ret)
        goto out_power;
    return 0;
}

int vfio_register_group_dev(struct vfio_device *device)
{
    return __vfio_register_dev(device, VFIO_IOMMU);
}
```

### vfio核心注册
```c
static int __vfio_register_dev(struct vfio_device *device,
                                enum vfio_group_type type)
{
    int ret;

    // 如果没有设备集，分配一个
    if (!device->dev_set)
        vfio_assign_device_set(device, device);

    // 设置设备名称
    ret = dev_set_name(&device->device, "vfio%d", device->index);

    // 核心 设置 group，创建 /dev/vfio/$GROUP_ID
    ret = vfio_device_set_group(device, type);
    if (ret)
        return ret;

    // 添加设备到设备模型，创建 /dev/vfio/devices/...
    ret = vfio_device_add(device);

    // 注册到 group
    vfio_device_group_register(device);

    return 0;
}

```

### 手动解绑和绑定设备过程

手动解绑的sysfs接口
```bash
cat /sys/bus/pci/devices/0000:01:00:0/driver/module

echo 0000:01:00:0 > /sys/bus/pci/driver/xxx/unbind

echo 0000:01:00:0 > /sys/bus/pco/drivers/vfio-pci/bind
```

- 绑定时候触发的流程

```c
driver_probe_device()
vfio_pci_probe();
vfio_pcie_core_register_device();
vfio_register_group_dev();
vfio_device_set_group();
```
然后会在/dev下创建 /dev/vfio/$group_id /dev/vfio/devices/vfioX

- 节点说明
/dev/vfio/vfio container节点
/dev/vfio/1    Group设备
/dev/vfio/devices/vfio0  设备文件

### qemu设备初始化

调用栈
```c
vfio_pci_realize() {
    // 打开设备 设置group container
    vfio_device_attach() {
        vfio_device_attach_by_iommu_type() {
            ops->attach_device(); // 两种可能
            // Legacy路径
            vfio_legacy_attach_device() {
                vfio_group_get() {
                    vfio_container_connect() {
                        vfio_create_container() {
                            vfio_set_iommu();
                        }
                        vfio_container_group_add();
                    }
                }
                vfio_device_get();
            }
            // Iommufd
        }
    }
    // 从内核获取region信息 bar大小等
    vfio_pci_populate_device();
    
    // 读配置空间，准备Bar 注册bar mmap bar
    vfio_pci_config_setup() {
        // 遍历bar
        vfio_region_setup() {
            // ioctl获取region大小 偏移 flag
            vfio_device_get_region_info();
            // 创建iobar
            memory_region_init_io();

            if (region->flags & VFIO_REGION_INFO_FLAG_MMAP) {
                // 如果支持mmap 计算mmap区间
                vfio_setup_region_sparse_mmaps(region, info, errp);
            }

        }
    }
    vfio_pci_config_setup() {
        // 读取设备空间
        vfio_pci_config_space_read();
        // 设置模拟位

        // 读原始bar的值，判断类型
        vfio_bars_prepare() {
            // for each bar
            vfio_bar_prepare() {
                // 读取bar原始值 
                vfio_pci_config_space_read();
                // 判断类型
                pci_bar = le32_to_cpu(pci_bar);
                bar->ioport = (pci_bar & PCI_BASE_ADDRESS_SPACE_IO);
                bar->mem64 = bar->ioport ? 0 : (pci_bar & PCI_BASE_ADDRESS_MEM_TYPE_64);
                bar->type = pci_bar & (bar->ioport ? ~PCI_BASE_ADDRESS_IO_MASK :
                                                    ~PCI_BASE_ADDRESS_MEM_MASK);
            }
        }

        // 注册bar 创建mr层次
        vfio_bars_register() {
            // for each bar
            vfio_bar_register() {
                // 创建io形bar
                bar->mr = g_new0();
                memory_region_init_io();
                
                if (bar->region.size) {
                    // 把region.mem 挂在 bar-mr下面
                    memory_region_add_subregion();

                    // 真正的mmp
                    vfio_region_mmap(&bar->region) {
                        map_base = mmap(); // 分配一个空间
                        // 获取region对应的fd
                        fd = vfio_device_get_region_fd();
                        // for each mem 逐段映射
                        region->mmaps[i].mmap = mmap();

                        // 用mmap创建出来的mmp 创建ram形mr
                        // kvm会为其建立ept页表，guset可以直接访问
                        memory_region_init_ram_device_ptr();
                    }
                }
                // 向pci核心注册这个bar
                pci_register_bar(pdev, nr, bar->type, bar->mr);

            }
        }
    }
}

```

```c
static void vfio_pci_realize(PCIDevice *pdev, Error **errp)
{
    VFIOPCIDevice *vdev = VFIO_PCI_DEVICE(pdev);
    VFIODevice *vbasedev = &vdev->vbasedev;

    // 1. 获取设备名称
    if (!vfio_device_get_name(vbasedev, errp)) {
        return;
    }

    // 2. 检查是否是 mdev 设备 
    // Mediated Device（ mediated device，中介设备）
    vbasedev->mdev = vfio_device_is_mdev(vbasedev);

    // 3. 核心 附加设备到 VFIO (打开设备文件，设置 group)
    if (!vfio_device_attach(name, vbasedev,
                            pci_device_iommu_address_space(pdev), errp)) {
        goto error;
    }

    // 4. 填充设备信息 (读取配置空间，BAR 信息)
    if (!vfio_pci_populate_device(vdev, errp)) {
        goto error;
    }

    // 5. 设置配置空间
    if (!vfio_pci_config_setup(vdev, errp)) {
        goto error;
    }

    // 6. 添加能力 (MSI, MSIX 等)
    if (!vfio_pci_add_capabilities(vdev, errp)) {
        goto out_unset_idev;
    }

    // 7. 设置中断
    if (!vfio_pci_interrupt_setup(vdev, errp)) {
        goto out_unset_idev;
    }
}
```

- 打开设备并设置 Group
```c
bool vfio_device_attach(char *name, VFIODevice *vbasedev,
                        AddressSpace *as, Error **errp)
{
    const char *iommu_type = vbasedev->iommufd ?
                            TYPE_VFIO_IOMMU_IOMMUFD :
                            TYPE_VFIO_IOMMU_LEGACY;

    return vfio_device_attach_by_iommu_type(iommu_type, name, vbasedev,
                                            as, errp); 
}

bool vfio_device_attach_by_iommu_type(const char *iommu_type, char *name,
                                    VFIODevice *vbasedev, AddressSpace *as,
                                    Error **errp)
{
    const VFIOIOMMUClass *ops =
        VFIO_IOMMU_CLASS(object_class_by_name(iommu_type));

    assert(ops);

    return ops->attach_device(name, vbasedev, as, errp);  // 根据iommu_type的区别调用不同的函数
    // 一个是 TYPE_VFIO_IOMMU_IOMMUFD TYPE_VFIO_IOMMU_LEGACY两个不同的架构
}
```

- Legacy路径

```c
static bool vfio_legacy_attach_device(const char *name, VFIODevice *vbasedev,
                                    AddressSpace *as, Error **errp)
{
    // 从 sysfs 读取 group id
    int groupid = vfio_device_get_groupid(vbasedev, errp); 

    // 核心 打开 group 并连接到 container 
    group = vfio_group_get(groupid, as, errp);

    // 从 group 中获取设备 fd
    if (!vfio_device_get(group, name, vbasedev, errp)) {
        goto group_put_exit;
    }

    // 创建 HostIOMMUDevice
    if (!vfio_device_hiod_create_and_realize(vbasedev,
                                            TYPE_HOST_IOMMU_DEVICE_LEGACY_VFIO,
                                            errp)) {
        goto device_put_exit;
    }
    return true;
}

// 打开group设备
static VFIOGroup *vfio_group_get(int groupid, AddressSpace *as, Error **errp)
{
    VFIOGroup *group;
    char path[32];
    struct vfio_group_status status = { .argsz = sizeof(status) };

    // 先查已有 group 链表
    QLIST_FOREACH(group, &vfio_group_list, next) {
        if (group->groupid == groupid) {
            if (VFIO_IOMMU(group->container)->space->as == as) {
                return group;
            } // ...
        }
    }

    group = g_malloc0(sizeof(*group));

    // 打开 /dev/vfio/<groupid>
    snprintf(path, sizeof(path), "/dev/vfio/%d", groupid);
    group->fd = cpr_open_fd(path, O_RDWR, ...);

    // 检查 group 是否 viable (所有设备都绑定了 vfio 驱动)
    if (ioctl(group->fd, VFIO_GROUP_GET_STATUS, &status)) {
        // ...
    }
    if (!(status.flags & VFIO_GROUP_FLAGS_VIABLE)) {
        error_setg(errp, "group %d is not viable", groupid);
        goto close_fd_exit;
    }

    group->groupid = groupid;

    // 核心 连接 container
    if (!vfio_container_connect(group, as, errp)) {
        goto close_fd_exit;
    }

    QLIST_INSERT_HEAD(&vfio_group_list, group, next);
    return group;
}


// 打开container并设置iommu
static bool vfio_container_connect(VFIOGroup *group, AddressSpace *as,
                                    Error **errp)
{
    VFIOLegacyContainer *container;
    int ret, fd = -1;
    VFIOAddressSpace *space;
    bool new_container = false;

    space = vfio_address_space_get(as);

    // --- 第一步：尝试加入已有 container ---
    QLIST_FOREACH(bcontainer, &space->containers, next) {
        container = VFIO_IOMMU_LEGACY(bcontainer);
        if (!ioctl(group->fd,
                    VFIO_GROUP_SET_CONTAINER, &container->fd)) {
            // 内核接受 -> 加入已有 container
            return vfio_container_group_add(container, group, errp);
        }
    }

    // --- 第二步：没有 container -> 创建新的 ---

    // 打开 /dev/vfio/vfio (container 设备)
    fd = qemu_open("/dev/vfio/vfio", O_RDWR, errp);

    // 检查 API 版本
    ret = ioctl(fd, VFIO_GET_API_VERSION);
    if (ret != VFIO_API_VERSION) { ... }

    // 核心 创建 container 对象 (内部调用 VFIO_SET_IOMMU)
    container = vfio_create_container(fd, group, errp);
    new_container = true;
    bcontainer = VFIO_IOMMU(container);

    // 获取 IOMMU 信息
    vioc = VFIO_IOMMU_GET_CLASS(bcontainer);
    if (!vioc->setup(bcontainer, errp)) {
        goto fail;
    }

    vfio_address_space_insert(space, bcontainer);

    // 将 group 加入 container
    if (!vfio_container_group_add(container, group, errp)) {
        goto fail;
    }

    // 注册 MemoryListener -> QEMU 可以跟踪 Guest 内存变化
    vfio_listener_register(bcontainer);

    bcontainer->initialized = true;
    return true;
}

static VFIOLegacyContainer *vfio_create_container(int fd, VFIOGroup *group,
                                            Error **errp)
{
    int iommu_type;
    const char *vioc_name;
    VFIOLegacyContainer *container;

    // 探测 IOMMU 类型 (TYPE1v2 -> TYPE1 -> SPAPR v2 -> SPAPR)
    iommu_type = vfio_get_iommu_type(fd, errp);

    // --- 两个关键的 ioctl ---
    if (!vfio_set_iommu(fd, group->fd, &iommu_type, errp)) {
        return NULL;
    }

    vioc_name = vfio_get_iommu_class_name(iommu_type);

    container = VFIO_IOMMU_LEGACY(object_new(vioc_name));
    container->fd = fd;
    container->iommu_type = iommu_type;
    return container;
}

static bool vfio_set_iommu(int container_fd, int group_fd,
                            int *iommu_type, Error **errp)
{
    // ioctl 1 将 group 绑定到 container
    if (ioctl(group_fd,
            VFIO_GROUP_SET_CONTAINER, &container_fd)) {
        error_setg_errno(errp, errno, "Failed to set group container");
        return false;
    }

    // ioctl 2  设置 IOMMU 类型
    while (ioctl(container_fd, VFIO_SET_IOMMU, *iommu_type)) {
        if (*iommu_type == VFIO_SPAPR_TCE_v2_IOMMU) {
            *iommu_type = VFIO_SPAPR_TCE_IOMMU;   // SPAPR v2 失败 → 回退到 v1
            continue;
        }
        error_setg_errno(errp, errno, "Failed to set iommu for container");
        return false;
    }

    return true;
} 

// 将group加入container
static bool vfio_container_group_add(VFIOLegacyContainer *container,
                                    VFIOGroup *group, Error **errp)
{
    if (!vfio_container_attach_discard_disable(container, group, errp)) {
        return false;
    }
    group->container = container;
    QLIST_INSERT_HEAD(&container->group_list,
                    group, container_next);
    vfio_group_add_kvm_device(group);
    return true;

}

static bool vfio_device_get(VFIOGroup *group, const char *name,
                            VFIODevice *vbasedev, Error **errp)
{
    int fd;

    // ioctl: 从 group 获取设备 fd
    fd = vfio_cpr_group_get_device_fd(group->fd, name);
    // fd = ioctl(group_fd, VFIO_GROUP_GET_DEVICE_FD, name);

    // 获取设备信息
    info = vfio_get_device_info(fd);

    // 填充 vbasedev
    vfio_device_prepare(vbasedev, VFIO_IOMMU(group->container), info);

    vbasedev->fd = fd;
    vbasedev->group = group;
    QLIST_INSERT_HEAD(&group->device_list, vbasedev, next);
    return true;
}
```
  
### 运行时调用栈

#### guest os访问configure space
```c
// QEMU
vfio_pci_read_config();

```