### 磁盘IO/ 加密/解密磁盘IO

#### 一、磁盘IO过程

##### 1. 普通磁盘IO基本调用栈

```mathematica
应用程序
   ↓
系统调用 (read/write)
   ↓
VFS (虚拟文件系统层)
   ↓
页缓存 (page cache)
   ↓
Block I/O Layer (bio, request queue)
   ↓
I/O 调度器 (mq-deadline / bfq / none)
   ↓
驱动程序
   ↓
硬件控制器 + 磁盘本体
```

##### 2. 加密磁盘使用

~~~mathematica
应用
 ↓
系统调用
 ↓
VFS
 ↓
页缓存 (仍是明文)
 ↓
dm-crypt（加密层）   ← 这里是 加密/解密
 ↓
Block I/O 层
 ↓
驱动
 ↓
磁盘 (数据以密文存储)

~~~

#### 二、加密/解密发生的位置

加密/解密是cpu算法 只会出现在onCPU上 

| 类型             | on/off      | 做什么             |
| ---------------- | ----------- | ------------------ |
| AES/XTS 加密     | **on-CPU**  | 算法执行需要 CPU   |
| AES/XTS 解密     | **on-CPU**  | 同上               |
| 等待磁盘完成读写 | **off-CPU** | CPU 阻塞、进程休眠 |

**主要分析的点**

| 分析地方                 | 工具                                      | 原因               |
| ------------------------ | ----------------------------------------- | ------------------ |
| CPU 花在哪（加密是否重） | `perf top` / `understand CPU flame graph` | 看 AES 算法开销    |
| 是否被磁盘阻塞           | `offcputime-bcc` / `iostat` / `btrace`    | 看 wait time       |
| I/O 延迟情况             | `ioping`, `fio`, `blktrace`               | 判断是否是磁盘瓶颈 |

#### 三、判断性能损失点

CPU 很高，offcpu 低 --> **加密是瓶颈** 

CPU 不高，offcpu 高 --> **磁盘 I/O 是瓶颈**

**主要看**

**你看 on-CPU flamegraph**：是不是 AES 占很大一块？

**你看 off-CPU flamegraph**：是不是 I/O wait 占很大一块？

```
┌─────────────────┬──────────────────┬──────────────────┐
│   阶段1:缓存写入  │   阶段2:缓存饱和   │   阶段3:直写模式   │
├─────────────────┼──────────────────┼──────────────────┤
│ 高速写入(SLC速度) │ 速度开始下降       │ 降至TLC/QLC基础速度│
│ 2000+ MB/s      │ 800-1500 MB/s    │ 100-400 MB/s     │
│ 缓存未满         │ 缓存部分填满       │ 缓存完全饱和       │
└─────────────────┴──────────────────┴──────────────────┘
```



#### 四、测试方法

~~~bash
# 生成堆栈文件
cat cputime_normal.txt | ../tmp/FlameGraph/stackcollapse.pl > normal_offcpu.folded
# 生成那个图片
flamegraph.pl normal_offcpu.folded  > normal_offcpu.svg


cat $1 | ${FlameGraphPath}/stackcollapse.pl > normal_offcpu.folded
${FlameGraphPath}/flamegraph.pl normal_offcpu.folded > normal_offcpu.svg

cat $2 | ${FlameGraphPath}/stackcollapse.pl > sm4_offcpu.folded
${FlameGraphPath}/flamegraph.pl sm4_offcpu.folded > sm4_offcpu.svg

cat $3 | ${FlameGraphPath}/stackcollapse.pl > sm4_offcpu.folded
${FlameGraphPath}/flamegraph.pl aes_offcpu.folded > aes_offcpu.svg

~~~

```bash
# 测试代码 工具为fio

# 测试nor
# write 配置文件
[global]
ioengine=libaio
direct=1
bs=1m
size=4096m
filename=/dev/nvme0n1p6
numjobs=1
iodepth=32
group_reporting=1

[write-test]
rw=write
name=write-performance

# read 配置文件
[global]
ioengine=libaio
direct=1
bs=1m
size=4096m
filename=/dev/nvme0n1p6
numjobs=1
iodepth=32
group_reporting=1

[read-test]
rw=read
name=read-performance



```



