# qemu vfio 实验说明

## 实验设计
本实验采用嵌套虚拟化的方式通过vfio共享显卡给虚拟机
- 本实验的三层虚拟机
1. l0 一个linux物理机
2. l1 使用qemu启动的一个虚拟机
3. l2 在l1里面使用qemu启动的虚拟机
   
- 目标
1. 在l1里面使用vfio接管显卡驱动
2. 在l2里面能接收并且正常使用显卡


## 具体实现

l1 启动命令
在l1里面给一个虚拟iommu

```bash
qemu-system-x86_64 \
    -enable-kvm \
    -m 4G \
    -smp 2 \
    -kernel bzImage \
    -drive if=virtio,file=debian.img,format=raw \
    -append "nokaslr root=/dev/vda console=ttyS0 earlyprintk=serial,ttyS0 debug ignore_loglevel intel_iommu=on iommu=pt" \
    -netdev user,id=net0,hostfwd=tcp::2222-:22 \
    -device virtio-net-pci,netdev=net0,disable-legacy=on,disable-modern=off \
        -cpu host \
        -enable-kvm \
        -machine q35,accel=kvm,kernel-irqchip=split \
    -device intel-iommu,intremap=on,caching-mode=on \
    -device virtio-gpu-pci \
    -display none \
    -nographic \
    -s #-S
```

l2 启动参数
```bash
qemu-system-x86_64 \
    -enable-kvm \
    -m 512M \
    -smp 2 \
    -kernel bzImage \
    -drive if=virtio,file=debian-rootfs.img,format=raw \
    -device vfio-pci,host=00:03.0 \
    -append "root=/dev/vda rw console=ttyS0 init=/etc/init" \
    -nographic \
    -netdev user,id=net0,hostfwd=tcp::2223-:22 \
    -device virtio-net-pci,netdev=net0
```

## 

## 实验结果

## 注意事项
1. 编译内核一定要关闭随机化，否则 GDB 的断点会失效
CONFIG_RANDOMIZE_BASE = n

