#### ntfsplus 小文件测试

#### 一、测试环境说明

~~~
i3测试机+amd独显
uosv20(baseline)
内存大小: 16G
测试u盘型号  UDisk 230614
测试固态型号 FORES L02090
~~~

#### 二、测试目的

在研究ntfsplus的io性能和稳定性的时候，发现ntfsplus在大量小文件删除的时候耗时非常长，经过初步研究发现是在写入后sync的耗时严重导致删除等待时间长，于是采用工具fs_mark测试ntfsplus小文件同步性能

#### 二、测试结论

ntfsplus在处理每个小文件同步写入时存在严重的阻塞问题。在u盘上ntfsplus的吞吐量只有ntfs3的28.5%, 是ext4的54.4%,  在固态硬盘上ntfsplus的吞吐量只有ntfs3的22.7%, 是ext4的35.3%

#### 三、测试结果

###### 说明

先查看测试前的磁盘状态 `cat /sys/block/xxx/stat` 然后使用 `iostat -x 1`记录测试过程的io数据，并使用time记录测试过程使用的时间，测试完成后再检查磁盘状态 `cat /sys/block/xxx/stat`，并做差值对比，输出测试中io操作等基本数据

并且分别测试ntfsplus ntfs3 ext4 在 u盘设备和固态磁盘上的表现，并作对比

###### fs_mark测试参数

```bash
time (fs_mark  -d  /mnt  -s  4096  -n  1000  -D  10  -t  1  -S  1  -L  5)
```

##### 1. USB

###### 1) ntfsplus

~~~bash
cat /sys/block/sdag/stat
=== 测试前磁盘状态 (/sys/block/sdag/stat) ===
  184367       60  1670125  1053383  4270506  1443792 34794938 155382825        0 155501725 156436208        0        0        0        0        0        0

# fs_mark测试命令
#  ./fs_mark  -d  /mnt  -s  4096  -n  1000  -D  10  -t  1  -S  1  -L  5 
#	Version 3.3, 1 thread(s) starting at Mon Dec 15 16:20:44 2025
#	Sync method: INBAND FSYNC: fsync() per file in write loop.
#	Directories:  Time based hash between directories across 10 subdirectories with 180 seconds per subdirectory.
#	File names: 40 bytes long, (16 initial bytes of time stamp with 24 random bytes at end of name)
#	Files info: size 4096 bytes, written with an IO size of 16384 bytes per write
#	App overhead is time in microseconds spent in the test not doing file writing related system calls.

FSUse%        Count         Size    Files/sec     App Overhead
     0         1000         4096         33.2            33703
     0         2000         4096         47.3            32103
     0         3000         4096         14.2            28345
     0         4000         4096         53.5            29443
     0         5000         4096         19.9            26712
Average Files/sec:         33.2
p50 Files/sec: 33
p90 Files/sec: 14
p99 Files/sec: 14

# fs_mark测试完成所用时间
real	3m10.725s
user	0m0.087s
sys	0m2.282s

cat /sys/block/sdag/stat
=== 测试后磁盘状态 (/sys/block/sdag/stat) ===
  185980       60  1683029  1057083  4320363  1443792 35083680 155567480        0 155689442 156624563        0        0        0        0        0        0

=== 磁盘 I/O 统计差值 ===

读取I/O操作次数: 1613
读取合并次数: 0
读取扇区数: 12904
读取耗时(ms): 3700
写入I/O操作次数: 49857
写入合并次数: 0
写入扇区数: 288742
写入耗时(ms): 184655
当前正在进行的I/O数: 0
I/O耗时(ms): 187717
加权I/O耗时(ms): 188355
~~~

iostat (部分数据)