#### 五、测试结果 fio

- 概述

```bash
# 裸盘数据
# 512m 
NOR write: 2900 - 3000  read: 2900 - 3000
AES write: 2400 - 2470  read: 2400 - 2500
 
# 4096m 
NOR write: 2900 - 3000  read: 3000 - 3100
AES write: 2400 - 2500  read: 2400 - 2450


# 文件系统数据 无 page cache
# 1024m
NOR write: 2900 - 3000  read: 3000 - 3100
AES write: 2400 - 2450  read: 3000 - 3100

# 2048m
NOR write: 2850 - 3000  read: 3100 - 3100
AES write: 2400 - 2500  read: 3100 - 3100

# 4096m
NOR write: 2930 - 2950  read: 3100 - 3130
AES write: 2350 - 2500  read: 3000 - 3100

# 8192m
NOR write: 2700 - 2800  read: 2600 - 2600
AES write: 2200 - 2270  read: 2400 - 2500

# 文件系统数据 有 page cache
# 1024m
NOR write: 1800 - 1900  read: 1700 - 1800
AES write: 1700 - 1800  read: 900  - 1000

# 2048m
NOR write: 1800 - 1850  read: 1800 - 1830
AES write: 1400 - 1500  read: 900  - 1000

# 4096m
NOR write: 1650 - 1700  read: 1750 - 1900
AES write: 1200 - 1300  read: 1000 - 1100

# 8192m
NOR write: 1650 - 1700  read: 1700 - 1900
AES write: 1200 - 1300  read: 1000 - 1000
```

 

4096组

```bash
# 裸盘数据
# 4096m 
NOR write: 2900 - 3000  read: 3000 - 3100
AES write: 2400 - 2500  read: 2400 - 2450
SM4 write: 2500 - 2540  read: 2500 - 2540

# 中位数计算
AES write loss: 16.9%, read loss: 20.4%
SM4 write loss: 14.5%, read loss: 17.3%


# 文件系统数据 无 page cache
# 4096m
NOR write: 2930 - 2950  read: 3100 - 3130
AES write: 2350 - 2500  read: 3000 - 3100
SM4 write: 2450 - 2550  read: 2700 - 2900

# 中位数计算
AES write loss: 17.5%, read loss: 2.0%
SM4 write loss: 14.5%, read loss: 10.1%


# 文件系统数据 有 page cache
# 4096m
NOR write: 1650 - 1700  read: 1750 - 1900
AES write: 1200 - 1300  read: 1000 - 1100
AES write: 1290 - 1350  read: 1100 - 1150

# 中位数计算
AES write loss: 25.3%, read loss: 41.6%
SM4 write loss: 21.1%, read loss: 37.6%
```







##### 1) 裸盘测试结果

1. **512m组 普通磁盘测试 读**

