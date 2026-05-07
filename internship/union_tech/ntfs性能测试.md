### ntfsplus ntfs3 ntfs3g 性能对比

#### 一、要求

测试一下ntfsplus

内核代码可以用gitlab仓库的ntfs-next分支：
https://gitlabwh.uniontech.com/ut004151/uosste-kernel/-/tree/ntfs-next

ntfsplus-progs用户态工具：
https://github.com/namjaejeon/ntfsprogs-plus

要求：
\- 编译内核（可以用debian配置或者uos v25配置）
\- 测试ntfsplus性能：类似于现在的落盘测试，只是分区用ntfs而不是ext4，暂时不用测试加密，就nor部分就行
\- 测试fsck.ntfs：格式化u盘ntfs，写入过程里拔盘（写一个大文件 AND/OR 写一堆小文件），看是不是会fs损坏，看是否可以用fsck.ntfs修复。

对比多个ntfsplus驱动：ntfs-3g, ntfs3, ntfsplus
\- 性能对比：关注带宽和cpu开销
\- 稳定性对比：两种情况拔盘时是否会导致文件系统损坏？是否可以用fsck.ntfs修复？

 测试环境：
\- 在i3上测试吧，系统用uos-v25或者debian13。

可以考虑去库房借一个i3测试机长期用，分硬盘时提前考虑会装多个系统，具体可以请教楚梦凡，有什么疑问可以直接在群里问

#### 二、环境搭建

##### 编译内核

```bash
开启ntfsplus [M]
make -j12 deb-pkg
```

问题

1. 在demsg里面发现 

```c
root@uos:~# dmesg | grep -i ntfs
[    0.304397] ntfs3: Enabled Linux POSIX ACLs support
[    0.304399] ntfs3: Warning: Activated 64 bits per cluster. Windows does not support this
[    0.304399] ntfs3: Read-only LZX/Xpress compression included
[    0.304417] ntfsplus: Failed to register NTFS sysctls!
```

在代码中搜索查看到

fs/ntfspuls/misc.c

```c
static const struct ctl_table ntfs_sysctls[] = {
	{
		.procname	= "ntfs-debug",
		.data		= &debug_msgs,		/* Data pointer and size. */
		.maxlen		= sizeof(debug_msgs),
		.mode		= 0644,			/* Mode, proc handler. */
		.proc_handler	= proc_dointvec
	},
	{} // < === here
};

/* Storage for the sysctls header. */
static struct ctl_table_header *sysctls_root_table;

/**
 * ntfs_sysctl - add or remove the debug sysctl
 * @add:	add (1) or remove (0) the sysctl
 *
 * Add or remove the debug sysctl. Return 0 on success or -errno on error.
 */
int ntfs_sysctl(int add)
{
	if (add) {
		sysctls_root_table = register_sysctl("fs", ntfs_sysctls);
		if (!sysctls_root_table)
			return -ENOMEM;
	} else {
		unregister_sysctl_table(sysctls_root_table);
		sysctls_root_table = NULL;
	}
	return 0;
}
```

有一个空表项 删除以后重新编译即可

#### 三、测试

##### 1. 三种ntfs驱动的性能测试

###### 1) 格式化以及挂载

```bash
# 加载ntfsplus
modprobe ntfsplus

# 格式化磁盘
mkntfs /dev/nvme0n1p5
# 检查磁盘格式
mount -t ntfsplus /dev/nvme0n1p5 /mnt
df -rT

# 测试ntfs3
rmmode ntfsplus
modporbe ntfs3
# 格式化磁盘
mkfs.ntfs /dev/nvme0n1p5
# 检查磁盘格式
mount -t ntfs3 /dev/nvme0n1p5 /mnt
df -rT

# 测试ntfs-3g
rmmod ntfs3
mkfs.ntfs /dev/nvme0n1p5
ntfs-3g /dev/nvme0n1p5
```

###### 2) 测试命令

