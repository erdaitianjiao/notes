## Uboot基础指令



- 帮助命令

  ~~~bash
  ?			# 打印帮助信息
  help		# 同上
  ? bootz
  help bootz
  ~~~

- 打印信息

  ~~~bash
  bdinfo		# 打印板载信息
  printenv	# 打印环境变量信息
  version		# uboot 的版本号
  ~~~

- 环境变量操作命令

  ~~~bash
  save env  	# 保存环境变量
  setenv [变量名] 新名称		  # 修改变量
  setenv [变量名]		   		# 删除变量
  setenv [新变量名] 新变量名称	    # 添加变量
  ~~~

- 内存操作

  ~~~bash
  md[.b, .w, .l] address [# of objects]
  # 命令中的[.b .w .l]对应 byte、word 和 long，也就是分别以 1 个字节、2 个字节、4 个字节来显示内存值。address 就是要查看的内存起始地址，[# of objects]表示要查看的数据长度，这个数据长度单位不是字节，而是跟你所选择的显示格式有关。比如你设置要查看的内存长度为20(十六进制为 0x14)，如果显示格式为.b 的话那就表示 20 个字节；如果显示格式为.w 的话就表示 20 个 word，也就是 20*2=40 个字节；如果显示格式为.l 的话就表示 20 个 long，也就是20*4=80 个字节。另外要注意 uboot 命令中的数字都是十六进制的！不是十进制的！
  
  nm [.b, .w, .l] address #nm 命令用于修改指定地址的内存值
  
  mm [.b, .w, .l] address #mm 命令也是修改指定地址内存值的使用 mm 修改内存值的时候地址会自增
  
  mw [.b, .w, .l] address value [count] # 命令 mw 用于使用一个指定的数据填充一段内存 address 表示要填充的内存起始地址，value 为要填充的数据，count 是填充的长度
  
  cp [.b, .w, .l] source target count # cp 是数据拷贝命令，用于将 DRAM 中的数据从一段内存拷贝到另一段内存中，或者把 Nor Flash 中的数据拷贝到 DRAM 中 cp 命令同样可以以.b、.w 和.l 来指定操作格式，source 为源地址，target 为目的地址，count为拷贝的长度
  
  cmp [.b, .w, .l] addr1 addr2 count # cmp 是比较命令，用于比较两段内存的数据是否相等 addr1 为第一段内存首地址，addr2 为第二段内存首地址，count 为要比较的长度
  ~~~

- 网络操作命令

| 环境变量  |                            描述                            |
| :-------: | :--------------------------------------------------------: |
|  ipaddr   | 开发板IP地址，可以不设置，使用dhcp命令来从路由器获取IP地址 |
|  ethaddr  |                开发板的MAC地址，一定要设置                 |
| gatewayip |                          网关地址                          |
|  netmask  |                          子网掩码                          |
| serverip  |     服务器IP地址，也就是Ubuntu主机IP地址，用于调试代码     |

~~~bash
ping 192.168.1.253 # 开发板的网络能否使用，是否可以和服务器(Ubuntu 主机)进行通信，通过 ping 命令就可以验证，直接 ping 服务器的 IP 地址即可

dhcp # 用于从路由器获取 IP 地址

nfs [loadAddress] [[hostIPaddr:]bootfilename] # 可以使用 nfs 命令来将 zImage 下载到开发板 DRAM 的 0X80800000 地址处

tftp 80800000 zImage # tftp 命令的作用和 nfs 命令一样，都是用于通过网络下载东西到 DRAM 中
~~~

- EMMC 和 SD 卡操作命令

  ~~~bash
  命令	描述
  mmc info	# 输出 MMC 设备信息
  mmc read addr blk cnt	# 读取 MMC 中的数据
  mmc write addr blk cnt	# 向 MMC 设备写入数据
  mmc rescan	# 扫描 MMC 设备
  mmc part	# 列出 MMC 设备的分区
  mmc dev [dev] [part] # 切换 MMC 设备
  mmc list	# 列出当前有效的所有 MMC 设备
  mmc hwpartition	# 设置 MMC 设备的分区
  mmc bootbus	# 设置指定 MMC 设备的 BOOT_BUS_WIDTH 域的值
  mmc bootpart	# 设置指定 MMC 设备的 boot 和 RPMB 分区的大小
  mmc partconf	# 设置指定 MMC 设备的 PARTITION_CONFIG 域的值
  mmc rst	# 复位 MMC 设备
  mmc setdsr	# 设置 DSR 寄存器的值
  ~~~

- FAT 格式文件系统操作命令

  ~~~bash
  fatinfo <interface> [<dev[:part]>] # 显示指定分区的文件系统信息
  
  fatls <interface> [<dev[:part]>] [directory] # 列出 FAT 分区中的文件和目录。
  
  fstype <interface> <dev>:<part> # 查看分区的文件系统格式（FAT/ext4/未知
  
  fatload <interface> [<dev[:part]>] <addr> <filename> [bytes [pos]] # 将文件从存储设备加载到 DRAM
  fatload mmc 1:1 0x80800000 zImage  # 将 zImage 读取到 DRAM 的 0x80800000 地址
  
  fatwrite <interface> <dev[:part]> <addr> <filename> <bytes> # 将 DRAM 中的数据写入 FAT 分区（需配置 CONFIG_FAT_WRITE）
  fatwrite mmc 1:1 0x80800000 zImage 0x6788f8  # 将 DRAM 中的 zImage 写入 eMMC
  
  fatload mmc 0:1 0x80800000 zImage
  fatload mmc 0:1 0x83000000 imx6ull-14x14-evk.dtb
  ~~~

- ddadEXT 格式文件系统命令

  ~~~bash
  ext4ls：列出 ext4 分区文件（如 ext4ls mmc 1:2）。
  ext4load：从 ext4 分区加载文件到内存。
  ext4write：写入数据到 ext4 分区。
  
  ~~~

- aaaaa