``` bash
# 配置文件
[global]
ioengine=libaio
direct=1
bs=1m
size=512m
filename=/dev/nvme0n1p6
numjobs=1
iodepth=32
group_reporting=1

[write-test]
rw=write
name=write-performance

# -------------------- 第一组 --------------------

write-performance: (g=0): rw=write, bs=(R) 1024KiB-1024KiB, (W) 1024KiB-1024KiB, (T) 1024KiB-1024KiB, ioengine=libaio, iodepth=32
fio-3.12
Starting 1 process

write-performance: (groupid=0, jobs=1): err= 0: pid=4571: Wed Nov 19 10:08:21 2025
  write: IOPS=2892, BW=2893MiB/s (3033MB/s)(512MiB/177msec); 0 zone resets
    slat (usec): min=14, max=139, avg=61.37, stdev=21.39
    clat (usec): min=2804, max=20131, avg=10870.66, stdev=2312.79
     lat (usec): min=2864, max=20171, avg=10932.12, stdev=2312.35
    clat percentiles (usec):
     |  1.00th=[ 4359],  5.00th=[ 9765], 10.00th=[10290], 20.00th=[10290],
     | 30.00th=[10290], 40.00th=[10290], 50.00th=[10421], 60.00th=[10421],
     | 70.00th=[10421], 80.00th=[10421], 90.00th=[13960], 95.00th=[16909],
     | 99.00th=[18482], 99.50th=[19530], 99.90th=[20055], 99.95th=[20055],
     | 99.99th=[20055]
  lat (msec)   : 4=0.78%, 10=4.49%, 20=94.53%, 50=0.20%
  cpu          : usr=14.20%, sys=3.41%, ctx=512, majf=0, minf=20
  IO depths    : 1=0.2%, 2=0.4%, 4=0.8%, 8=1.6%, 16=3.1%, 32=93.9%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=99.8%, 8=0.0%, 16=0.0%, 32=0.2%, 64=0.0%, >=64=0.0%
     issued rwts: total=0,512,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=32

Run status group 0 (all jobs):
  WRITE: bw=2893MiB/s (3033MB/s), 2893MiB/s-2893MiB/s (3033MB/s-3033MB/s), io=512MiB (537MB), run=177-177msec

Disk stats (read/write):
  nvme0n1: ios=28/756, merge=0/0, ticks=6/7988, in_queue=7994, util=48.39%

# -------------------- 第二组 --------------------

write-performance: (g=0): rw=write, bs=(R) 1024KiB-1024KiB, (W) 1024KiB-1024KiB, (T) 1024KiB-1024KiB, ioengine=libaio, iodepth=32
fio-3.12
Starting 1 process

write-performance: (groupid=0, jobs=1): err= 0: pid=4246: Wed Nov 19 10:10:12 2025
  write: IOPS=2497, BW=2498MiB/s (2619MB/s)(512MiB/205msec); 0 zone resets
    slat (usec): min=21, max=181, avg=61.06, stdev=21.37
    clat (usec): min=2819, max=23683, avg=12684.02, stdev=2588.52
     lat (usec): min=2875, max=23760, avg=12745.16, stdev=2591.66
    clat percentiles (usec):
     |  1.00th=[ 4555],  5.00th=[11338], 10.00th=[12125], 20.00th=[12125],
     | 30.00th=[12125], 40.00th=[12125], 50.00th=[12256], 60.00th=[12256],
     | 70.00th=[12256], 80.00th=[12256], 90.00th=[16450], 95.00th=[18482],
     | 99.00th=[21890], 99.50th=[22938], 99.90th=[23725], 99.95th=[23725],
     | 99.99th=[23725]
  lat (msec)   : 4=0.78%, 10=3.52%, 20=93.75%, 50=1.95%
  cpu          : usr=12.68%, sys=4.39%, ctx=512, majf=0, minf=21
  IO depths    : 1=0.2%, 2=0.4%, 4=0.8%, 8=1.6%, 16=3.1%, 32=93.9%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=99.8%, 8=0.0%, 16=0.0%, 32=0.2%, 64=0.0%, >=64=0.0%
     issued rwts: total=0,512,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=32

Run status group 0 (all jobs):
  WRITE: bw=2498MiB/s (2619MB/s), 2498MiB/s-2498MiB/s (2619MB/s-2619MB/s), io=512MiB (537MB), run=205-205msec

Disk stats (read/write):
  nvme0n1: ios=28/649, merge=0/0, ticks=5/7932, in_queue=7938, util=44.80%


# -------------------- 第三组 --------------------

write-performance: (g=0): rw=write, bs=(R) 1024KiB-1024KiB, (W) 1024KiB-1024KiB, (T) 1024KiB-1024KiB, ioengine=libaio, iodepth=32
fio-3.12
Starting 1 process

write-performance: (groupid=0, jobs=1): err= 0: pid=4291: Wed Nov 19 10:11:19 2025
  write: IOPS=2473, BW=2473MiB/s (2594MB/s)(512MiB/207msec); 0 zone resets
    slat (nsec): min=13800, max=93080, avg=55572.46, stdev=18497.39
    clat (usec): min=2826, max=23654, avg=12771.26, stdev=2596.04
     lat (usec): min=2884, max=23724, avg=12826.93, stdev=2596.42
    clat percentiles (usec):
     |  1.00th=[ 4621],  5.00th=[11207], 10.00th=[12125], 20.00th=[12125],
     | 30.00th=[12125], 40.00th=[12125], 50.00th=[12256], 60.00th=[12256],
     | 70.00th=[12256], 80.00th=[12256], 90.00th=[16450], 95.00th=[18744],
     | 99.00th=[21890], 99.50th=[22938], 99.90th=[23725], 99.95th=[23725],
     | 99.99th=[23725]
  lat (msec)   : 4=0.78%, 10=3.52%, 20=93.75%, 50=1.95%
  cpu          : usr=15.05%, sys=0.00%, ctx=512, majf=0, minf=21
  IO depths    : 1=0.2%, 2=0.4%, 4=0.8%, 8=1.6%, 16=3.1%, 32=93.9%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=99.8%, 8=0.0%, 16=0.0%, 32=0.2%, 64=0.0%, >=64=0.0%
     issued rwts: total=0,512,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=32

Run status group 0 (all jobs):
  WRITE: bw=2473MiB/s (2594MB/s), 2473MiB/s-2473MiB/s (2594MB/s-2594MB/s), io=512MiB (537MB), run=207-207msec

Disk stats (read/write):
  nvme0n1: ios=28/639, merge=0/0, ticks=5/7961, in_queue=7967, util=48.39%

# -------------------- 第四组 --------------------

write-performance: (g=0): rw=write, bs=(R) 1024KiB-1024KiB, (W) 1024KiB-1024KiB, (T) 1024KiB-1024KiB, ioengine=libaio, iodepth=32
fio-3.12
Starting 1 process

write-performance: (groupid=0, jobs=1): err= 0: pid=4369: Wed Nov 19 10:14:03 2025
  write: IOPS=2534, BW=2535MiB/s (2658MB/s)(512MiB/202msec); 0 zone resets
    slat (usec): min=15, max=127, avg=57.26, stdev=19.07
    clat (usec): min=2782, max=23054, avg=12461.12, stdev=2543.53
     lat (usec): min=2841, max=23124, avg=12518.48, stdev=2546.60
    clat percentiles (usec):
     |  1.00th=[ 4555],  5.00th=[10814], 10.00th=[11863], 20.00th=[11863],
     | 30.00th=[11863], 40.00th=[11863], 50.00th=[11863], 60.00th=[11863],
     | 70.00th=[11863], 80.00th=[11994], 90.00th=[16057], 95.00th=[18220],
     | 99.00th=[21365], 99.50th=[22414], 99.90th=[22938], 99.95th=[22938],
     | 99.99th=[22938]
  lat (msec)   : 4=0.78%, 10=3.71%, 20=93.75%, 50=1.76%
  cpu          : usr=10.95%, sys=3.98%, ctx=512, majf=0, minf=21
  IO depths    : 1=0.2%, 2=0.4%, 4=0.8%, 8=1.6%, 16=3.1%, 32=93.9%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=99.8%, 8=0.0%, 16=0.0%, 32=0.2%, 64=0.0%, >=64=0.0%
     issued rwts: total=0,512,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=32

Run status group 0 (all jobs):
  WRITE: bw=2535MiB/s (2658MB/s), 2535MiB/s-2535MiB/s (2658MB/s-2658MB/s), io=512MiB (537MB), run=202-202msec

Disk stats (read/write):
  nvme0n1: ios=28/689, merge=0/0, ticks=6/7953, in_queue=7958, util=45.16%

# -------------------- 第五组 --------------------

write-performance: (g=0): rw=write, bs=(R) 1024KiB-1024KiB, (W) 1024KiB-1024KiB, (T) 1024KiB-1024KiB, ioengine=libaio, iodepth=32
fio-3.12
Starting 1 process

write-performance: (groupid=0, jobs=1): err= 0: pid=4504: Wed Nov 19 10:18:16 2025
  write: IOPS=2994, BW=2994MiB/s (3140MB/s)(512MiB/171msec); 0 zone resets
    slat (usec): min=14, max=108, avg=56.90, stdev=19.20
    clat (usec): min=2801, max=20114, avg=10520.72, stdev=1752.69
     lat (usec): min=2866, max=20151, avg=10577.71, stdev=1753.88
    clat percentiles (usec):
     |  1.00th=[ 4293],  5.00th=[ 9765], 10.00th=[10290], 20.00th=[10290],
     | 30.00th=[10290], 40.00th=[10290], 50.00th=[10290], 60.00th=[10421],
     | 70.00th=[10421], 80.00th=[10421], 90.00th=[11600], 95.00th=[11994],
     | 99.00th=[18482], 99.50th=[19530], 99.90th=[20055], 99.95th=[20055],
     | 99.99th=[20055]
  lat (msec)   : 4=0.98%, 10=4.10%, 20=94.73%, 50=0.20%
  cpu          : usr=13.53%, sys=4.71%, ctx=512, majf=0, minf=21
  IO depths    : 1=0.2%, 2=0.4%, 4=0.8%, 8=1.6%, 16=3.1%, 32=93.9%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=99.8%, 8=0.0%, 16=0.0%, 32=0.2%, 64=0.0%, >=64=0.0%
     issued rwts: total=0,512,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=32

Run status group 0 (all jobs):
  WRITE: bw=2994MiB/s (3140MB/s), 2994MiB/s-2994MiB/s (3140MB/s-3140MB/s), io=512MiB (537MB), run=171-171msec

Disk stats (read/write):
  nvme0n1: ios=33/777, merge=0/0, ticks=8/7845, in_queue=7853, util=43.55%

```