```bash
fio iotest.jon --section=ntfsplus-fs-sync-r

# iotest.job 文件内容
[global]
size=4096m
bs=1m
numjobs=1
group_reporting=1

[ntfsplus-fs-sync-r]
filename=/mnt/testfile
ioengine=psync
iodepth=32
rw=read

[ntfsplus-fs-sync-w]
filename=/mnt/testfile
ioengine=psync
iodepth=32
rw=write

[ntfsplus-fs-sync-rand-r]
filename=/mnt/testfile
ioengine=psync
iodepth=32
bs=4k
rw=randread

[ntfsplus-fs-sync-rand-w]
filename=/mnt/testfile
ioengine=psync
iodepth=32
bs=4k
rw=randwrite
```

###### 3) 检查文件系统格式

```bash
root@uos:~/fio_test# df -hT | grep nvme0n1p5
/dev/nvme0n1p5 ntfsplus   20G   66M   20G    1% /mnt
```

##### 2. 测试文件系统稳定性 以及修复工具  ntfsck

###### 1) ntfsplus测试

大文件测试

```bash
# 生成4g的测试文件 用于mv
dd if=/dev/zero of=./4GB_file.bin bs=1M count=4096
dd if=/dev/urandom of=./4GB_random.bin bs=1M count=4096

# 格式化文件系统
mkntfs /dev/sdh1

# 检查挂载用的驱动
root@uos:~/tmp# mount -t ntfsplus /dev/sdh1 /mnt
root@uos:~/tmp# df -hT | grep sdh
/dev/sdh1      ntfsplus   10G   52M   10G    1% /mnt

root@uos:~/tmp# mv 4GB_random.bin /mnt
# ============== 这个时候拔出U 盘 ==================
mv: 写入 '/mnt/4GB_random.bin' 时出错: 输入/输出错误

# 重新插入u盘
# 进行ntfsck检查 
root@uos:~/tmp# /root/ntfsprogs-plus/src/ntfsck /dev/sdk1
Parse #1: Check system files...
Parse #2: Replay logfile...
Parse #3: Check index entries in volume...
lost+found(65) created
Parse #4: Check mft entries in volume...
Parse #5: Check cluster bitmap...
Clean, No errors found or left (errors:0, fixed:0)

# 检查文件状态
# 能保留已经传输
root@uos:/# ls mnt -alh
总计 876M
drwxrwxrwx  1 root root    56 12月 5日 14:22 .
drwxr-xr-x 21 root root  4.0K 12月 4日 14:21 ..
-rw-r--r--  1 root root 1007M 12月 5日 14:22 4GB_random.bin
-rw-r--r--  1 root root     6 12月 5日 14:22 hello.txt
```

###### 2) ntfs3 测试

