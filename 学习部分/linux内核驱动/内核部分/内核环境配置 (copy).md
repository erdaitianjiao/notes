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
mkdir fs       					# 用于挂载镜像
sudo mount -t ext4 -o loop rootfs.img ./fs

cd busybox-1.35.0 
sudo make install -j $(nproc) CONFIG_PREDIX = ../fs
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
gdb vmlinux老师

# 在gdb中链接调试端口
(gdb) target remote:1234
(gdb) c
~~~



#### 二、Linux2.6系列内核的调试环境配置



##### 创建docker环境

~~~bash
sudo docker run -it --name ubuntu-kernel.2.6.34 -v /home/tianjiao/workspace/linux/virtualmachine/kernel-linux2.6.34-machine:/mnt <your-image> /bin/bash

# docker 指令
sudo docker ps -a
sudo docker start <id>
sudo docker exrc -it <id> <sh> 
~~~

下载软件包

~~~bash
#更新
apt update
#安装安装包
apt install gcc gdb make qemu qemu-system-x86 libncurses5-dev libncurses5-dev build-essential -y
~~~



##### 1. 安装gcc-4.4

因为ubuntu22.04太新了 没有gcc4.4的软件源 所以需要添加新的源

~~~bash
sudo vim /etc/apt/sources.list

# 在最后添加
deb http://archive.ubuntu.com/ubuntu/ trusty main universe
deb http://archive.ubuntu.com/ubuntu/ trusty-updates main universe
~~~

添加公钥

~~~bash
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 40976EAF437D05B5
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 3B4FE6ACC0B21F32
~~~

安装gcc-4.4

~~~bash
sudo apt update
sudo apt install gcc-4.4 g++-4.4
~~~

##### 2. 编译linux内核

~~~bash
wget https://www.kernel.org/pub/linux/kernel/v2.6/linux-2.6.24.tar.xz
tar -xvf 
~~~

如果报错

~~~bash
vim arch/x86/vdso/Makefile
# 修改 -m elf_x86_64 为 -m32
# 修改 -m elf_i386 为 -m32
~~~



##### 3. 编译busybox

由于版本问题真的很麻烦 这里选择与高版本不同的内核版本busybox1.28.4

~~~bash
wget https://busybox.net/downloads/busybox-1.28.4.tar.bz2
wget https://busybox.net/downloads/busybox-1.10.4.tar.bz2
tar xjf busybox-1.28.4.tar.bz2

# 先修改
# 修改busybox 下include/libbb.h 添加 #include "sys/resource.h"
~~~

然后进入编译配置

~~~bash
# 得关闭旧版配置 使用图形化界面配置

make menuconfig CC=gcc-4.4
# 在图形化里面 勾选
# Settings->Build static binary
# 然后保存
~~~

使用参数编译

~~~bash
CC=gcc-4.4

vim arch/x86/vdso/Makefile
~~~



如果遇到`stime`的报错 是`stime`因为是一个过时的系统调用，在现代 glibc 中已被移除。BusyBox 需要适配新方法

~~~bash
make menuconfig
# 进入 "Coreutils" -> 取消选择 "date"
# 进入 "Linux-System-Utilities" -> 取消选择 "rdate
~~~



#### Linux0.11 版本的调试