2. 文件系统测试结果

``` bash

```

##### 2) 文件系统测试结果

2. 文件系统iozone测试

``` bash

```

##### 3) 有缓存dd策划

```bash

```



##### 4) 有缓存fio测试 

###### 8G组

```bash

```

###### 16G组

```bash

```



#### 六、原因猜想

##### 1. 磁盘xts加密

xts加密导致硬盘io从并行变成串行 导致速度变慢
验证方法

~~~bash
 sudo cryptsetup status /dev/mapper/sm4_disk 
 
 /dev/mapper/sm4_disk is active and is in use.
  type:    LUKS2
  cipher:  sm4-xts-plain64
  keysize: 256 bits
  key location: keyring
  device:  /dev/nvme0n1p6
  sector size:  512
  offset:  32768 sectors
  size:    169093775 sectors
  mode:    read/write


sudo cryptsetup status /dev/mapper/aes_disk
/dev/mapper/aes_disk is active and is in use.
  type:    LUKS2
  cipher:  aes-xts-plain64
  keysize: 512 bits
  key location: keyring
  device:  /dev/nvme0n1p5
  sector size:  512
  offset:  32768 sectors
  size:    314540032 sectors
  mode:    read/write

~~~

dirty写回的时候将bio下发到



##### 2. 加密性能