```bash
root@uos:~/tmp# mount -t ntfs3 /dev/sdq1 /mnt
root@uos:~/tmp# df -hT | grep sds
/dev/sdq1      ntfs3      10G   52M   10G    1% /mnt

root@uos:~/tmp# cp 4GB_random.bin /mnt
# ============== 这个时候拔出U 盘 ==================
cp: 写入 '/mnt/4GB_random.bin' 时出错: 设备上没有空间

# 重新插入 尝试挂载
# 挂载不上
root@uos:~/tmp# mount -t ntfs3 /dev/sds1 /mnt
mount: /mnt: wrong fs type, bad option, bad superblock on /dev/sds1, missing codepage or helper program, or other error.
       dmesg(1) may have more information after failed mount system call.

# 使用ntfsck检查 并修复
root@uos:~/tmp# /root/ntfsprogs-plus/src/ntfsck -n /dev/sds1
$MFTMirr does not match $MFT (record 0). Correcting differences in $MFT record (y/N)? N
ntfsck mount failed, errno : 5
2 errors left (errors:2, fixed:0)

root@uos:~/tmp# /root/ntfsprogs-plus/src/ntfsck -a /dev/sds1
$MFTMirr does not match $MFT (record 0). Correcting differences in $MFT record (y/N)? y
$MFTMirr does not match $MFT (record 3). Correcting differences in $MFT record (y/N)? y
Parse #1: Check system files...
Parse #2: Replay logfile...
Parse #3: Check index entries in volume...
lost+found(64) created
Parse #4: Check mft entries in volume...
Parse #5: Check cluster bitmap...
Clean, No errors found or left (errors:2, fixed:2)

root@uos:~/tmp# /root/ntfsprogs-plus/src/ntfsck -a /dev/sds1       
$MFTMirr does not match $MFT (record 0). Correcting differences in $MFT record (y/N)? y
$MFTMirr does not match $MFT (record 3). Correcting differences in $MFT record (y/N)? y
Parse #1: Check system files...
Parse #2: Replay logfile...
Parse #3: Check index entries in volume...
lost+found(64) created
Parse #4: Check mft entries in volume...
Parse #5: Check cluster bitmap...
Clean, No errors found or left (errors:2, fixed:2)
root@uos:~/tmp# /root/ntfsprogs-plus/src/ntfsck -a /dev/sds1
Parse #1: Check system files...
Parse #2: Replay logfile...
Parse #3: Check index entries in volume...
Trying to read non-allocated mft records (65 > 27): Illegal seek
Failed to open/check 'lost+found'
Trying to read non-allocated mft records (28 > 27): Illegal seek
Failed to open inode(27)
Index entry(27:hello.txt) is corrupted, Removing index entry from parent(5) (y/N)? y
Trying to read non-allocated mft records (65 > 27): Illegal seek
Failed to open inode(64)
Index entry(64:lost+found) is corrupted, Removing index entry from parent(5) (y/N)? y
Parse #4: Check mft entries in volume...
Parse #5: Check cluster bitmap...
Total lcn bitmap clear:281, Total missing lcn bitmap:0
Clean, No errors found or left (errors:283, fixed:283)

root@uos:~/tmp# /root/ntfsprogs-plus/src/ntfsck -a /dev/sds1
$MFTMirr does not match $MFT (record 0). Correcting differences in $MFT record (y/N)? y
$MFTMirr does not match $MFT (record 3). Correcting differences in $MFT record (y/N)? y
Parse #1: Check system files...
Parse #2: Replay logfile...
Parse #3: Check index entries in volume...
lost+found(64) created
Parse #4: Check mft entries in volume...
Parse #5: Check cluster bitmap...
Clean, No errors found or left (errors:2, fixed:2)

root@uos:~/tmp# /root/ntfsprogs-plus/src/ntfsck -a /dev/sds1
Parse #1: Check system files...
Parse #2: Replay logfile...
Parse #3: Check index entries in volume...
Trying to read non-allocated mft records (65 > 27): Illegal seek
Failed to open/check 'lost+found'
Trying to read non-allocated mft records (28 > 27): Illegal seek
Failed to open inode(27)
Index entry(27:hello.txt) is corrupted, Removing index entry from parent(5) (y/N)? y
Trying to read non-allocated mft records (65 > 27): Illegal seek
Failed to open inode(64)
Index entry(64:lost+found) is corrupted, Removing index entry from parent(5) (y/N)? y
Parse #4: Check mft entries in volume...
Parse #5: Check cluster bitmap...
Total lcn bitmap clear:281, Total missing lcn bitmap:0
Clean, No errors found or left (errors:283, fixed:283)

root@uos:~/tmp# /root/ntfsprogs-plus/src/ntfsck -a /dev/sds1
Parse #1: Check system files...
Parse #2: Replay logfile...
Parse #3: Check index entries in volume...
lost+found(64) created
Parse #4: Check mft entries in volume...
Clear MFT bitmap count:1
Parse #5: Check cluster bitmap...
Clean, No errors found or left (errors:1, fixed:1)

root@uos:~/tmp# /root/ntfsprogs-plus/src/ntfsck /dev/sde1
$MFTMirr does not match $MFT (record 3). Correcting differences in $MFT record (y/N)? y
Parse #1: Check system files...
Parse #2: Replay logfile...
Parse #3: Check index entries in volume...
Parse #4: Check mft entries in volume...
Parse #5: Check cluster bitmap...
Clean, No errors found or left (errors:1, fixed:1)

root@uos:~/tmp# /root/ntfsprogs-plus/src/ntfsck -a /dev/sdf1
$MFTMirr does not match $MFT (record 3). Correcting differences in $MFT record (y/N)? y
Parse #1: Check system files...
Parse #2: Replay logfile...
Parse #3: Check index entries in volume...
Parse #4: Check mft entries in volume...
Parse #5: Check cluster bitmap...
Total lcn bitmap clear:2048, Total missing lcn bitmap:0
Clean, No errors found or left (errors:2049, fixed:2049)

root@uos:~/tmp# /root/ntfsprogs-plus/src/ntfsck -a /dev/sdi1
Parse #1: Check system files...
Parse #2: Replay logfile...
Parse #3: Check index entries in volume...
lost+found(64) created
Parse #4: Check mft entries in volume...
Clear MFT bitmap count:1
Parse #5: Check cluster bitmap...
Total lcn bitmap clear:645081, Total missing lcn bitmap:0
Clean, No errors found or left (errors:645082, fixed:645082)

# 检查文件是否还存在
root@uos:/# ls /mnt -alh
总计 2.5G
drwxrwxrwx  1 root root 4.0K 12月 8日 09:30 .
drwxr-xr-x 21 root root 4.0K 12月 5日 15:06 ..
-rw-r--r--  1 root root 2.5G 12月 8日 09:31 4GB_random.bin
drwxr-xr-x  1 root root    0 12月 8日 09:24 lost+found

# ntfsck会创建lost+found文件夹 有时会这样也很少见 1/5
root@uos:/# ls /mnt -alh
ls: 无法访问 '/mnt/lost+found': 无效的参数
总计 2.5G
drwxrwxrwx  1 root root 4.0K 12月 8日 09:46 .
drwxr-xr-x 21 root root 4.0K 12月 5日 15:06 ..
-rw-r--r--  1 root root 2.5G 12月 8日 09:46 4GB_random.bin
d?????????  ? ?    ?       ?              ? lost+found

# 有概率发生复制的文件丢失 5次出现1次丢失
root@uos:/# ls /mnt -alh
总计 8.0K
drwxrwxrwx  1 root root 4.0K 12月 8日 09:50 .
drwxr-xr-x 21 root root 4.0K 12月 5日 15:06 ..
drwxr-xr-x  1 root root    0 12月 8日 09:52 lost+found
```



