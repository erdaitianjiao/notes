### 正点原子Imx6ull sd卡启动 uboot Linux busybox文件系统流程

#### 简单说明

- 由于原教程没有详细给出怎么从sd卡启动busybox，所以本文章在这里写一下流程，思路是将sd卡分成三部分，一部分用于存uboot，一部分用于存zImage和dtb文件，最后一部分用于存busybox文件系统

#### 一、 格式化并对sd卡进行分区

1. **对sd卡进行分区(使用ubuntu)**

~~~bash
lsblk  					#查看sd卡设备名称 和挂载情况

# 这里以sdb为例
# 如果有挂载先取消挂载
sudo umount /dev/sdb1

sudo fdisk /dev/sdb
# 随后先输入d 删除原有分区结构
# 输入n 创建新的分区
# 选择1 创建sdb1
# 选择起始大小 默认就好 2048
# 选择终止大小 我在这里选择sd卡大小的一半

# 再次输入n 创建第二个分区
# 直接默认就好 因只需要两个分区
# 最后输入w 写入分区
~~~

2. **对sd卡进行文件格式化**

第一个分区格式化成 fat32 用于存放 zImage 镜像和 dtb 文件, 第二个分区格式化成 ext4 格式，用于存放busybox跟文件系统

~~~bash
sudo mkfs.vfat -F 32 /dev/sdb1		# 第一个分区

sudo mkfs.ext4 /dev/sdb2  			# 第二个分区
~~~

#### 二、将uboot zimage busybox 等文件放进sd卡里面

1. **烧录uboot**

~~~bash
# 进入移植好的uboot里面
./imxdownload u-boot.bin /dev/sdb
~~~

2. **复制移植好的 zImage 和 dtb 文件进 /dev/sdb1 里面**

~~~bash
sudo mkdir /mnt/boot /mnt/rootfs		# 创建临时目录先挂载
sudo mount /dev/sdb1 /mnt/boot
sudo mount /dev/sdb2 /mnt/rootf

# 进去zImage和dtb目录
cp zImage imx6ull-alientek-emmc.dtb /mnt/boot/
~~~

3. **将Busybox文件系统编译进sdb2分区**

对ext4文件系统的一切操作都需要 sudo

~~~bash
make 
sudo make install CONFIG_PREFIX=/mnt/rootfs/
# 然后根据文档向ext4文件里面拷贝东西，记得一定要用sudo
~~~

#### 三、启动uboot进行操作

~~~bash
# 先检查文件格式是否正确
fstype mmc 0:1 	# 输出fat
fstype mmc 0:2	# 输出ext4

# 检查文件是否写入 
ls mmc 0:1 		# 输出zImage 和 dtb 文件
ls mmc 0:2		# 输出根文件系统 lib usr 等

# 加载zImage 和 dtb文件
load mmc 0:1 80800000 zImage
load mmc 0:1 83000000 imx6ull-alientek-emmc.dtb

# 修改uboot启动变量
# 设置 bootargs 从 eMMC 分区 2 启动
setenv bootargs 'console=ttymxc0,115200 root=/dev/mmcblk0p2 rootwait rw'
saveenv

# 启动
bootz 80800000 - 83000000
~~~