aes有硬件加密加速 然后sm4是软件实现加密

##### 3. 两个加密的性能开销的地方

| 类别         | SM4加密 | AES加密 | 普通磁盘 |
| ------------ | ------- | ------- | -------- |
| SIMD框架开销 | 43.55%  | 4.83%   | 0%       |
| 加密计算     | 12.71%  | 35.73%  | 0%       |
| 数据拷贝     | 0%      | 8.88%   | 33.11%   |
| 文件系统     | ~10%    | ~10%    | ~15%     |



数据部分

```
19.36%  [kernel]                                       [k] fpsimd_flush_cpu_state                      ◆
  13.37%  [kernel]                                       [k] kernel_neon_begin                           ▒
  11.09%  [kernel]                                       [k] fpsimd_save                                 ▒
  10.28%  [kernel]                                       [k] sm4_ce_encrypt                              ▒
   9.47%  [kernel]                                       [k] arch_cpu_idle                               ▒
   7.81%  [kernel]                                       [k] pre_crypt                                   ▒
   3.09%  [kernel]                                       [k] __local_bh_enable_ip                        ▒
   2.43%  [kernel]                                       [k] sm4_ce_do_crypt                             ▒
   2.31%  [kernel]                                       [k] crypt_convert                               ▒
   2.19%  [kernel]                                       [k] post_crypt                                  ▒
   1.73%  [kernel]                                       [k] kernel_neon_end                             ▒
   1.49%  [kernel]                                       [k] skcipher_walk_done                          ▒
   1.38%  [kernel]                                       [k] kfree                                       ▒
   1.19%  [kernel]                                       [k] crypto_ecb_crypt                            ▒
   1.10%  [kernel]                                       [k] __kmalloc                                   ▒
   1.05%  [kernel]                                       [k] skcipher_walk_next                          ▒
   1.02%  [kernel]                                       [k] blkcipher_walk_done                         ▒
   0.63%  [kernel]                                       [k] memset                                      ▒
   0.53%  [kernel]                                       [k] skcipher_walk_skcipher                      ▒
   0.52%  [kernel]                                       [k] do_direct_IO                                ▒
   0.44%  [kernel]                                       [k] skcipher_walk_first

这是sm4

25.93%  [kernel]                                       [k] ce_aes_xts_encrypt
  11.84%  [kernel]                                       [k] crypt_convert
   9.80%  [kernel]                                       [k] aes_encrypt_block4x
   5.14%  [kernel]                                       [k] __arch_copy_from_user
   4.83%  [kernel]                                       [k] fpsimd_flush_cpu_state
   3.74%  [kernel]                                       [k] __arch_copy_to_user
   2.08%  [kernel]                                       [k] skcipher_walk_done
   1.60%  [kernel]                                       [k] get_page_from_freelist
   1.44%  [kernel]                                       [k] __test_set_page_writeback
   1.39%  [kernel]                                       [k] find_get_pages_range_tag
   1.34%  [kernel]                                       [k] mark_buffer_dirty
   1.09%  [kernel]                                       [k] arch_cpu_idle
   1.03%  [kernel]                                       [k] memcpy
   1.01%  [kernel]                                       [k] find_get_entry
   0.96%  [kernel]                                       [k] blk_queue_split
   0.89%  [kernel]                                       [k] __radix_tree_lookup
   0.79%  [kernel]                                       [k] mpage_process_page_bufs
   0.75%  [kernel]                                       [k] __softirqentry_text_start
   0.75%  [kernel]                                       [k] unlock_page
   0.66%  [kernel]                                       [k] __block_commit_write.isra.11


这是aes磁盘io的数据 分析一

18.83%  [kernel]                                       [k] __arch_copy_from_user
  14.28%  [kernel]                                       [k] __arch_copy_to_user
   4.05%  [kernel]                                       [k] arch_cpu_idle
   3.00%  [kernel]                                       [k] __test_set_page_writeback
   2.55%  [kernel]                                       [k] free_unref_page_list
   2.44%  [kernel]                                       [k] find_get_entry
   2.01%  [kernel]                                       [k] get_page_from_freelist
   1.71%  [kernel]                                       [k] __block_commit_write.isra.11
   1.67%  [kernel]                                       [k] release_pages
   1.66%  [kernel]                                       [k] mark_buffer_dirty
   1.50%  [kernel]                                       [k] __set_page_dirty
   1.33%  [kernel]                                       [k] generic_file_buffered_read
   1.30%  [kernel]                                       [k] unlock_page
   1.28%  [kernel]                                       [k] delete_from_page_cache_batch
   1.23%  [kernel]                                       [k] iov_iter_fault_in_readable
   1.16%  [kernel]                                       [k] __add_to_pag

这是normal
```