###### 3) ntfs-3g测试

```bash
root@uos:~/tmp# mount /dev/sdm1 /mnt
root@uos:~/tmp# df -hT | grep sdm
/dev/sdm1      fuseblk    10G   52M   10G    1% /mnt

root@uos:~/tmp# cp 4GB_file.bin /mnt
cp: 写入 '/mnt/4GB_file.bin' 时出错: 输入/输出错误

# 再次插入u盘 

# 因为uos自动把我的磁盘用ntfs3挂载了 我就看了一下
root@uos:~/tmp# df -hT | grep sdn
/dev/sdn1      ntfs3      10G   57M   10G    1% /media/uos/30C432E94525D754

root@uos:/media/uos/30C432E94525D754# ls -alh
ls: 无法访问 '4GB_random.bin': 无效的参数
总计 8.0K
drwxrwxrwx  1 uos  uos  4.0K 12月 5日 14:39 .
drwxr-x---+ 5 root root 4.0K 12月 5日 14:39 ..
-?????????? ? ?    ?       ?              ? 4GB_random.bin
-rw-r--r--  1 root root    6 12月 5日 14:22 hello.txt
drwxr-xr-x  1 root root    0 12月 5日 14:24 lost+found
root@uos:/media/uos/30C432E94525D754# 


# 挂载成ntfs-3g检查文件情况
root@uos:/mnt# ls -alhg
总计 4.1M
drwxrwxrwx  1 root 4.0K 12月 5日 14:39 .
drwxr-xr-x 21 root 4.0K 12月 4日 14:21 ..
-rwxrwxrwx  1 root 4.1M 12月 5日 14:39 4GB_random.bin
-rwxrwxrwx  1 root    6 12月 5日 14:22 hello.txt
drwxrwxrwx  1 root    0 12月 5日 14:24 lost+found

# 再挂载成为ntfsplus再检查一下
root@uos:/mnt# ls -lah
总计 4.1M
drwxrwxrwx  1 uos  uos    56 12月 5日 14:39 .
drwxr-xr-x 21 root root 4.0K 12月 4日 14:21 ..
-rwxrwxrwx  1 root root 4.1M 12月 5日 14:39 4GB_random.bin
-rw-r--r--  1 root root    6 12月 5日 14:22 hello.txt
drwxrwxrwx  1 root root   48 12月 5日 14:24 lost+found

# 如果传输时间比较长载拔出则挂不上磁盘
root@uos:~/tmp# mount /dev/sdj1 /mnt
$MFTMirr does not match $MFT (record 0).
Failed to mount '/dev/sdj1': 输入/输出错误
NTFS is either inconsistent, or there is a hardware fault, or it's a
SoftRAID/FakeRAID hardware. In the first case run chkdsk /f on Windows
then reboot into Windows twice. The usage of the /f parameter is very
important! If the device is a SoftRAID/FakeRAID then first activate
it and mount a different device under the /dev/mapper/ directory, (e.g.
/dev/mapper/nvidia_eahaabcc1). Please see the 'dmraid' documentation
for more details.

# 使用ntfsck检查
root@uos:~/tmp# /root/ntfsprogs-plus/src/ntfsck /dev/sdn1
Parse #1: Check system files...
Parse #2: Replay logfile...
Parse #3: Check index entries in volume...
Size of non resident(64:128) are corrupted, (new size:0, allocated:475136, aligned:479232 data:479232, init:479232, rls(475136:475136) (y/N)? ^C
root@uos:~/tmp# /root/ntfsprogs-plus/src/ntfsck -n /dev/sdn1
Parse #1: Check system files...
Parse #2: Replay logfile...
Parse #3: Check index entries in volume...
Size of non resident(64:128) are corrupted, (new size:0, allocated:475136, aligned:479232 data:479232, init:479232, rls(475136:475136) (y/N)? N
Allocated size is different (IDX/$FN:479232 MFT/$DATA:475136) on inode(64, 4GB_file.bin). Fix it. (y/N)? N
Parse #3 Errors remains: 2
Parse #4: Check mft entries in volume...
Parse #4 Errors remains: 2
Parse #5: Check cluster bitmap...
Total lcn bitmap clear:66, Total missing lcn bitmap:0
Parse #5 Errors remains: 68
68 errors left (errors:68, fixed:0)

# 再使用ntfs修复
root@uos:~/tmp# /root/ntfsprogs-plus/src/ntfsck -a /dev/sdn1
Parse #1: Check system files...
Parse #2: Replay logfile...
Parse #3: Check index entries in volume...
Size of non resident(64:128) are corrupted, (new size:0, allocated:475136, aligned:479232 data:479232, init:479232, rls(475136:475136) (y/N)? y
Allocated size is different (IDX/$FN:479232 MFT/$DATA:0) on inode(64, 4GB_file.bin). Fix it. (y/N)? y
Parse #4: Check mft entries in volume...
Parse #5: Check cluster bitmap...
Total lcn bitmap clear:66, Total missing lcn bitmap:0
Clean, No errors found or left (errors:68, fixed:68)

# 看到修复成功了

# 再次查看文件
root@uos:/mnt# ls -alh
总计 8.5K
drwxrwxrwx  1 root root 4.0K 12月 5日 14:27 .
drwxr-xr-x 21 root root 4.0K 12月 4日 14:21 ..
-rwxrwxrwx  1 root root    0 12月 5日 14:32 4GB_file.bin
-rwxrwxrwx  1 root root    6 12月 5日 14:22 hello.txt
drwxrwxrwx  1 root root    0 12月 5日 14:24 lost+found
# 可以看到文件被清空了

# 情况二 文件修复失败了 文件也丢失完了
root@uos:~/tmp# /root/ntfsprogs-plus/src/ntfsck -a /dev/sdj1
$MFTMirr does not match $MFT (record 0). Correcting differences in $MFT record (y/N)? y
Incomplete multi-sector transfer: magic: 0x58444e49  size: 4096  usa_ofs: 40  usa_count: 7  data: 50271  usn: 50270: Input/output error
Corrupt index block signature: vcn(0) inode(5)
Failed to open $Secure: No such file or directory
ntfsck mount failed, errno : 2
1 errors left (errors:2, fixed:1)
root@uos:~/tmp# /root/ntfsprogs-plus/src/ntfsck -a /dev/sdj1
Incomplete multi-sector transfer: magic: 0x58444e49  size: 4096  usa_ofs: 40  usa_count: 7  data: 50271  usn: 50270: Input/output error
Corrupt index block signature: vcn(0) inode(5)
Failed to open $Secure: No such file or directory
ntfsck mount failed, errno : 2
1 errors left (errors:1, fixed:0)

root@uos:~/tmp# mount /dev/sdj1 /mnt
Incomplete multi-sector transfer: magic: 0x58444e49  size: 4096  usa_ofs: 40  usa_count: 7  data: 50271  usn: 50270: 输入/输出错误
Corrupt index block signature: vcn 0 inode 5
Failed to open $Secure: 没有那个文件或目录
Failed to mount '/dev/sdj1': 没有那个文件或目录

```

