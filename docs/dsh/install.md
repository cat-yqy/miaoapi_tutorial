# 安装 DSH：Desktop、CLI 与 TUI

DeepSeek Harness 有三条主要安装路线：

1. **Desktop**：官方图形安装包，自带运行时，不需要 Node；
2. **CLI**：npm 包 `@deepseek-ai/dsh`，提供 Web、Headless、ACP、SDK；
3. **TUI**：社区插件 `@deepseek-harness-tui/dsh-tui`，需要先有 CLI。

先选路线，不必全装。Desktop 和 CLI 共享 `$DSH_HOME` 下的会话、设置和凭据，但 profile 与插件激活彼此隔离。

## 平台支持

| 平台 | Desktop | CLI / Web | TUI |
| --- | --- | --- | --- |
| **macOS Apple Silicon** | 官方 DMG，macOS 13+ | 支持 | 支持 |
| **macOS Intel** | 官方暂未提供 | 支持 | 支持 |
| **Windows x64** | 官方 EXE | 支持 | 支持 |
| **Linux x64/arm64** | 官方暂未提供桌面包 | 支持 | 支持 |

!!! note "Linux Desktop"
    截至本文，官方下载页只提供 macOS Apple Silicon 和 Windows x64 桌面安装包。Linux 用户请使用 Web UI 或 CLI/TUI；社区第三方桌面壳不属于官方发布，使用前需自行审查。

## 方式一：Desktop

### macOS（Apple Silicon）

官方下载：

<https://download.deepseek.com/desktop/dsh-latest-macos-arm64.dmg>

要求：

- Apple Silicon（M 系列）；
- macOS 13 或更高；
- 官方目前不提供 Intel Mac 的 DMG。

安装：

1. 打开 DMG；
2. 把 **DeepSeek Harness** 拖进“应用程序”；
3. 从“应用程序”启动；
4. 首次启动按欢迎页提示登录或配置 API Key。

如果 macOS 提示无法验证开发者：

- 确认安装包来自上面的官方域名；
- 在“系统设置 → 隐私与安全性”里按系统提示允许；
- 不要对来源不明的包全局关闭 Gatekeeper。

### Windows x64

官方下载：

<https://download.deepseek.com/desktop/dsh-latest-windows-x64.exe>

安装：

1. 运行 EXE；
2. 选择安装目录（安装器只面向当前用户）；
3. 完成页可直接启动；
4. 首次启动按欢迎页提示登录或配置 API Key。

如果 SmartScreen 提示：

- 先确认文件来自 `download.deepseek.com`；
- 查看文件属性/数字签名；
- 不要对来源不明的安装包选择“仍要运行”。

### Desktop 自带 CLI

Desktop 安装后可以把 `dsh` 命令注册到系统：

1. 菜单 **管理 dsh 命令…**（在“检查更新…”下方）；
2. 选择安装/修复；
3. 打开新终端：
   ```bash
   dsh --version
   ```

行为：

- macOS 创建 `/usr/local/bin/dsh`，可能需要管理员授权；
- Windows 写入当前用户 PATH；
- 使用 Desktop 内置运行时，不要求单独安装 Node；
- 卸载 Desktop 前，先在同一个菜单里“移除”命令。

## 方式二：CLI（npm）

### 前置条件

- **Node.js 24.9 或更高版本**（`@deepseek-ai/dsh` 的 `engines` 要求）；
- npm；
- 能访问 npm registry，或配置好代理/镜像。

检查：

```bash
node --version
npm --version
```

### 先试用，不安装

```bash
npx @deepseek-ai/dsh web
```

这是官方 README 的推荐启动方式：不全局安装，直接运行 Web UI。默认在 `http://127.0.0.1:3080` 启动并打开浏览器；SSH 环境只打印 URL。

### 全局安装

```bash
npm install -g @deepseek-ai/dsh
dsh --version
```

macOS/Linux 如果遇到全局目录权限问题，优先配置 npm prefix 或使用 `npx`，不要习惯性 `sudo npm install -g`。

### Windows

PowerShell：

```powershell
npm install -g @deepseek-ai/dsh
dsh --version
```

如果 `dsh` 不在 PATH，检查 npm 全局 bin 目录，或重开终端。

### 从源码运行

```bash
git clone https://github.com/deepseek-ai/deepseek-harness.git
cd deepseek-harness
pnpm install
pnpm run build
pnpm dsh web
```

`pnpm run build` 准备产物；`pnpm dsh web` 使用已构建产物，不会自动重建。

## 方式三：TUI（社区插件）