```
avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           5.93    0.00    1.26   12.25    0.00   80.56

Device            r/s     rkB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wkB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dkB/s   drqm/s  %drqm d_await dareq-sz     f/s f_await  aqu-sz  %util
sdag             1.00      4.00     0.00   0.00    0.00     4.00    1.00      4.00     0.00   0.00  656.00     4.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    0.66  65.70


avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           5.93    0.00    0.88   15.26    0.00   77.93

Device            r/s     rkB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wkB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dkB/s   drqm/s  %drqm d_await dareq-sz     f/s f_await  aqu-sz  %util
sdag             2.00      8.00     0.00   0.00    1.00     4.00   24.00     61.00     0.00   0.00   55.33     2.54    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    1.33 133.10


avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           6.31    0.00    1.26   12.25    0.00   80.18

Device            r/s     rkB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wkB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dkB/s   drqm/s  %drqm d_await dareq-sz     f/s f_await  aqu-sz  %util
sdag             6.00     24.00     0.00   0.00    0.83     4.00  164.00    482.00     0.00   0.00    5.05     2.94    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    0.83  83.50


avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           7.05    0.00    1.51   14.36    0.00   77.08

Device            r/s     rkB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wkB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dkB/s   drqm/s  %drqm d_await dareq-sz     f/s f_await  aqu-sz  %util
sdag             5.00     20.00     0.00   0.00    1.00     4.00   98.00    290.00     0.00   0.00    8.05     2.96    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    0.79  79.20


avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           6.29    0.00    1.26   12.20    0.00   80.25

Device            r/s     rkB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wkB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dkB/s   drqm/s  %drqm d_await dareq-sz     f/s f_await  aqu-sz  %util
sdag             4.00     16.00     0.00   0.00    1.00     4.00   97.00    290.00     0.00   0.00    7.76     2.99    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    0.76  75.90


avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           6.68    0.00    1.26   14.23    0.00   77.83

Device            r/s     rkB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wkB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dkB/s   drqm/s  %drqm d_await dareq-sz     f/s f_await  aqu-sz  %util
sdag             4.00     16.00     0.00   0.00    1.00     4.00   97.00    290.00     0.00   0.00   14.41     2.99    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    1.40 140.20
```



###### 2) ntfs3

~~~bash
cat /sys/block/sdag/stat
=== 测试前磁盘状态 (/sys/block/sdag/stat) ===
  186116       60  1684117  1057122  4326229  1443792 35098186 155580243        0 155702314 156637365        0        0        0        0        0        0

# fs_mark测试命令
#  ./fs_mark  -d  /mnt  -s  4096  -n  1000  -D  10  -t  1  -S  1  -L  5 
#	Version 3.3, 1 thread(s) starting at Mon Dec 15 16:24:42 2025
#	Sync method: INBAND FSYNC: fsync() per file in write loop.
#	Directories:  Time based hash between directories across 10 subdirectories with 180 seconds per subdirectory.
#	File names: 40 bytes long, (16 initial bytes of time stamp with 24 random bytes at end of name)
#	Files info: size 4096 bytes, written with an IO size of 16384 bytes per write
#	App overhead is time in microseconds spent in the test not doing file writing related system calls.

FSUse%        Count         Size    Files/sec     App Overhead
     0         1000         4096        119.7            30848
     0         2000         4096        138.9            33325
     0         3000         4096        117.3            36245
     0         4000         4096         79.3            33278
     0         5000         4096        128.7            35116
Average Files/sec:        116.2
p50 Files/sec: 119
p90 Files/sec: 79
p99 Files/sec: 79

# fs_mark测试完成所用时间
real	0m44.474s
user	0m0.093s
sys	0m1.256s

cat /sys/block/sdag/stat
=== 测试后磁盘状态 (/sys/block/sdag/stat) ===
  187372       60  1694165  1059654  4341299  1443800 35218810 155620669        0 155745388 156680324        0        0        0        0        0        0

=== 磁盘 I/O 统计差值 ===

读取I/O操作次数: 1256
读取合并次数: 0
读取扇区数: 10048
读取耗时(ms): 2532
写入I/O操作次数: 15070
写入合并次数: 8
写入扇区数: 120624
写入耗时(ms): 40426
当前正在进行的I/O数: 0
I/O耗时(ms): 43074
加权I/O耗时(ms): 42959

~~~

iostat (部分数据)

~~~
avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           4.67    0.00    2.40   11.73    0.00   81.21

Device            r/s     rkB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wkB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dkB/s   drqm/s  %drqm d_await dareq-sz     f/s f_await  aqu-sz  %util
sdag            65.00    260.00     0.00   0.00    0.94     4.00  773.00   3092.00     0.00   0.00    1.90     4.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    1.53 153.10


avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           5.67    0.00    1.26   12.47    0.00   80.60

Device            r/s     rkB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wkB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dkB/s   drqm/s  %drqm d_await dareq-sz     f/s f_await  aqu-sz  %util
sdag             3.00     12.00     0.00   0.00    1.00     4.00   42.00    168.00     0.00   0.00   16.40     4.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    0.69  69.30


avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           4.41    0.00    1.26   12.11    0.00   82.22

Device            r/s     rkB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wkB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dkB/s   drqm/s  %drqm d_await dareq-sz     f/s f_await  aqu-sz  %util
sdag            46.00    184.00     0.00   0.00    0.91     4.00  547.00   2188.00     0.00   0.00    2.24     4.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    1.27 127.40


avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           4.30    0.00    1.14   12.15    0.00   82.41

Device            r/s     rkB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wkB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dkB/s   drqm/s  %drqm d_await dareq-sz     f/s f_await  aqu-sz  %util
sdag            20.00     80.00     0.00   0.00    1.00     4.00  238.00    952.00     0.00   0.00    3.99     4.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    0.97  97.60


avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           3.65    0.00    1.38   12.20    0.00   82.77

Device            r/s     rkB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wkB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dkB/s   drqm/s  %drqm d_await dareq-sz     f/s f_await  aqu-sz  %util
sdag            51.00    204.00     0.00   0.00    0.88     4.00  613.00   2452.00     0.00   0.00    1.00     4.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    0.66  65.30


avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           5.18    0.00    1.77   15.91    0.00   77.15

Device            r/s     rkB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wkB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dkB/s   drqm/s  %drqm d_await dareq-sz     f/s f_await  aqu-sz  %util
sdag            11.00     44.00     0.00   0.00    0.82     4.00  138.00    552.00     0.00   0.00    5.74     4.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    0.80  80.60
~~~

###### 3)  ext4

~~~bash
cat /sys/block/sdag/stat
=== 测试前磁盘状态 (/sys/block/sdag/stat) ===
  187506       61  1700663  1060405  4342762  1444826 35536858 155649400        0 155774818 156709805        0        0        0        0        0        0

# fs_mark测试命令
#  ./fs_mark  -d  /mnt  -s  4096  -n  1000  -D  10  -t  1  -S  1  -L  5 
#	Version 3.3, 1 thread(s) starting at Mon Dec 15 16:29:05 2025
#	Sync method: INBAND FSYNC: fsync() per file in write loop.
#	Directories:  Time based hash between directories across 10 subdirectories with 180 seconds per subdirectory.
#	File names: 40 bytes long, (16 initial bytes of time stamp with 24 random bytes at end of name)
#	Files info: size 4096 bytes, written with an IO size of 16384 bytes per write
#	App overhead is time in microseconds spent in the test not doing file writing related system calls.

FSUse%        Count         Size    Files/sec     App Overhead
     6         1000         4096         82.8            28421
     6         2000         4096         38.7            30536
     6         3000         4096         66.5            35250
     6         4000         4096         55.5            35083
     6         5000         4096         64.0            32211
Average Files/sec:         61.0
p50 Files/sec: 64
p90 Files/sec: 38
p99 Files/sec: 38

# fs_mark测试完成所用时间
real	1m26.631s
user	0m0.091s
sys	0m0.909s

cat /sys/block/sdag/stat
=== 测试后磁盘状态 (/sys/block/sdag/stat) ===
  187508       61  1700679  1060407  4358618  1476437 36060794 155734805        0 155860163 156795212        0        0        0        0        0        0

=== 磁盘 I/O 统计差值 ===

读取I/O操作次数: 2
读取合并次数: 0
读取扇区数: 16
读取耗时(ms): 2
写入I/O操作次数: 15825
写入合并次数: 31610
写入扇区数: 517400
写入耗时(ms): 84543
当前正在进行的I/O数: 0
I/O耗时(ms): 84482
加权I/O耗时(ms): 84544
~~~

iostat (部分)

~~~
avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           3.24    0.00    1.06    5.51    0.00   90.19

Device            r/s     rkB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wkB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dkB/s   drqm/s  %drqm d_await dareq-sz     f/s f_await  aqu-sz  %util
sdag             0.22      0.99     0.00   0.03    5.66     4.53    5.05     20.66     1.68  24.96   35.84     4.09    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    0.18  18.11


avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           6.92    0.00    1.26    8.30    0.00   83.52

Device            r/s     rkB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wkB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dkB/s   drqm/s  %drqm d_await dareq-sz     f/s f_await  aqu-sz  %util
sdag             0.00      0.00     0.00   0.00    0.00     0.00   11.00   1208.00     0.00   0.00   64.00   109.82    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    0.70  70.50


avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           6.78    0.00    1.51    3.02    0.00   88.69

Device            r/s     rkB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wkB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dkB/s   drqm/s  %drqm d_await dareq-sz     f/s f_await  aqu-sz  %util
sdag             0.00      0.00     0.00   0.00    0.00     0.00   20.00   2060.00     1.00   4.76    7.90   103.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    0.16  15.80


avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           6.31    0.00    1.77   10.86    0.00   81.06

Device            r/s     rkB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wkB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dkB/s   drqm/s  %drqm d_await dareq-sz     f/s f_await  aqu-sz  %util
sdag             1.00      4.00     0.00   0.00    0.00     4.00  193.00   4096.00   343.00  63.99    4.96    21.22    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    0.96  95.70


avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           6.19    0.00    1.52   17.05    0.00   75.25

Device            r/s     rkB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wkB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dkB/s   drqm/s  %drqm d_await dareq-sz     f/s f_await  aqu-sz  %util
sdag             0.00      0.00     0.00   0.00    0.00     0.00  218.00   4468.00   399.00  64.67    4.46    20.50    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    0.97  96.80


avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           5.03    0.00    1.13   16.35    0.00   77.48

Device            r/s     rkB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wkB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dkB/s   drqm/s  %drqm d_await dareq-sz     f/s f_await  aqu-sz  %util
sdag             0.00      0.00     0.00   0.00    0.00     0.00  297.00   4120.00   588.00  66.44    3.31    13.87    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    0.98  98.50


avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           6.30    0.00    0.88   14.74    0.00   78.09

Device            r/s     rkB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wkB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dkB/s   drqm/s  %drqm d_await dareq-sz     f/s f_await  aqu-sz  %util
sdag             0.00      0.00     0.00   0.00    0.00     0.00   15.00   1480.00     6.00  28.57   48.07    98.67    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    0.72  71.80

~~~

##### 2.NVME

###### 1) ntfsplus

~~~bash
cat /sys/block/nvme0n1/stat
=== 测试前磁盘状态 (/sys/block/nvme0n1/stat) ===
 1732281   341469 176842452   454859 14053692 76158335 810107172 235240160        0  1280963 235783940     6917        0 183230232    19528   130422    69391

# fs_mark测试命令
#  ./fs_mark  -d  /mnt  -s  4096  -n  1000  -D  10  -t  1  -S  1  -L  5 
#	Version 3.3, 1 thread(s) starting at Mon Dec 15 16:40:28 2025
#	Sync method: INBAND FSYNC: fsync() per file in write loop.
#	Directories:  Time based hash between directories across 10 subdirectories with 180 seconds per subdirectory.
#	File names: 40 bytes long, (16 initial bytes of time stamp with 24 random bytes at end of name)
#	Files info: size 4096 bytes, written with an IO size of 16384 bytes per write
#	App overhead is time in microseconds spent in the test not doing file writing related system calls.

FSUse%        Count         Size    Files/sec     App Overhead
     0         1000         4096        522.3            24621
     0         2000         4096        494.6            25377
     0         3000         4096        528.7            22965
     0         4000         4096        465.1            28956
     0         5000         4096        470.5            29345
Average Files/sec:        495.8
p50 Files/sec: 494
p90 Files/sec: 465
p99 Files/sec: 465

# fs_mark测试完成所用时间
real	0m10.109s
user	0m0.089s
sys	0m1.425s

cat /sys/block/nvme0n1/stat
=== 测试后磁盘状态 (/sys/block/nvme0n1/stat) ===
 1733858   341469 176855068   456766 14099380 76158355 810328376 235246930        0  1289631 235798308     6917        0 183230232    19528   135424    75082

=== 磁盘 I/O 统计差值 ===

读取I/O操作次数: 1577
读取合并次数: 0
读取扇区数: 12616
读取耗时(ms): 1907
写入I/O操作次数: 45688
写入合并次数: 20
写入扇区数: 221204
写入耗时(ms): 6770
当前正在进行的I/O数: 0
I/O耗时(ms): 8668
加权I/O耗时(ms): 14368
~~~

 iostat 

~~~
avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           4.97    0.00    2.55   10.32    0.00   82.17

Device            r/s     rkB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wkB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dkB/s   drqm/s  %drqm d_await dareq-sz     f/s f_await  aqu-sz  %util
nvme0n1        152.00    608.00     0.00   0.00    1.54     4.00 4436.00  10704.00     0.00   0.00    0.14     2.41    0.00      0.00     0.00   0.00    0.00     0.00  488.00    1.05    1.37  86.90


avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           4.56    0.00    2.53   10.52    0.00   82.38

Device            r/s     rkB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wkB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dkB/s   drqm/s  %drqm d_await dareq-sz     f/s f_await  aqu-sz  %util
nvme0n1        161.00    644.00     0.00   0.00    1.24     4.00 4625.00  11158.00     0.00   0.00    0.15     2.41    0.00      0.00     0.00   0.00    0.00     0.00  507.00    1.09    1.43  85.10


avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           5.18    0.00    2.91   10.11    0.00   81.80

Device            r/s     rkB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wkB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dkB/s   drqm/s  %drqm d_await dareq-sz     f/s f_await  aqu-sz  %util
nvme0n1        165.00    660.00     0.00   0.00    0.84     4.00 4727.00  11441.00     0.00   0.00    0.15     2.42    0.00      0.00     0.00   0.00    0.00     0.00  517.00    1.17    1.46  85.70


avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           3.79    0.00    2.27   10.98    0.00   82.95

Device            r/s     rkB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wkB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dkB/s   drqm/s  %drqm d_await dareq-sz     f/s f_await  aqu-sz  %util
nvme0n1        171.00    684.00     0.00   0.00    0.96     4.00 4928.00  11960.00     0.00   0.00    0.15     2.43    0.00      0.00     0.00   0.00    0.00     0.00  538.00    1.18    1.53  90.00


avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           5.58    0.00    2.92    9.90    0.00   81.60

Device            r/s     rkB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wkB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dkB/s   drqm/s  %drqm d_await dareq-sz     f/s f_await  aqu-sz  %util
nvme0n1        135.00    540.00     0.00   0.00    1.70     4.00 3970.00   9615.00     9.00   0.23    0.15     2.42    0.00      0.00     0.00   0.00    0.00     0.00  434.00    1.15    1.33  82.70


avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           5.18    0.00    3.03    9.97    0.00   81.82

Device            r/s     rkB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wkB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dkB/s   drqm/s  %drqm d_await dareq-sz     f/s f_await  aqu-sz  %util
nvme0n1        151.00    604.00     0.00   0.00    1.09     4.00 4405.00  10711.00     0.00   0.00    0.15     2.43    0.00      0.00     0.00   0.00    0.00     0.00  481.00    1.19    1.42  84.00


avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           5.85    0.00    3.05    9.92    0.00   81.17

Device            r/s     rkB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wkB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dkB/s   drqm/s  %drqm d_await dareq-sz     f/s f_await  aqu-sz  %util
nvme0n1        139.00    556.00     0.00   0.00    1.56     4.00 4081.00   9889.00     0.00   0.00    0.15     2.42    0.00      0.00     0.00   0.00    0.00     0.00  446.00    1.16    1.35  82.80


avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           4.68    0.00    2.53   10.63    0.00   82.15

Device            r/s     rkB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wkB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dkB/s   drqm/s  %drqm d_await dareq-sz     f/s f_await  aqu-sz  %util
nvme0n1        156.00    624.00     0.00   0.00    1.19     4.00 4523.00  10976.00     0.00   0.00    0.15     2.43    0.00      0.00     0.00   0.00    0.00     0.00  493.00    1.17    1.46  88.60

~~~

###### 2) ntfs3

~~~bash
cat /sys/block/nvme0n1/stat
=== 测试前磁盘状态 (/sys/block/nvme0n1/stat) ===
 1730895   341469 176830221   454801 14033385 76156939 809947946 235237969        0  1278948 235780150     6917        0 183230232    19528   125407    67850

# fs_mark测试命令
#  ./fs_mark  -d  /mnt  -s  4096  -n  1000  -D  10  -t  1  -S  1  -L  5 
#	Version 3.3, 1 thread(s) starting at Mon Dec 15 16:38:41 2025
#	Sync method: INBAND FSYNC: fsync() per file in write loop.
#	Directories:  Time based hash between directories across 10 subdirectories with 180 seconds per subdirectory.
#	File names: 40 bytes long, (16 initial bytes of time stamp with 24 random bytes at end of name)
#	Files info: size 4096 bytes, written with an IO size of 16384 bytes per write
#	App overhead is time in microseconds spent in the test not doing file writing related system calls.

FSUse%        Count         Size    Files/sec     App Overhead
     0         1000         4096       2294.7            24111
     0         2000         4096       1884.0            16647
     0         3000         4096       2284.9            14561
     0         4000         4096       2205.5            16181
     0         5000         4096       2227.8            15304
Average Files/sec:       2178.8
p50 Files/sec: 2227
p90 Files/sec: 1884
p99 Files/sec: 1884

# fs_mark测试完成所用时间
real	0m2.312s
user	0m0.042s
sys	0m0.284s

cat /sys/block/nvme0n1/stat
=== 测试后磁盘状态 (/sys/block/nvme0n1/stat) ===
 1732204   341469 176840693   454843 14053407 76156941 810069642 235239938        0  1280927 235783695     6917        0 183230232    19528   130407    69384

=== 磁盘 I/O 统计差值 ===

读取I/O操作次数: 1309
读取合并次数: 0
读取扇区数: 10472
读取耗时(ms): 42
写入I/O操作次数: 20001
写入合并次数: 0
写入扇区数: 120008
写入耗时(ms): 1961
当前正在进行的I/O数: 0
I/O耗时(ms): 1978
加权I/O耗时(ms): 3537
~~~

iostat

