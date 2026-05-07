### Linux内核环境配置

#### 一、高版本内核的编译运行

##### 1. 编译linux内核

- 下载解压在官网下载的内核(这里以5.10.240为例子)

~~~bash
tar -xvf linux-5.10.240.tar.xz
cd linux-5.10.240

make deconfig
make -j $(nproc) bzImage
~~~



##### 2.配置busybox

- 使用wget下载busybox进行文件系统的编译

~~~bash
wget https://busybox.net/downloads/busybox-1.35.0.tar.bz2
tar xvf busybox-1.35.0.tar.bz2
~~~

- 编译的时候需要开启静态链接库

~~~bash
make menuconfig

# 在图形化里面 勾选
# Settings->Build static binary
# 然后保存
~~~



##### 3.创建内核使用的磁盘镜像，并对其进行格式化

~~~bash
dd if=/dev/zero of=root.img bs=1M count=20			# 创建磁盘镜像
mkfs.ext4 root.img									# 格式化成ext4格式
~~~



##### 4.将busybox编译进磁盘镜像

~~~ bash
mkdir fs       		# 用于挂载镜像
sudo mount -t ext4 -o loop rootfs.img ./fs

cd busybox-1.35.0
sudo make install -j $(nproc) CONFIG_PREFIX = ../fs
sudo mkdir proc dev etc home mnt
sudo cp -r ../busybox-1.35.0/examples/bootfloppy/etc/* etc/

cd ..
sudo chmod -R 777 fs/ 			# 更改权限，以免无法运行
sudo umount fs
~~~



##### 5.启动linux内核

~~~bash
# 建议将bzImage和rootfs.img放在同一文件夹，方便操作，然后编辑一个sh文件，用sh文件一键启动 
# 如果要进行调试的话 需要将linux源码编译出来的vmlinux一并复制到目录下
qemu-system-x86_64 -kernel bzImage  -hda rootfs.img  -append "root=/dev/sda"
~~~

~~~
./configure --prefix=/home/tianjiao/workspace/linux/virtualmachine/bochs --enable-debugger --enable-disasm --enable-iodebug --enable-x86-debugger --with-x --with-x11

~~~

##### 6. 使用gdb调试linux内核

~~~bash
# 调试命令 -s -S
qemu-system-x86_64 -kernel bzImage -drive file=rootfs.img,format=raw -append "root=/dev/sda" -s -S

# 在新的窗口打开
gdb vmlinux

# 在gdb中链接调试端口
(gdb) target remote:1234
(gdb) c
~~~



#### 二、Linux2.6系列内核的调试环境配置

##### 1. 在ubuntu22.04中安装docker qume gdb等

~~~bash
sudo apt install docker-ce docker-ce-cli containerd.io
sudo apt install qemu-kvm qemu-system-x86
sudo apt install gdb
~~~

##### 2. 创建docker环境

~~~bash
# 拉取ubuntu14.04环境
sudo docker pull ubuntu:14.04

# 选择一个目录 创建一个放虚拟机所有文件的文件夹
mkdir linux2.6.34-virtual-machine
wget https://www.kernel.org/pub/linux/kernel/v2.6/linux-2.6.34.1.tar.xz
wget https://busybox.net/downloads/busybox-1.20.1.tar.bz2

# 开一个docker环境 创建一个环境名字叫ubuntu-kernel2.6.34的环境 然后挂载我当前的linux目录到 ubuntu容器的/mnt目录下

# 不能直接复制这个命令
sudo docker run -it --name ubuntu-kernel.2.6.34 -v (当前文件位置)kernel-linux2.6.34-machine:/mnt ubuntu:14.04 /bin/bash

# docker 指令
sudo docker ps -a					# 列出当前存在的容器
sudo docker start <id>				# 开始一个容器
sudo docker exec -it <id> <sh> 		# 执行一个容器 并启动命令行编译
~~~

在ubuntu14.01下载软件包

~~~bash
#更新
apt update
#安装安装包
apt install gcc gdb make qemu qemu-system-x86 libncurses5-dev libncurses5-dev build-essential -y
~~~

##### 3. 编译linux内核

~~~bash
# 下过了就不用下了
wget https://www.kernel.org/pub/linux/kernel/v2.6/linux-2.6.34.1.tar.xz
tar -xvf linux-2.6.34.1
cd linux-2.6.34.1
make defconfig
make menuconfig
# 进入 Kernel_hacking 选择 Compile_the_kernel_with_dubug_info [*]
# 保存退出
make
~~~

如果报错

~~~bash
vim arch/x86/vdso/Makefile
# 修改 -m elf_x86_64 为 -m64  	中间没有空格
# 修改 -m elf_i386 为 -m32		中间没有空格
~~~

将镜像目录复制到我们的虚拟机目录下 方便启动用

~~~bash
cp arch/x86_64/boot/bzImage ../
~~~



##### 4. 编译busybox

由于版本问题真的很麻烦 这里选择与高版本不同的内核版本busybox1.20.1

~~~bash
# 下过了就不用下了
wget https://busybox.net/downloads/busybox-1.20.1.tar.bz2
tar xjf busybox-1.28.4.tar.bz2l

# 先修改
# 修改busybox 下include/libbb.h 添加 #include "sys/resource.h"
~~~

然后进入编译配置

~~~bash
make defconfig
make menuconfig
# 得关闭旧版配置 使用图形化界面配置
# 在图形化里面 勾选
# Busy_Settings->Build_Options->Build_as_static_binary [*]
# 取消选择下面的选项
# Shell->Job_Control
# 然后保存

# 开始编译
make
~~~

如果有报错则

~~~bash
# 修改busybox 下include/libbb.h 
# 在开头添加 #include "sys/resource.h"
vim include/libbb.h
 
# 重新make
make
~~~

配置文件系统其他文件

~~~bash
# 制作 系统镜像盘
cd _install
mkdir etc proc sys mnt dev tmp
mkdir -p etc/init.d

cat >> etc/fstab<<EOFcd .
proc    /proc   proc    defaults        0       0
tmpfs   /tmp    tmpfs   defaults        0       0
sysfs   /sys    sysfs   defaults        0       0
EOF

cat>>etc/init.d/rcS<<EOF
echo "Welcome to linux..."
mount -o remount rw /
EOF

chmod 755 etc/init.d/rcS 
cat>>etc/inittab<<EOF
::sysinit:/etc/init.d/rcS
::respawn:-/bin/sh
::askfirst:-/bin/sh
::ctrlaltdel:/bin/umount -a -r
EOF

chmod 755 etc/inittab
cd dev
sudo mknod console c 5 1
sudo mknod null c 1 3
sudo mknod tty1 c 4 1

cd ../..

~~~

在ubuntu14.04里面创建镜像

~~~bash
# 这个在ubuntu14.04中执行
sudo dd if=/dev/zero of=initrd.img bs=4096 count=1024
sudo mkfs.ext3 initrd.img
mkdir rootfs
~~~

将文件目录复制到initrd.img镜像中

~~~bash
# 挂载的时候在ubuntu22.04中执行 目录是在busybox目录下
sudo mount -o loop initrd.img rootfs
sudo cp -rf ./_install/* ./rootfs
sudo umount ./rootfs  
~~~

做完检查一下`initrd.img`的文件格式

~~~bash
file initrd.img 	# 如果是 ext3 则正常
cp initrd.img ../
~~~

到这里 内核镜像和文件系统都在/mnt目录下了 在宿主机的linux2.6.34-virtual-machine文件夹里面

使用QEMU启动内核 在容器里和外面运行都行

~~~
qemu-system-x86_64 \
-nographic \
-kernel ./bzImage \
-initrd ./initrd.img \
-append "root=/dev/ram init=/bin/bash console=ttyS0"
~~~



