# wsl介绍以及安装指南

## wsl 是什么
wsl 全称 (windows subsystem for linux) 翻译或来就是 针对windows的linux子系统，这里介绍一下linux是什么

- linux
linux 是除了window mac之外的第三个操作系统，基本上全世界的服务器都是用的linux系统，那么linux系统有什么好处
1. 配环境方便，如果要在windows系统安装c语言编译环境，那么需要去网上下载gcc或者  ，非常麻烦，但是在linux下配c语言环境只需要一行命令
```bash
sudo apt install gcc g++
# 或者
sudo dnf install gcc g++
```
然后你就可以编译c语言程序了，非常方便，并且现在如果想使用claude，如果在linux环境下，claude可以帮你一键配置环境，包括pyhon等，claude可以帮你编辑word文档，做ppt，但是需要很多工具，如果你在windows环境下，下载这些工具会很麻烦，但是如果你在linux下，将可以一件配置，关于claude的下载，将在另一个教程里面介绍
2. 嵌入式和后端很多地方都会用到wsl，包括前端的地方，部署一般都在linux下，所以早点接触学习linux是好事

说回来wsl是什么，一般一个电脑只能安装一个操作系统，要么安装win mac linux只能选择一个，但是wsl给了一个在windows下面用linux系统的方法，这个是微软自带的，如果想知道wsl是怎么实现的，可以自行去微软官方文档了解

## wsl 安装指南

1. 进bios开启虚拟化
这一步可以先不做，如果后面出错了，然后再回来检查这个，每个主板的bios不同，搜自己型号的电脑的bios开启虚拟化的方法

2. 打开powershell
运行
```bash
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```
这一步是启用windows的虚拟化功能

也可以通过图形化的方式打开
在win的搜索窗口里面搜索 "打开或关闭Windows功能"
在里面找到 Windows虚拟机监控程序平台，打开这选项，然后确定，会让你重启

3. 重启完成后再次打开powershell
运行 
```bash
wsl --list --online
```

会发现有很多可以安装的，这里需要解释一下，linux是一个操作系统内核，不是完整的操作系统，但是很多厂商会使用这个内核做一个完整的操作系统

在这里推荐安装 debian 或者 Ubuntu 或者 fedora，如果你有其他更喜欢的可以自行安装自己喜欢的发行版

安装
```bash
wsl --install Debian
```

然后会提示你输入用户名 你随便输一个就好，然后会让你输入一个密码，这个密码不会回显，直接输入完成回车就行

到此wsl安装完成

以后想要进去wsl 只需要打开powershell输入wsl就行 如果你下了多个wsl 可以使用 wsl --list 看看自己安装了什么，然后使用wsl -d <distro> 进入你需要的发行版

## 使用wsl
wsl实际上就是一个linux，所以使用wsl本身需要会使用linux，我这里给出一些命令做参考

```bash
ls
cd
cd ~
pwd
mkdir
rmdir
touch 
vim 
cat
tar
tee
grep
awk

# debian ubuntu的包管理器
apt
# 更新系统
sudo apt update 
sudo apt upgrade

# fedora centos的包管理器
dnf yum
sudo dnf update
sudo yum update
```