###### 4) ntfsplus 小文件测试

```
for((i = 0; i < 60000; i ++)); do cp test.txt test_b$i.txt; done
```



###### 5) ntfs3 小文件测试

```
root@uos:~# mount -t ntfs3 /dev/sdae1 /mnt
mount: /mnt: wrong fs type, bad option, bad superblock on /dev/sdae1, missing codepage or helper program, or other error.
       dmesg(1) may have more information after failed mount system call.

```



###### 6) ntfs-3g小文件测试

```
root@uos:~# mount /dev/sdad1 /mnt
$MFTMirr does not match $MFT (record 0).
Failed to mount '/dev/sdad1': 输入/输出错误
NTFS is either inconsistent, or there is a hardware fault, or it's a
SoftRAID/FakeRAID hardware. In the first case run chkdsk /f on Windows
then reboot into Windows twice. The usage of the /f parameter is very
important! If the device is a SoftRAID/FakeRAID then first activate
it and mount a different device under the /dev/mapper/ directory, (e.g.
/dev/mapper/nvidia_eahaabcc1). Please see the 'dmraid' documentation


root@uos:~# /root/ntfsprogs-plus/src/ntfsck -n /dev/sdad1
$MFTMirr does not match $MFT (record 0). Correcting differences in $MFT record (y/N)? N
ntfsck mount failed, errno : 5
2 errors left (errors:2, fixed:0)
root@uos:~# /root/ntfsprogs-plus/src/ntfsck -a /dev/sdad1
$MFTMirr does not match $MFT (record 0). Correcting differences in $MFT record (y/N)? y
ntfs_mst_post_read_fixup_warn: magic: 0x2078756e  size: 4096   usa_ofs: 25963  usa_count: 28274: Invalid argument
Corrupt index block signature: vcn(91) inode(5)
Failed to open $Secure: No such file or directory
ntfsck mount failed, errno : 2
1 errors left (errors:2, fixed:1)
```