测试数据2 io等待时间

``` 
aes的io等待

test@test:~$ sudo biolatency-bpfcc 
Tracing block device I/O... Hit Ctrl-C to end.
^C
     usecs               : count     distribution
         0 -> 1          : 0        |                                        |
         2 -> 3          : 0        |                                        |
         4 -> 7          : 135      |                                        |
         8 -> 15         : 741      |                                        |
        16 -> 31         : 387      |                                        |
        32 -> 63         : 109      |                                        |
        64 -> 127        : 265      |                                        |
       128 -> 255        : 825      |                                        |
       256 -> 511        : 2391     |**                                      |
       512 -> 1023       : 4875     |****                                    |
      1024 -> 2047       : 9620     |*********                               |
      2048 -> 4095       : 14359    |*************                           |
      4096 -> 8191       : 10156    |*********                               |
      8192 -> 16383      : 10468    |**********                              |
     16384 -> 32767      : 13690    |*************                           |
     32768 -> 65535      : 20370    |*******************                     |
     65536 -> 131071     : 18574    |*****************                       |
    131072 -> 262143     : 38501    |*************************************   |
    262144 -> 524287     : 41373    |****************************************|
    524288 -> 1048575    : 13903    |*************                           |
   1048576 -> 2097151    : 2222     |**                                      |
   2097152 -> 4194303    : 78       |                                        |
   
normal 磁盘io等待时间
     usecs               : count     distribution
         0 -> 1          : 0        |                                        |
         2 -> 3          : 0        |                                        |
         4 -> 7          : 161      |                                        |
         8 -> 15         : 668      |*                                       |
        16 -> 31         : 275      |                                        |
        32 -> 63         : 110      |                                        |
        64 -> 127        : 517      |                                        |
       128 -> 255        : 1696     |**                                      |
       256 -> 511        : 1837     |**                                      |
       512 -> 1023       : 2141     |***                                     |
      1024 -> 2047       : 3597     |*****                                   |
      2048 -> 4095       : 5120     |*******                                 |
      4096 -> 8191       : 5718     |********                                |
      8192 -> 16383      : 7786     |***********                             |
     16384 -> 32767      : 13243    |*******************                     |
     32768 -> 65535      : 25283    |*************************************   |
     65536 -> 131071     : 26638    |****************************************|
    131072 -> 262143     : 16443    |************************                |
    262144 -> 524287     : 23490    |***********************************     |
    524288 -> 1048575    : 12369    |******************                      |
   1048576 -> 2097151    : 2019     |***                                     |


sm4 io 延迟
test@test:~$ sudo biolatency-bpfcc 
Tracing block device I/O... Hit Ctrl-C to end.
^C
     usecs               : count     distribution
         0 -> 1          : 0        |                                        |
         2 -> 3          : 0        |                                        |
         4 -> 7          : 90       |                                        |
         8 -> 15         : 626      |                                        |
        16 -> 31         : 416      |                                        |
        32 -> 63         : 116      |                                        |
        64 -> 127        : 764      |                                        |
       128 -> 255        : 1194     |*                                       |
       256 -> 511        : 1631     |*                                       |
       512 -> 1023       : 1682     |*                                       |
      1024 -> 2047       : 3097     |**                                      |
      2048 -> 4095       : 5971     |*****                                   |
      4096 -> 8191       : 9038     |*******                                 |
      8192 -> 16383      : 14530    |************                            |
     16384 -> 32767      : 25033    |*********************                   |
     32768 -> 65535      : 27967    |************************                |
     65536 -> 131071     : 12717    |***********                             |
    131072 -> 262143     : 42849    |*************************************   |
    262144 -> 524287     : 45701    |****************************************|
    524288 -> 1048575    : 10599    |*********                               |
   1048576 -> 2097151    : 745      |                                        |

```



