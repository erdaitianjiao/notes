# wsl介绍以及安装指南

## wsl 是什么
wsl 全称 (windows subsystem for linux) 翻译过来就是 针对windows的linux子系统。简单来说，它让你在windows里直接跑linux，不需要装虚拟机或者双系统。

### 为什么要用linux

linux 是一个操作系统内核，基于它有很多发行版（如 Ubuntu、Debian、Fedora），全世界绝大多数服务器都跑在linux上。对开发者来说linux有几个明显的好处：

1. **配环境方便** — 在windows上装gcc要自己找安装包、配环境变量，但在linux下只需要一行命令：
```bash
sudo apt install gcc g++
# 或者 (fedora)
sudo dnf install gcc g++
```
2. **工具链丰富** — 很多开发工具在linux下安装更简单。比如 Claude Code 在linux下可以直接辅助你安装和配置开发工具（python、gcc等），在windows下则需要额外折腾。Claude Code 的安装会在另一个教程里介绍。
3. **行业需要** — 嵌入式、后端、前端的部署基本都在linux下进行，早点接触没坏处。

### WSL1 和 WSL2 的区别

| | WSL1 | WSL2 |
|---|---|---|
| 架构 | 转译层，没有真正的linux内核 | 真正的linux内核跑在虚拟机里 |
| 文件性能 | 访问windows文件快 | linux内部文件操作快很多 |
| 兼容性 | 部分软件跑不了 | 接近完整linux兼容性 |
| 网络和USB | 不支持 | 支持更好 |

**推荐装WSL2**，下面的步骤也是按WSL2来的。

## WSL2 安装指南

### 步骤1：确认虚拟化已开启

进 BIOS 确认虚拟化（Intel VT-x / AMD-V）已开启。不同主板操作不同，搜自己电脑型号 + "bios开启虚拟化" 就能找到教程。如果后面安装报错再回来检查这步也行。

### 步骤2：启用WSL和虚拟机平台

**方式一：命令行（管理员权限打开 PowerShell）**

```powershell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

**方式二：图形化**

在 Windows 搜索栏搜索"打开或关闭Windows功能"，勾选以下两项：
- 适用于 Linux 的 Windows 子系统
- 虚拟机平台

确定后会提示重启。

### 步骤3：重启电脑

### 步骤4：安装发行版

重启后打开 PowerShell，先看看有哪些可以装：

```powershell
wsl --list --online
```

推荐安装 **Ubuntu**（对新手最友好，教程最多）或 **Debian**：

```powershell
wsl --install Ubuntu
```

> 如果你的 Windows 10 版本较新或者用的是 Windows 11，也可以直接运行 `wsl --install`，它会自动装好 WSL2 + Ubuntu。

安装过程中会提示你设置用户名和密码。密码输入时不会显示字符，这是正常的，输完回车就行。

### 步骤5：设置默认版本为WSL2

```powershell
wsl --set-default-version 2
```

到此安装完成。

### 日常使用

- 打开 PowerShell 输入 `wsl` 即可进入
- 如果装了多个发行版，用 `wsl --list` 查看，用 `wsl -d <发行版名>` 切换
- 推荐使用 Windows Terminal，可以在里面直接打开多个标签页（PowerShell + WSL）

### 注意事项

- WSL 可以直接访问 Windows 文件，路径在 `/mnt/` 下面：
  - `/mnt/c/` 对应 C 盘
  - `/mnt/d/` 对应 D 盘
  - 以此类推
- **绝对不要在 WSL 里执行 `rm -rf /`**，因为 `/mnt/` 挂载了 Windows 的磁盘，这会把 Windows 上的文件也删掉

## 常用命令速查

wsl 本质上就是一个 linux 环境，以下是一些常用命令：

| 命令 | 说明 | 示例 |
|---|---|---|
| `ls` | 列出当前目录的文件 | `ls -la` |
| `cd` | 切换目录 | `cd ~/projects` |
| `pwd` | 显示当前路径 | |
| `mkdir` | 创建目录 | `mkdir my-project` |
| `rm` | 删除文件/目录 | `rm file.txt` / `rm -r dir/` |
| `cp` | 复制 | `cp a.txt b.txt` |
| `mv` | 移动/重命名 | `mv old.txt new.txt` |
| `cat` | 查看文件内容 | `cat readme.md` |
| `grep` | 搜索文本 | `grep "error" log.txt` |
| `vim` / `nano` | 编辑文件 | `nano main.c` |
| `tar` | 解压/压缩 | `tar -xzf archive.tar.gz` |

### 包管理

```bash
# Ubuntu / Debian
sudo apt update          # 更新软件源列表
sudo apt upgrade         # 升级已安装的包
sudo apt install <包名>   # 安装软件

# Fedora
sudo dnf update
sudo dnf install <包名>
```

更详细的 linux 命令学习推荐看 [Linux命令大全](https://www.runoob.com/linux/linux-command-manual.html)

## 常见问题

**Q: 报错"WSL2 requires an update to its kernel component"**
运行 `wsl --update` 更新内核即可。

**Q: 报错虚拟化相关错误**
回到步骤1检查 BIOS 虚拟化是否开启。

**Q: 网络连接不上**
WSL2 的网络是 NAT 模式，偶尔会有 DNS 问题，尝试 `echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf`。



