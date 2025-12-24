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



