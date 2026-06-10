### ai 编程工具安装教程（claude code + codex）

> 基于 node.js + npm 安装，支持 wsl / linux / windows 环境

#### 一、安装 node.js 和 npm

claude code 和 codex 都需要 node.js（>=18），先装 node.js。

##### wsl / linux（ubuntu / debian）

```bash
# 用 NodeSource 装 node.js 22.x LTS
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs

# 验证
node -v
npm -v
```

##### wsl / linux（fedora）

```bash
curl -fsSL https://rpm.nodesource.com/setup_22.x | sudo bash -
sudo dnf install -y nodejs

node -v
npm -v
```

##### windows — winget

```powershell
winget install OpenJS.NodeJS.LTS
```

装完重启 PowerShell，验证：

```powershell
node -v
npm -v
```

##### windows — 官网下载

去 [nodejs.org](https://nodejs.org/) 下载 LTS 版本安装包，双击安装。

建议装 LTS 版本。

如果 `npm install -g` 报权限错误，推荐用 nvm 管理 node.js：

linux / wsl：

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
source ~/.bashrc
nvm install --lts
```

windows — 用 nvm-windows：

```powershell
winget install CoreyButler.NVMforWindows
# 重启终端后
nvm install lts
nvm use lts
```

#### 二、安装 claude code

```bash
npm install -g @anthropic-ai/claude-code
```

装完使用：

```bash
cd your-project
claude
```

首次运行会提示登录 claude 账号，先别登录，后面用第三方 api。

#### 三、安装 codex（openai）

```bash
npm install -g @openai/codex
```

装完运行：

```bash
codex
```

首次运行会提示登录 chatgpt 账号。

#### 四、接入第三方 api

##### claude code + deepseek

1. 进入 deepseek api 开放平台：[platform.deepseek.com/usage](https://platform.deepseek.com/usage)

   可以充点钱试试，5 块能用很久。

2. 在左边点 api keys，创建一个 api key，随便取个名字。

3. 安装依赖：

   ```bash
   sudo apt install python3 python3-pip
   pip install anthropic
   ```

4. 启动时设置环境变量：

   ```bash
   export ANTHROPIC_BASE_URL=https://api.deepseek.com/anthropic
   export ANTHROPIC_API_KEY=${YOUR_API_KEY}
   claude
   ```

##### codex + 第三方中转

推荐一个学长的 codex 中转站：[kycode.shop](https://kycode.shop/home)

进去申请一个 api key，复制对应的配置到配置文件里，然后运行：

```bash
codex
```

#### 五、常见问题

**Q: `npm install -g` 报权限错误（linux / wsl）**

不要用 `sudo npm install -g`，用上面提到的 nvm 方式安装 node.js 即可解决。

**Q: windows 下 `node` 命令找不到**

确认安装时勾选了 "Add to PATH"，或重启终端 / 电脑。

**Q: claude code 登录失败**

检查网络。如果在国内可能需要配置代理或使用第三方 api（见第四节）。