!!! warning "社区组件"
    `@deepseek-harness-tui/dsh-tui` 是社区维护的 TUI，不属于 DeepSeek 官方包。它质量不错、更新频繁，但安装前仍应查看源码和发布记录。官方 CLI 本身提供 Web/Headless/ACP/SDK，不内置 TUI。

### 安装

需要 Node 24.9+、pnpm 和官方 CLI：

```bash
npm install -g pnpm
npm install -g @deepseek-ai/dsh @deepseek-harness-tui/dsh-tui
```

启动：

```bash
dsh-tui
# 或短别名
dst
```

首次启动会自动初始化 `dsh-tui` profile，需要 pnpm。

### 手动挂到 profile

如果不想全局安装 TUI：

```bash
dsh plugin --profile dsh-tui add @deepseek-harness-tui/dsh-tui
dsh --profile dsh-tui
```

### 常用命令

```bash
dsh-tui --resume        # 恢复会话
dsh-tui update          # 更新 TUI
dsh-tui doctor          # 环境检查
dsh-tui safe            # 只读诊断/修复
dsh-tui version
```

TUI 自带自动化 CLI：

```bash
dst -p "总结这个仓库"
dst run --json "检查测试失败"
dst session list --json
```

TUI 的配置默认在 `~/.dsh-tui/settings.json`，与 `$DSH_HOME` 分离。

## 验证安装

### Desktop

- 能打开欢迎页/工作区；
- 设置 → 模型里能看到模型；
- 新建会话能收到回复；
- 菜单“管理 dsh 命令…”里能看到命令状态。

### CLI

```bash
dsh --version
dsh --help
dsh web --help
```

启动 Web：

```bash
dsh web --no-open --port 3080
```

浏览器打开 `http://127.0.0.1:3080`。

### TUI

```bash
dsh-tui doctor
dsh-tui
```

能进入界面并正常对话即可。

## 更新

### Desktop

- 应用内“检查更新…”；
- 自动检查不会自动安装；
- 更新前会检查运行中的任务，可能提示中断风险。

### CLI

```bash
npm update -g @deepseek-ai/dsh
# 或
npm install -g @deepseek-ai/dsh@latest
```

### TUI

```bash
dsh-tui update
```

或在 TUI 里 `/update`。

## 卸载

### Desktop

1. 菜单 **管理 dsh 命令… → 移除**；
2. 退出 Desktop；
3. macOS 把应用从“应用程序”移到废纸篓；Windows 在“设置 → 应用”卸载；
4. `$DSH_HOME`（默认 `~/.dsh`）不会被自动删除。

### CLI

```bash
npm uninstall -g @deepseek-ai/dsh
```

### TUI

```bash
npm uninstall -g @deepseek-harness-tui/dsh-tui
```

TUI 卸载不会留下 profile patch；但 `~/.dsh-tui` 和 profile 目录可能保留，按需清理。

## 目录与数据

| 内容 | 默认位置 |
| --- | --- |
| Harness 主目录 | `~/.dsh`，可用 `$DSH_HOME` 覆盖 |
| 凭据 | `$DSH_HOME/.credentials.yaml` |
| 环境变量文件 | `$DSH_HOME/.env` |
| 会话/设置/插件 | `$DSH_HOME/` 下 |
| Web profile | `$DSH_HOME/profiles/web/` |
| Desktop profile | `$DSH_HOME/profiles/desktop/` |
| TUI profile | `$DSH_HOME/profiles/dsh-tui/` |
| 共享 AGENTS.md | `~/.agents/AGENTS.md` |
| 共享 skills | `~/.agents/skills/` |
| Desktop 壳设置（macOS） | `~/Library/Application Support/@deepseek-ai/dsh-desktop/desktop/settings.json` |
| Desktop 壳设置（Windows） | `%APPDATA%\@deepseek-ai\dsh-desktop\desktop\settings.json` |

## 安装检查清单

- [ ] 选对平台/架构
- [ ] Desktop 来自官方下载域名
- [ ] CLI Node 版本 ≥ 24.9
- [ ] TUI 知道它是社区包
- [ ] `dsh --version` 正常
- [ ] `$DSH_HOME` 已备份或确认位置
- [ ] 卸载前移除 Desktop 注册的 `dsh` 命令

## 官方参考

- [官方下载页](https://www.deepseek.com/download/)
- [GitHub README](https://github.com/deepseek-ai/deepseek-harness/blob/master/README.zh.md)
- [CLI README](https://github.com/deepseek-ai/deepseek-harness/blob/master/apps/cli/README.zh.md)
- [Desktop README](https://github.com/deepseek-ai/deepseek-harness/blob/master/apps/desktop/README.zh.md)
- [TUI 项目](https://github.com/ccch1mneyyy/dsh-TUI)
