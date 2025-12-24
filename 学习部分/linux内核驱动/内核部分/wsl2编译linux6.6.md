### linux6.6搭建调试环境

#### 一、下载linux内核

```bash
git clone --branch v6.6 https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git linux-6.6
cd linux-6.6
```

#### 二、使用debian作为文件系统

~~~bash
sudo apt install debootstrap
mkdir ~/debian-rootfs
sudo debootstrap --arch=amd64 bullseye ./debian-rootfs http://deb.debian.org/debian/
~~~

#### 三、创建文件镜像

```bash
dd if=/dev/zero of=debian.img bs=1M count=4096
mkfs.ext4 debian.img


mkdir fs
sudo mount debian.img fs
sudo cp -a debian-rootfs/* fs/
sudo umount fs

```

#### 四、启动

```bash
qemu-system-x86_64 \
        -kernel bzImage  \
        -hda debian.img  \
        -append "root=/dev/sda init=/bin/bash console=ttyS0 rw" \
        -nographic
        
# 先进root改密码 然后再执行
exec /sbin/init     

qemu-system-x86_64 \
        -kernel bzImage  \
        -hda debian.img  \
        -append "root=/dev/sda console=ttyS0 rw" \
        -nographic
```