##### 四、修复工具ntfsck测试

```

```

##### 五、小文件删除测试

###### 1) ntfsplus 小文件删除

测试1

```bash
root@uos:~/tmp# time cp randomfile/* /mnt

real    0m16.153s
user    0m0.582s
sys     0m3.628s
root@uos:~/tmp# time sync
              
real    11m24.048s
user    0m0.002s
sys     0m0.004s

# 删除文件
root@uos:/mnt# time rm *

real    0m11.257s
user    0m0.168s
sys     0m1.126s
root@uos:/mnt# time sync

real    0m35.677s
user    0m0.000s
sys     0m0.005s
root@uos:/mnt# 

root@uos:/mnt# time -p rm *
real 922.53
user 0.43
sys 5.98
```

测试2

```bash
root@uos:~/tmp# time cp randomfile/* /mnt

real    0m15.688s
user    0m0.567s
sys     0m3.784s
root@uos:~/tmp# time sync

real    10m48.586s
user    0m0.000s
sys     0m0.006s

root@uos:/mnt# time rm *

real    0m10.685s
user    0m0.146s
sys     0m1.157s
root@uos:/mnt# time sync
  
real    0m44.935s
user    0m0.001s
sys     0m0.001s
```

测试3

```
root@uos:~/tmp# time cp randomfile/* /mnt && time sync

real    0m16.079s
user    0m0.557s
sys     0m3.584s

real    11m22.036s
user    0m0.000s
sys     0m0.005s



root@uos:/mnt# time rm * && time sync

real    0m11.190s
user    0m0.274s
sys     0m1.050s

real    0m47.789s
user    0m0.000s
sys     0m0.001s
```