```
avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           3.24    0.00    1.06    5.51    0.00   90.19

Device            r/s     rkB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wkB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dkB/s   drqm/s  %drqm d_await dareq-sz     f/s f_await  aqu-sz  %util
nvme0n1          2.01    102.72     0.40  16.48    0.26    51.08   16.30    470.51    88.48  84.44   16.76    28.86    0.01    106.44     0.00   0.00    2.82 13244.92    0.15    0.54    0.27   0.15


avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           5.18    0.00    0.88    0.00    0.00   93.94

Device            r/s     rkB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wkB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dkB/s   drqm/s  %drqm d_await dareq-sz     f/s f_await  aqu-sz  %util
nvme0n1          0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    0.00   0.00


avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           5.79    0.00    1.01    0.00    0.00   93.20

Device            r/s     rkB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wkB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dkB/s   drqm/s  %drqm d_await dareq-sz     f/s f_await  aqu-sz  %util
nvme0n1          0.00      0.00     0.00   0.00    0.00     0.00   21.00    844.00     2.00   8.70    0.38    40.19    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    0.01   0.10


avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           3.26    0.00    2.63   10.41    0.00   83.69

Device            r/s     rkB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wkB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dkB/s   drqm/s  %drqm d_await dareq-sz     f/s f_await  aqu-sz  %util
nvme0n1        541.00   2164.00     0.00   0.00    0.03     4.00 8252.00  24760.00     0.00   0.00    0.10     3.00    0.00      0.00     0.00   0.00    0.00     0.00 2062.00    0.31    1.49  84.70


avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           3.40    0.00    2.01   10.44    0.00   84.15

Device            r/s     rkB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wkB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dkB/s   drqm/s  %drqm d_await dareq-sz     f/s f_await  aqu-sz  %util
nvme0n1        768.00   3072.00     0.00   0.00    0.03     4.00 8936.00  26808.00     0.00   0.00    0.09     3.00    0.00      0.00     0.00   0.00    0.00     0.00 2234.00    0.31    1.56  85.30
```

###### 3) ext4

~~~bash
cat /sys/block/nvme0n1/stat
=== 测试前磁盘状态 (/sys/block/nvme0n1/stat) ===
 1730640   341469 176826290   454760  9365580 38698939 767381609 173929767        0  1158333 174469319     6917        0 183230232    19528   120006    65263

# fs_mark测试命令
#  ./fs_mark  -d  /mnt  -s  4096  -n  1000  -D  10  -t  1  -S  1  -L  5 
#	Version 3.3, 1 thread(s) starting at Mon Dec 15 16:32:14 2025
#	Sync method: INBAND FSYNC: fsync() per file in write loop.
#	Directories:  Time based hash between directories across 10 subdirectories with 180 seconds per subdirectory.
#	File names: 40 bytes long, (16 initial bytes of time stamp with 24 random bytes at end of name)
#	Files info: size 4096 bytes, written with an IO size of 16384 bytes per write
#	App overhead is time in microseconds spent in the test not doing file writing related system calls.

FSUse%        Count         Size    Files/sec     App Overhead
     5         1000         4096       1392.4            18768
     5         2000         4096       1367.6            14322
     5         3000         4096       1419.8            15063
     5         4000         4096       1417.6            15359
     5         5000         4096       1434.2            13868
Average Files/sec:       1405.8
p50 Files/sec: 1417
p90 Files/sec: 1367
p99 Files/sec: 1367

# fs_mark测试完成所用时间
real	0m3.562s
user	0m0.054s
sys	0m0.245s

cat /sys/block/nvme0n1/stat
=== 测试后磁盘状态 (/sys/block/nvme0n1/stat) ===
 1730642   341469 176826306   454761  9381129 38731318 767763601 173932816        0  1161340 174474778     6917        0 183230232    19528   125186    67672

=== 磁盘 I/O 统计差值 ===

读取I/O操作次数: 2
读取合并次数: 0
读取扇区数: 16
读取耗时(ms): 1
写入I/O操作次数: 15547
写入合并次数: 32360
写入扇区数: 381824
写入耗时(ms): 3049
当前正在进行的I/O数: 0
I/O耗时(ms): 3007
加权I/O耗时(ms): 5457
~~~

iostat

