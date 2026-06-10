### QEMU 启动 Linux 内核及 GDB 调试教程

> 基于 WSL2 + Linux 6.12 + Busybox 1.38 + QEMU + GDB
>
> 本教程在 WSL2 (Ubuntu/Debian) 环境下完成，其他 Linux 发行版流程基本一致。

#### 一、WSL 编译环境配置

##### 1. 安装编译工具链和依赖

```bash
sudo apt update
sudo apt install -y build-essential libncurses5-dev libssl-dev \
  bison flex libelf-dev bc qemu-system-x86 gdb
```

各包作用：
- `build-essential`: gcc, make 等基础编译工具
- `libncurses5-dev`: `make menuconfig` 图形化配置需要
- `libssl-dev`, `bison`, `flex`, `libelf-dev`, `bc`: 内核编译依赖
- `qemu-system-x86`: QEMU x86 模拟器
- `gdb`: 调试器

##### 2. 创建工作目录

```bash
mkdir -p ~/linux-kernel && cd ~/linux-kernel
```

#### 二、编译 Linux 内核

##### 1. 下载并解压内核源码

```bash
wget https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.12.90.tar.xz
tar -xvf linux-6.12.90.tar.xz
cd linux-6.12.90
```

> 也可以从 [kernel.org](https://kernel.org/) 下载其他版本。建议选 longterm 版本，稳定且长期维护。

##### 2. 配置内核

```bash
make defconfig
```

`defconfig` 会生成一个默认的 x86_64 配置，对于学习和调试已经足够。

如果需要自定义（一般不需要），可以运行 `make menuconfig` 进行图形化配置。

##### 3. 编译

```bash
make -j$(nproc)
```

`$(nproc)` 会自动使用所有 CPU 核心并行编译，首次编译大约需要 10-20 分钟。

编译完成后，内核镜像在 `arch/x86/boot/bzImage`，带调试符号的未压缩镜像在源码根目录的 `vmlinux`（GDB 调试需要这个文件）。

#### 三、编译 Busybox 构建根文件系统

##### 1. 下载并解压 Busybox

```bash
cd ~/linux-kernel
wget https://busybox.net/downloads/busybox-1.38.0.tar.bz2
tar -xvf busybox-1.38.0.tar.bz2
cd busybox-1.38.0
```

##### 2. 配置静态编译

```bash
make defconfig
sed -i 's/# CONFIG_STATIC is not set/CONFIG_STATIC=y/' .config
```

必须开启静态链接，否则在 QEMU 的最小文件系统里会找不到动态库。

也可以用 `make menuconfig` 手动勾选：`Settings -> Build static binary`。

##### 3. 编译安装

```bash
make -j$(nproc)
make CONFIG_PREFIX=~/linux-kernel/rootfs install
```

##### 4. 完善根文件系统

```bash
cd ~/linux-kernel/rootfs

# 创建必要的目录
mkdir -p proc sys dev etc etc/init.d tmp mnt root

# fstab：挂载虚拟文件系统
cat > etc/fstab << 'EOF'
proc    /proc   proc    defaults    0 0
sysfs   /sys    sysfs   defaults    0 0
tmpfs   /tmp    tmpfs   defaults    0 0
devtmpfs /dev   devtmpfs defaults   0 0
EOF

# init 脚本
cat > etc/init.d/rcS << 'EOF'
#!/bin/sh
echo "Welcome to Linux (QEMU)"
mount -o remount,rw /
mount -a
EOF
chmod +x etc/init.d/rcS

# inittab
cat > etc/inittab << 'EOF'
::sysinit:/etc/init.d/rcS
::respawn:-/bin/sh
::askfirst:-/bin/sh
::ctrlaltdel:/bin/umount -a -r
EOF
chmod +x etc/inittab

# 创建基本设备节点
sudo mknod -m 622 dev/console c 5 1
sudo mknod -m 666 dev/null c 1 3
sudo mknod -m 666 dev/tty c 5 0
sudo mknod -m 444 dev/random c 1 8
sudo mknod -m 444 dev/urandom c 1 9
```

##### 5. 制作 initramfs 镜像

```bash
cd ~/linux-kernel/rootfs
find . | cpio -o -H newc | gzip > ../initramfs.cpio.gz
```

> 这种方式使用 initramfs（cpio 格式），比旧的 ext4 磁盘镜像方式更简单，也是内核社区推荐的做法。

#### 四、启动内核

##### 1. 准备启动目录

```bash
mkdir -p ~/linux-kernel/boot && cd ~/linux-kernel/boot

# 复制内核镜像和 initramfs
cp ~/linux-kernel/linux-6.12.90/arch/x86/boot/bzImage .
cp ~/linux-kernel/initramfs.cpio.gz .
```

##### 2. 创建启动脚本

```bash
cat > ~/linux-kernel/boot/run.sh << 'EOF'
#!/bin/bash
qemu-system-x86_64 \
  -kernel bzImage \
  -initrd initramfs.cpio.gz \
  -append "console=ttyS0" \
  -nographic \
  -m 512M
EOF
chmod +x ~/linux-kernel/boot/run.sh
```

> `-nographic` 把所有输出重定向到当前终端，不需要图形界面。
> `-m 512M` 给虚拟机分配 512MB 内存。

##### 3. 启动

```bash
cd ~/linux-kernel/boot
./run.sh
```

看到 `Welcome to Linux (QEMU)` 和 shell 提示符就说明成功了。按 `Ctrl+A` 再按 `X` 可以退出 QEMU。

#### 五、使用 GDB 调试内核

##### 1. 准备调试文件

```bash
# vmlinux 是带调试符号的未压缩内核镜像
cp ~/linux-kernel/linux-6.12.90/vmlinux ~/linux-kernel/boot/
```

##### 2. 以调试模式启动 QEMU

```bash
cat > ~/linux-kernel/boot/debug.sh << 'EOF'
#!/bin/bash
qemu-system-x86_64 \
  -kernel bzImage \
  -initrd initramfs.cpio.gz \
  -append "console=ttyS0 nokaslr" \
  -nographic \
  -m 512M \
  -S -s
EOF
chmod +x ~/linux-kernel/boot/debug.sh
```

参数说明：
- `-S`: QEMU 启动后暂停 CPU，等待 GDB 连接
- `-s`: 开放 1234 端口供 GDB 连接（等价于 `-gdb tcp::1234`）
- `nokaslr`: 关闭内核地址随机化，方便设断点

##### 3. 启动调试

终端1：启动 QEMU（会停住等待连接）

```bash
cd ~/linux-kernel/boot
./debug.sh
```

终端2：启动 GDB

```bash
cd ~/linux-kernel/boot
gdb vmlinux
```

在 GDB 中连接并开始调试：

```
(gdb) target remote :1234    # 连接 QEMU
(gdb) break start_kernel     # 在内核入口打断点
(gdb) c                      # 继续执行，会停在 start_kernel
(gdb) n                      # 单步执行
(gdb) print some_variable    # 查看变量
(gdb) bt                     # 查看调用栈
```

##### 4. 常用 GDB 调试命令

| 命令 | 说明 |
|---|---|
| `break start_kernel` | 在函数入口打断点 |
| `break init/main.c:100` | 在指定文件行号打断点 |
| `c` | 继续执行 |
| `n` | 单步执行（不进入函数） |
| `s` | 单步执行（进入函数） |
| `bt` | 查看调用栈 |
| `print var` | 打印变量值 |
| `list` | 查看当前代码 |
| `info breakpoints` | 查看所有断点 |
| `delete 1` | 删除 1 号断点 |
| `quit` | 退出 GDB |

#### 六、文件结构总览

```
~/linux-kernel/
├── linux-6.12.90/          # 内核源码
├── busybox-1.38.0/         # Busybox 源码
├── rootfs/                 # 根文件系统目录
├── initramfs.cpio.gz       # 打包好的 initramfs 镜像
└── boot/
    ├── bzImage             # 压缩内核镜像
    ├── vmlinux             # 未压缩内核镜像（GDB 调试用）
    ├── initramfs.cpio.gz   # initramfs 镜像
    ├── run.sh              # 正常启动脚本
    └── debug.sh            # 调试启动脚本
```

#### 七、常见问题

**Q: 编译内核报错 `No rule to make target 'debian/certs/debian-uefi-certs.pem'`**
WSL 跑的是 Ubuntu/Debian 发行版内核配置时会引用这个证书。运行：
```bash
scripts/config --disable SYSTEM_TRUSTED_KEYS
scripts/config --disable SYSTEM_REVOCATION_KEYS
```
然后重新 `make -j$(nproc)`。

**Q: QEMU 启动后黑屏没有任何输出**
确认启动参数里有 `console=ttyS0`，并且使用了 `-nographic`。

**Q: GDB 连接不上 QEMU**
确认 QEMU 用了 `-S -s` 参数，并且在另一个终端启动 GDB 时执行 `target remote :1234`。

**Q: Busybox 编译报错**
确认已经开启了静态编译（`CONFIG_STATIC=y`），并且安装了 `build-essential`。