| 优先级 | 测试项 | 目的 |
| ------ | ------ | ---- |
|        |        |      |

| ⭐ 必测 | dm-crypt 队列参数 | 是否只有一个队列导致串行 |
| ------ | ----------------- | ------------------------ |
|        |                   |                          |

| ⭐ 必测 | dirty writeback bcc 跟踪 | 证明 writeback 是瓶颈 |
| ------ | ------------------------ | --------------------- |
|        |                          |                       |

| ⭐ 必测 | biolatency + ext4slower | 确认加密层前后的延迟差 |
| ------ | ----------------------- | ---------------------- |
|        |                         |                        |

| ⭐ 必测 | CPU 指令支持(AES-NI/SM4) | 确认是否软加密放大代价 |
| ------ | ------------------------ | ---------------------- |
|        |                          |                        |

| 中   | on-CPU profile（perf record） | 验证 CPU 核分布是否均匀 |
| ---- | ----------------------------- | ----------------------- |
|      |                               |                         |

| 中   | /proc/vmstat 的 dirty 行为 | dirty 节流是否提前发生 |
| ---- | -------------------------- | ---------------------- |
|      |                            |                        |

| 低   | 内存泄漏/内存高使用排查 | 你已经基本排除 |
| ---- | ----------------------- | -------------- |
|      |                         |                |