```
           3.24    0.00    1.06    5.51    0.00   90.19

Device            r/s     rkB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wkB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dkB/s   drqm/s  %drqm d_await dareq-sz     f/s f_await  aqu-sz  %util
nvme0n1          2.01    102.77     0.40  16.48    0.26    51.09   10.89    445.98    44.98  80.51   18.57    40.97    0.01    106.49     0.00   0.00    2.82 13244.92    0.14    0.54    0.20   0.13


avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           6.93    0.00    1.01    0.00    0.00   92.07

Device            r/s     rkB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wkB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dkB/s   drqm/s  %drqm d_await dareq-sz     f/s f_await  aqu-sz  %util
nvme0n1          0.00      0.00     0.00   0.00    0.00     0.00    2.00     84.00    19.00  90.48    0.00    42.00    0.00      0.00     0.00   0.00    0.00     0.00    1.00    1.00    0.00   0.00


avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           6.40    0.00    1.13    0.00    0.00   92.47

Device            r/s     rkB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wkB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dkB/s   drqm/s  %drqm d_await dareq-sz     f/s f_await  aqu-sz  %util
nvme0n1          0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    0.00   0.00


avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           4.04    0.00    2.65   10.10    0.00   83.21

Device            r/s     rkB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wkB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dkB/s   drqm/s  %drqm d_await dareq-sz     f/s f_await  aqu-sz  %util
nvme0n1          1.00      4.00     0.00   0.00    0.00     4.00 4229.00  52164.00  8812.00  67.57    0.19    12.33    0.00      0.00     0.00   0.00    0.00     0.00 1410.00    0.47    1.48  81.10


avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           2.66    0.00    1.90   10.89    0.00   84.56

Device            r/s     rkB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wkB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dkB/s   drqm/s  %drqm d_await dareq-sz     f/s f_await  aqu-sz  %util
nvme0n1          1.00      4.00     0.00   0.00    1.00     4.00 4067.00  53428.00  9305.00  69.59    0.21    13.14    0.00      0.00     0.00   0.00    0.00     0.00 1355.00    0.49    1.53  86.70


avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           3.02    0.00    2.26   10.68    0.00   84.05

Device            r/s     rkB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wkB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dkB/s   drqm/s  %drqm d_await dareq-sz     f/s f_await  aqu-sz  %util
nvme0n1          0.00      0.00     0.00   0.00    0.00     0.00 4658.00  55572.00  9340.00  66.72    0.18    11.93    0.00      0.00     0.00   0.00    0.00     0.00 1551.00    0.45    1.56  84.00


```

#### 四、附录

##### 测试脚本