结论是写入很慢  删除较快

之前的现象是因为还没有写入完全 导致删除的时间为写入时间加删除时间

###### 4) ntfs3 

```bash
# 复制时间
root@uos:~/tmp# time cp randomfile/* /mnt

real    0m12.538s
user    0m0.581s
sys     0m2.637s
root@uos:~/tmp# time sync

real    2m13.204s
user    0m0.001s
sys     0m0.005s

# 删除时间
root@uos:/mnt# time rm *

real    0m0.633s
user    0m0.121s
sys     0m0.499s
root@uos:/mnt# time sync
                                                     
real    0m43.623s
user    0m0.000s
sys     0m0.005s

```

 测试2

````bash
root@uos:~/tmp# time cp randomfile/* /mnt

real    0m12.460s
user    0m0.560s
sys     0m2.624s
root@uos:~/tmp# time sync

real    2m14.644s
user    0m0.001s
sys     0m0.005s

root@uos:/mnt# time rm *

real    0m0.666s
user    0m0.140s
sys     0m0.511s
root@uos:/mnt# time sync

real    0m13.495s
user    0m0.000s
sys     0m0.003s
````

测试3

```bash
root@uos:~/tmp# time cp randomfile/* /mnt && time sync

real    0m12.473s
user    0m0.544s
sys     0m2.639s

real    2m16.716s
user    0m0.000s
sys     0m0.003s

root@uos:/mnt# time rm * && time sync

real    0m0.667s
user    0m0.129s
sys     0m0.524s

real    0m49.868s
user    0m0.000s
sys     0m0.002s
```



###### 5) ntfs-3g

测试1

```bash
# 写入
root@uos:~/tmp# time cp randomfile/* /mnt && time sync

real    0m17.724s
user    0m0.630s
sys     0m3.102s
                                    
real    5m32.690s
user    0m0.000s
sys     0m0.604s


# 删除
root@uos:/mnt# time rm * && time sync

real    0m3.650s
user    0m0.156s
sys     0m0.586s

real    0m52.840s
user    0m0.000s
sys     0m0.140s
```

###### 4) ext4 文件对照

```bash
# 复制时间
root@uos:~/tmp# time cp randomfile/* /mnt

real    0m10.450s
user    0m0.548s
sys     0m2.710s
root@uos:~/tmp# time sync

real    2m2.275s
user    0m0.000s
sys     0m0.018s

# 删除时间
root@uos:/mnt# time rm *          
rm: 无法删除 'lost+found': 是一个目录

real    0m0.972s
user    0m0.166s
sys     0m0.695s

root@uos:/mnt# time sync

real    0m3.350s
user    0m0.000s
sys     0m0.019s

```

##### 六、追踪sync性能测试

###### 1) 先整个模块的追踪看结果

```bash
trace-cmd record -p function_graph --module ntfsplus --max-graph-depth 10
trace-cmd report > ntfsplus.log

```



##### 结论

1. ntfsplus最稳定不会出错，复制时中断文件系统不会损坏，已传输文件部分会被保留

2. ntfs3 每次用ntfsck检查会出现1-2个错误 有概率出现很多 (几十到几k不等) 以上的错误，但是修复后可以正常使用，文件有概率损坏 (五次出现一次)

3. ntfs-3g 刚开始测试的时候 文件系统会损坏，错误在60左右，能修复，传输文件会损坏，其他文件正常

   第二次测试 两个错误 一个错误无法修复，文件系统损坏