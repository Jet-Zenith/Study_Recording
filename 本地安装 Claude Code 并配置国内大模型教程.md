# 本地安装 Claude Code 并配置国内大模型教程

> 适用系统：Windows | 无需依赖 Node.js/Python | 兼容 CMD/PowerShell 终端

## 一、前置准备

1. 准备一个国内大模型的 API Key（以智谱 AI 为例，当然也支持 DeepSeek、通义千问等）
   - 智谱 AI：登录[智谱开放平台](https://open.bigmodel.cn/) → 控制台 → API Key 管理，创建并复制 Key
2. 确保你的终端能正常访问互联网（国内模型无需代理）

## 二、安装 Claude Code

### 1. 打开 PowerShell 或 CMD

- 按下 `Win + R`，输入 `powershell `或 `cmd` ，回车打开终端 / Mac打开终端或命令提示符

### 2. 执行官方安装命令

Claude Code官方网站：https://claude.com/product/claude-code

```powershell
Windows：irm https://claude.ai/install.ps1 | iex

Mac：curl -fsSL https://claude.ai/install.sh | bash
```

- 命令执行过程中会自动下载并安装 Claude Code，安装完成后会输出版本号和安装路径

- 安装成功示例：

  ```plaintext
  ✓Claude Code successfully installed!
  Version: 2.1.121
  Location: C:\Users\你的用户名\.local\bin\claude.exe
  Next：Run claude --help to get start
  □ Installation complete!
  ```

### 3. 验证安装

在终端中输入以下命令，查看是否正常输出版本信息：

```
claude --version
```

------

## 三、配置国内大模型（以智谱 GLM-4.6 为例）

### 1. 找到 Claude Code 配置文件路径

Claude Code 的配置文件 `settings.json` 位于以下路径：

```plaintext
Windows：C:\Users\你的用户名\.claude\settings.json

Mac：~/.claude/settings.json
```

如果文件不存在，手动创建该文件即可。

### 2. 写入配置文件

用记事本打开 `settings.json`，复制以下内容并修改：

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "https://open.bigmodel.cn/api/anthropic",
    "ANTHROPIC_AUTH_TOKEN": "你的智谱API Key",
    "API_TIMEOUT_MS": "300000"
  },
  "model": "glm-4.6",
  "smallModel": "glm-4-flash"
}
```

#### 配置说明

|          字段          |              作用              |                智谱配置值                |           DeepSeek 配置值            |
| :--------------------: | :----------------------------: | :--------------------------------------: | :----------------------------------: |
|  `ANTHROPIC_BASE_URL`  | 兼容 Anthropic 协议的 API 地址 | `https://open.bigmodel.cn/api/anthropic` | `https://api.deepseek.com/anthropic` |
| `ANTHROPIC_AUTH_TOKEN` |       你的大模型 API Key       |          智谱开放平台获取的 Key          |     DeepSeek 开放平台获取的 Key      |
|        `model`         |   主模型（用于复杂编码任务）   |                `glm-4.6`                 |           `deepseek-chat`            |
|      `smallModel`      |    轻量模型（用于简单任务）    |              `glm-4-flash`               |           `deepseek-coder`           |
|    `API_TIMEOUT_MS`    |      请求超时时间（毫秒）      |            `300000`（5 分钟）            |          `300000`（5 分钟）          |

------

## 四、启动 Claude Code 并使用

### 1. 重新打开终端（CMD 或 PowerShell）

配置文件修改后，需要重启终端才能生效。

### 2. 启动 Claude Code

在终端中输入：

```cmd
claude
```

- 此时 Claude Code 会自动使用你配置的国内大模型，无需登录 Anthropic 官方账号
- 首次启动会进入交互界面，输入 `/help` 可查看所有可用命令

### 3. 基础使用示例

- 在当前目录创建 Python 文件：输入 `/create app.py`，按提示生成代码
- 让模型修改代码：直接输入你的需求，如 “帮我优化这个 app.py 的性能”
- 退出交互界面：输入 `/exit` 或按 `Ctrl + C`

## 五、常见问题与解决

### 1. 提示 `claude 不是内部或外部命令`

解决方法：

1. 将安装路径 `C:\Users\你的用户名\.local\bin` 添加到系统环境变量 `PATH` 中
2. 重启终端，重新执行 `claude --version` 验证

### 2. 模型调用超时 / 失败

解决方法：

- 检查 `ANTHROPIC_BASE_URL` 是否正确（智谱为 `https://open.bigmodel.cn/api/anthropic`）
- 检查 API Key 是否复制完整，没有多余空格
- 增大 `API_TIMEOUT_MS` 配置（如改为 `600000`，即 10 分钟）

### 3.网络环境限制（未配置代理、防火墙拦截）/PowerShell 未走系统代理

#### a.先确认网络连通性

在 PowerShell 中执行以下命令，测试能否访问目标服务器：

```powershell
irm https://downloads.claude.ai/claude-code-releases/latest -UseBasicParsing
```

- 如果返回报错，说明你的网络 / 代理确实无法访问该域名，需要先解决网络问题。
- 如果返回正常的文本（版本信息），则可能是 PowerShell 配置问题，继续往下看。

#### b.给 PowerShell 配置代理（最有效的解决方法）

大多数情况是因为 PowerShell 没有走你电脑的代理。

```powershell
# 先配置代理（替换为你的代理地址和端口，例如本地代理 127.0.0.1:7890）
$env:HTTP_PROXY = "http://127.0.0.1:7890"
$env:HTTPS_PROXY = "http://127.0.0.1:7890"

# 再重新执行安装命令
irm https://claude.ai/install.ps1 | iex
```