```
#!/bin/bash

# ================================
# 用户配置区域 - 请根据实际情况修改
# ================================

# 文件系统类型 (ext4, ntfs3, ntfsplus 等)
FILESYSTEM_TYPE="ntfsplus"

# 磁盘设备名称 (sdag, nvme0n1 等)
DISK_DEVICE="nvme0n1"

# 测试挂载点
MOUNT_POINT="/mnt"

# 测试参数
FS_MARK_PARAMS="-d $MOUNT_POINT -s 4096 -n 1000 -D 10 -t 1 -S 1 -L 5"

# 输出文件夹和文件前缀
OUTPUT_DIR="./performance_logs"          # 日志文件夹
OUTPUT_PREFIX="perf_test"                # 文件前缀

# 创建日志文件夹
mkdir -p "$OUTPUT_DIR"

# 生成带时间戳的输出文件名
TIMESTAMP=$(date +"%Y%m%d_%H%M%S")
LOG_FILE="${OUTPUT_DIR}/${OUTPUT_PREFIX}_${FILESYSTEM_TYPE}_${DISK_DEVICE}_${TIMESTAMP}.log"
IOSTAT_FILE="${OUTPUT_DIR}/${OUTPUT_PREFIX}_${FILESYSTEM_TYPE}_${DISK_DEVICE}_iostat_${TIMESTAMP}.log"
SUMMARY_FILE="${OUTPUT_DIR}/test_summary.txt"

echo "=============================================="
echo "文件系统性能测试开始"
echo "文件系统: $FILESYSTEM_TYPE"
echo "磁盘设备: $DISK_DEVICE"
echo "挂载点: $MOUNT_POINT"
echo "日志文件夹: $OUTPUT_DIR"
echo "时间: $(date)"
echo "=============================================="
echo ""

# 检查挂载点是否存在
if [ ! -d "$MOUNT_POINT" ]; then
    echo "错误: 挂载点 $MOUNT_POINT 不存在!"
    exit 1
fi

# 检查磁盘设备是否存在
if [ ! -b "/dev/$DISK_DEVICE" ]; then
    echo "错误: 磁盘设备 /dev/$DISK_DEVICE 不存在!"
    exit 1
fi

# 创建日志文件头
{
    echo "=============================================="
    echo "文件系统性能测试报告"
    echo "=============================================="
    echo "测试时间: $(date)"
    echo "文件系统类型: $FILESYSTEM_TYPE"
    echo "磁盘设备: $DISK_DEVICE"
    echo "挂载点: $MOUNT_POINT"
    echo "fs_mark 参数: $FS_MARK_PARAMS"
    echo "日志文件夹: $OUTPUT_DIR"
    echo ""
    
    # 记录磁盘基本信息
    echo "=== 磁盘基本信息 ==="
    echo "设备: /dev/$DISK_DEVICE"
    lsblk "/dev/$DISK_DEVICE" 2>/dev/null || echo "无法获取设备信息"
    echo ""
    
    # 记录挂载信息
    echo "=== 挂载信息 ==="
    mount | grep "$MOUNT_POINT" || echo "未找到挂载信息"
    echo ""
    
    # 记录测试前的磁盘状态
    echo "=== 测试前磁盘状态 (/sys/block/$DISK_DEVICE/stat) ==="
    cat "/sys/block/$DISK_DEVICE/stat" 2>/dev/null || echo "无法读取磁盘状态"
    echo ""
    
    echo "=== 测试开始 ==="
} > "$LOG_FILE"

# 后台启动 iostat 监控
echo "启动 iostat 监控..."
iostat -x "/dev/$DISK_DEVICE" 1 > "$IOSTAT_FILE" &
IOSTAT_PID=$!

# 等待监控启动
sleep 2

# 记录测试前的磁盘统计
echo "记录测试前磁盘统计..."
BEFORE_STATS=$(cat "/sys/block/$DISK_DEVICE/stat" 2>/dev/null)

{
    echo "测试前磁盘统计: $BEFORE_STATS"
    echo ""
} >> "$LOG_FILE"

# 执行性能测试
echo "开始执行 fs_mark 测试..."
{
    echo "执行命令: time (./fs_mark $FS_MARK_PARAMS | tee -a \"$LOG_FILE\") 2> >(tee -a \"$LOG_FILE\")"
    echo ""
    
    # 执行测试并计时
    time (./fs_mark $FS_MARK_PARAMS | tee -a "$LOG_FILE") 2> >(tee -a "$LOG_FILE")
    
    echo ""
    echo "=== 测试完成 ==="
}

# 停止 iostat 监控
echo "停止 iostat 监控..."
kill $IOSTAT_PID 2>/dev/null
wait $IOSTAT_PID 2>/dev/null

# 记录测试后的磁盘状态
{
    echo ""
    echo "=== 测试后磁盘状态 (/sys/block/$DISK_DEVICE/stat) ==="
    AFTER_STATS=$(cat "/sys/block/$DISK_DEVICE/stat" 2>/dev/null || echo "无法读取磁盘状态")
    echo "$AFTER_STATS"
    echo ""
    
    # 计算差值
    if [ -n "$BEFORE_STATS" ] && [ -n "$AFTER_STATS" ]; then
        echo "=== 磁盘 I/O 统计差值 ==="
        echo "说明: 这些数字表示测试期间发生的 I/O 操作次数"
        echo ""
        
        # 将统计信息转换为数组
        IFS=' ' read -r -a before <<< "$BEFORE_STATS"
        IFS=' ' read -r -a after <<< "$AFTER_STATS"
        
        # 定义统计字段的含义
        stat_fields=(
            "读取I/O操作次数" 
            "读取合并次数" 
            "读取扇区数" 
            "读取耗时(ms)" 
            "写入I/O操作次数" 
            "写入合并次数" 
            "写入扇区数" 
            "写入耗时(ms)" 
            "当前正在进行的I/O数" 
            "I/O耗时(ms)" 
            "加权I/O耗时(ms)"
        )
        
        # 计算并显示每个字段的差值
        for i in {0..10}; do
            if [ $i -lt ${#before[@]} ] && [ $i -lt ${#after[@]} ]; then
                diff=$((after[i] - before[i]))
                echo "${stat_fields[$i]}: $diff"
            fi
        done
    fi
    
    echo ""
    echo "=============================================="
    echo "测试完成于: $(date)"
    echo "日志文件: $LOG_FILE"
    echo "I/O统计文件: $IOSTAT_FILE"
    echo "=============================================="
} >> "$LOG_FILE"

# 更新测试摘要文件
{
    echo "=== 测试摘要 ==="
    echo "测试时间: $(date)"
    echo "文件系统: $FILESYSTEM_TYPE"
    echo "磁盘设备: $DISK_DEVICE"
    echo "测试参数: $FS_MARK_PARAMS"
    echo "主要日志: $(basename "$LOG_FILE")"
    echo "I/O统计: $(basename "$IOSTAT_FILE")"
    echo ""
    echo "=== 性能结果 ==="
    grep -E "(Files/sec|real|user|sys)" "$LOG_FILE" | tail -5
    echo ""
} >> "$SUMMARY_FILE"

echo ""
echo "=============================================="
echo "性能测试完成!"
echo "日志文件夹: $OUTPUT_DIR"
echo "主要输出文件: $(basename "$LOG_FILE")"
echo "I/O监控数据: $(basename "$IOSTAT_FILE")"
echo "测试摘要: $(basename "$SUMMARY_FILE")"
echo "=============================================="

# 显示测试结果摘要
echo ""
echo "=== 测试结果摘要 ==="
tail -20 "$LOG_FILE" | grep -E "(Files/sec|real|user|sys|测试完成)" | head -10

# 显示文件夹内容
echo ""
echo "=== 日志文件夹内容 ==="
ls -la "$OUTPUT_DIR"

```



