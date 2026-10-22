# 安装 Pi：macOS、Linux、Windows

Pi 是终端程序。安装本身不复杂，真正需要选的是：**原生 Windows 还是 WSL**，以及**用官方安装器还是 npm**。

## 前置条件

- 一个可用的模型来源：订阅、API Key 或本地模型；
- macOS / Linux / Windows 10+；
- Windows 原生使用 bash 工具时，需要 Git for Windows；
- 用 npm 安装时，需要 **Node.js 22.19 或更高版本**。

## macOS / Linux

### 官方安装器（推荐）

```bash
curl -fsSL https://pi.dev/install.sh | sh
```

安装器会固定依赖版本，后续用：

```bash
pi update
```

更新 Pi 本身。

### npm

```bash
npm install -g --ignore-scripts @earendil-works/pi-coding-agent
```

要求 Node.js 22.19+。`--ignore-scripts` 是官方建议，Pi 正常安装不依赖依赖项的生命周期脚本。

### 验证

```bash
pi --version
```

能输出版本号就说明安装成功。如果提示 `pi: command not found`，检查 npm 全局 bin 目录是否在 `PATH` 中，或重开终端。

## Windows

Windows 有三条路：

1. **原生 Windows + PowerShell 安装器**（最简单）；
2. **原生 Windows + npm**；
3. **WSL**（文件/工具链在 Linux 时更顺）。

### 方式一：PowerShell 安装器

在 PowerShell 中：

```powershell
powershell -c "irm https://pi.dev/install.ps1 | iex"
```

或直接：

```powershell
irm https://pi.dev/install.ps1 | iex
```

安装完成后重开终端：

```powershell
pi --version
```

!!! warning "原生 Windows 需要 Git Bash"
    Pi 的默认 `bash` 工具需要 Bash。原生 Windows 请安装 [Git for Windows](https://git-scm.com/download/win)，并确保 Git Bash 可用。

如果 Pi 找不到 Bash，会报告它检查过的路径。可以：

- 安装 Git for Windows；
- 把其他 Bash 放到 `PATH`；
- 或在设置里配置 `shellPath`。

### 方式二：npm

```powershell
npm install -g --ignore-scripts @earendil-works/pi-coding-agent
```

同样要求 Node.js 22.19+。

### 方式三：WSL

在 WSL 里按 Linux 方式安装：

```bash
curl -fsSL https://pi.dev/install.sh | sh
```

WSL 的好处是 Linux 工具链完整；缺点是跨文件系统访问 Windows 文件可能变慢。项目放在 WSL 文件系统里通常更顺。

### 可选：让模型使用 PowerShell

原生 Windows 默认用 Git Bash。若任务依赖 PowerShell 模块，可以在 `~/.pi/agent/settings.json` 里把默认工具换成：

```json
{
  "defaultTools": ["read", "powershell", "edit", "write"]
}
```

或：

```json
{
  "defaultTools": ["-bash", "+powershell"]
}
```

`!` 和 `!!` 仍然使用 Bash。

## 首次启动

进入一个项目目录：

```bash
cd /path/to/project
pi
```

首次启动可能询问是否信任当前目录。**只在你信任的仓库里选信任**。项目 trust 会允许加载 `.pi` 下的项目扩展、skills、prompts、MCP 等资源。

然后：

```text
/login
```

选择 provider 并完成登录。

```text
/model
```

选择模型。也可以启动时指定：

```bash
pi --model <model>
```

最后给它一个任务：

```text
解释这个仓库的结构，不要改文件
```

## 更新

```bash
pi update              # 更新 Pi
pi update --models     # 刷新模型目录
pi update --extensions # 更新已安装的 Pi 包
pi update --all        # 更新 Pi + 包
```

如果 Pi 由 Nix 等包管理器提供，`pi update` 不能更新自身；用对应包管理器升级。

## 卸载

### npm

```bash
npm uninstall -g @earendil-works/pi-coding-agent
```

### 安装器

重新运行安装器，选择 **Uninstall Pi**：

```bash
curl -fsSL https://pi.dev/install.sh | sh
```

### Nix

```bash
nix profile remove pi
```

以上都不会删除 `~/.pi/agent/` 里的配置、凭据、会话和已安装包。需要彻底清理时再手动处理。

## 安装后目录

Pi 的用户级目录默认是：

```text
~/.pi/agent/
```

里面会有：

```text
settings.json       用户设置
auth.json           登录凭据
models.json         自定义模型/端点
mcp.json            MCP 服务器
AGENTS.md           全局指令
extensions/         用户扩展
skills/             用户技能
prompts/            提示词模板
themes/             主题
```

项目级配置在项目根目录的 `.pi/`，只有在 trust 后加载。

## 常见安装问题

| 现象 | 处理 |
| --- | --- |
| `pi: command not found` | 检查 npm 全局 bin 是否在 PATH；重开终端 |
| `npm` 报 Node 版本不支持 | 升级到 Node 22.19+ |
| Windows 找不到 bash | 安装 Git for Windows，或配置 `shellPath` |
| PowerShell 执行被阻止 | 检查执行策略；不要在不明来源脚本上绕过 |
| Windows 快捷键异常 | 见官方 Terminal Setup；Windows Terminal 会占用部分 Alt 组合 |
| WSL 里很慢 | 把项目放到 WSL 文件系统，而不是 `/mnt/c` |
| 首次启动一直问信任 | 只对信任目录选择 trust；不要全局关闭 |

## 官方参考

- [Quickstart](https://pi.dev/docs/latest/quickstart)
- [Windows Setup](https://pi.dev/docs/latest/windows)
- [Terminal Setup](https://pi.dev/docs/latest/terminal-setup)
- [CLI](https://pi.dev/docs/latest/cli)
