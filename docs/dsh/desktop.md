# Desktop：安装后的首次配置与日常使用

Desktop 是 DSH 的 Electron 桌面壳：里面运行完整的 Web 应用和内置 Host，不需要你单独装 Node。它适合想直接打开窗口就用的人。

## 安装

官方下载页：<https://www.deepseek.com/download/>

直接下载：

- macOS Apple Silicon（macOS 13+）：
  <https://download.deepseek.com/desktop/dsh-latest-macos-arm64.dmg>
- Windows x64：
  <https://download.deepseek.com/desktop/dsh-latest-windows-x64.exe>

安装细节见[安装页](install.md)。安装包只提供 macOS Apple Silicon 和 Windows x64；Linux 与 Intel Mac 请用 Web/CLI。

## 首次启动

### 1. 欢迎页 / 账号

首次启动会进入欢迎页。按当前版本提示：

- 登录 DeepSeek 账号；或
- 添加 DeepSeek API Key；或
- 配置第三方/自定义 provider。

!!! note "预览版差异"
    不同预览版本的登录入口可能不同。如果登录按钮暂不可用，直接用 API Key：在 [DeepSeek 开放平台](https://platform.deepseek.com/api_keys) 创建密钥，然后在设置 → 模型中填写。

API Key 是只写保存的：保存后界面只显示脱敏描述符，明文不会回显。密钥存储在：

```text
$DSH_HOME/.credentials.yaml
```

### 2. 选择工作区

选择你要让 Agent 操作的项目目录。Desktop 默认绑定 `127.0.0.1`，工作区文件通过 Host 访问。

选择工作区前，会话输入框不可用。

### 3. 选择模型

打开 **设置 → 模型**：

- DeepSeek 卡片：填 API Key；
- 第三方 provider：选择内置 provider（如 `anthropic`、`openai`、`moonshotai`、`zai`）；
- 自定义模型 API：填 Provider ID、Base URL、协议、Key 和模型 ID。

保存后下一次请求生效，不需要重启服务器。

### 4. 发第一个任务

```text
先不要改文件。总结这个仓库的结构，列出主要包和入口。
```

确认 Agent 能读文件、能回复，再开始真实任务。

## 界面怎么用

Desktop 主界面和 Web UI 基本一致：

| 区域 | 用途 |
| --- | --- |
| **左侧栏** | 会话、工作区、文件等 |
| **中间** | 对话、计划、工具调用、轨迹 |
| **输入框** | 发任务；支持 `@` 文件、图片、命令 |
| **右侧栏** | 文件、终端、浏览器等面板（取决于插件） |
| **设置** | 模型、通用、插件、快捷键、账号 |
| **插件页** | 已安装、发现、Creator 模式 |

常用操作：

- 新建会话；
- 选择工作区；
- 查看计划/待办；
- 展开工具调用和轨迹；
- 在设置里切换模型、语言、主题；
- 在插件页安装/禁用插件。

具体布局会随插件和版本变化，以当前界面为准。

## 权限与审批

DSH 支持权限预设、审批和沙箱：

- 某些命令/文件操作会先询问；
- 可以在设置里选择权限预设；
- 不同 profile 的策略可以不同。

不要因为“总弹窗”就无脑选“永远允许”。尤其是：

- `rm`、`git push`、部署、数据库写入；
- 读取 `.env`、SSH key、云凭据；
- 访问工作区之外的目录。

Desktop 的沙箱和审批是风险降低手段，不是绝对隔离。

## 插件

打开 **插件**：

- **已安装**：启用/禁用已装插件；
- **发现**：第三方插件索引（需要先添加索引源）；
- **Creator 模式**：用对话创建/安装插件；
- 官方插件通常随 DSH 提供。

安装第三方插件前：

1. 看来源（npm 包、Git 仓库、commit）；
2. 看权限需求；
3. 看最近维护；
4. 必要时先在隔离环境试。

Desktop 的插件属于 `desktop` profile，和 npm CLI 的 `web`/`headless` profile 分开。npm CLI 不能启动或管理 `desktop` profile；Desktop 自带命令可以。

## Desktop 注册的 `dsh` 命令

菜单 **管理 dsh 命令…** 可以把 Desktop 内置的 CLI 注册到系统：

- macOS：`/usr/local/bin/dsh`；
- Windows：当前用户 PATH；
- 不需要单独装 Node；
- 命令使用 Desktop 的运行时版本；
- 卸载 Desktop 前先“移除”。

注册后打开新终端：

```bash
dsh --version
dsh web
```

这样你可以在终端复用 Desktop 的运行时，但 profile 仍然是分开的。

## 关闭、隐藏与退出

- **关闭主窗口**：macOS `⌘W` / 关闭按钮；Windows `×`、`Alt+F4` 或任务栏关闭。默认是**隐藏**，任务继续跑。
- **macOS 找回窗口**：Dock 图标、再次启动、`dsh://open`。
- **Windows 找回窗口**：系统托盘图标。
- **退出**：macOS `⌘Q`、应用菜单、Dock 菜单；Windows 托盘“退出”。
- 有运行中的 Agent/子代理/后台任务时，退出会提示中断风险。

Windows 首次隐藏窗口前会确认一次；macOS 没有菜单栏图标。

## 更新

- 应用内 **检查更新…**；
- 自动检查存在，但不会自动安装；
- 下载完成后会提示重启；
- 更新前会检查运行中的任务；
- 强制更新策略会阻止继续使用旧版本。

不要从第三方“绿色版/破解版”更新；Desktop 的更新源和签名是安全边界的一部分。

## 数据与备份

Desktop 与 CLI 共享：

```text
$DSH_HOME/                 # 默认 ~/.dsh
├── .credentials.yaml      # 凭据
├── .env                   # 代理/环境变量
├── sessions/              # 会话
├── profiles/
│   ├── desktop/           # Desktop profile
│   ├── web/               # CLI web profile
│   └── dsh-tui/           # TUI profile
└── ...
```

Desktop 壳自己的设置：

- macOS：`~/Library/Application Support/@deepseek-ai/dsh-desktop/desktop/settings.json`
- Windows：`%APPDATA%\@deepseek-ai\dsh-desktop\desktop\settings.json`

备份建议：

```bash
# 退出 Desktop 后
cp -a ~/.dsh ~/.dsh.backup
```

`~/.dsh` 含凭据和会话，按敏感数据保管。不要把它提交到 Git。

## Linux 和 Intel Mac 怎么办

官方 Desktop 目前只有：

- macOS Apple Silicon；
- Windows x64。

其他平台用：

```bash
npx @deepseek-ai/dsh web
```

或安装 CLI/TUI。Linux 的第三方桌面壳不是官方发布，安全性和更新责任由使用者承担。

## 常见问题

| 现象 | 处理 |
| --- | --- |
| macOS 打不开 | 确认 Apple Silicon + macOS 13+；确认官方 DMG；看系统隐私与安全性提示 |
| Windows 安装被 SmartScreen 拦 | 核对官方域名和签名；不要对不明来源包放行 |
| 登录按钮不可用 | 用 DeepSeek API Key；或升级到最新预览版 |
| 选不了工作区 | 先添加/选择工作区；会话输入框在选中前不可用 |
| 模型 401/403 | 检查 Key、provider、Base URL、模型 ID；看设置里的错误 |
| `dsh` 命令找不到 | 菜单“管理 dsh 命令…”安装；打开新终端；检查 PATH |
| 关闭窗口后以为退出了 | 默认隐藏；用 `⌘Q`/托盘退出 |
| 卸载后配置还在 | `$DSH_HOME` 不会被卸载器删除，按需备份/清理 |
| Linux 没有桌面包 | 用 Web UI/CLI/TUI |

## 官方参考

- [Desktop README](https://github.com/deepseek-ai/deepseek-harness/blob/master/apps/desktop/README.zh.md)
- [Desktop 诊断](https://github.com/deepseek-ai/deepseek-harness/blob/master/apps/desktop/docs/diagnostics/README.zh.md)
- [官方下载页](https://www.deepseek.com/download/)
- [配置模型](https://deepseek-harness.github.io/deepseek-harness/guide/providers)
