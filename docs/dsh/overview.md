# DeepSeek Harness 功能全景

DeepSeek Harness（命令名 `dsh`）是 DeepSeek 开源的 Agent Harness，构建在 Cordis 的“一切皆插件”架构上。它既可以是桌面应用，也可以从终端启动 Web UI、TUI、Headless、ACP 和 SDK。

!!! warning "开发者预览"
    DeepSeek Harness 目前是 **developer preview**，官方明确说明会出现破坏兼容性的变更，且尚未完成安全审计。不要把它当成生产级安全边界。

## 它是什么

一个 Agent Harness 负责把模型、工具、上下文、会话、权限和界面组装起来。DSH 的特点是：

- **插件化**：模型、工具、UI、搜索、记忆、终端等都可以是插件；
- **多表面**：Desktop、Web UI、TUI、Headless、ACP、SDK 共用同一套 Agent/Session；
- **profile 启动**：`dsh <profile>` 由按顺序叠加的插件 bundle 和用户 patch 组成；
- **工作区导向**：在某个项目目录里启动，Agent 读写该工作区；
- **权限/沙箱**：支持审批、权限预设、文件沙箱和命令沙箱，但不是绝对隔离。

## 五种使用表面

| 表面 | 启动方式 | 适合 |
| --- | --- | --- |
| **Desktop** | 官方 DMG/EXE | 想要图形界面、少配置的用户 |
| **Web UI** | `dsh web` | 浏览器使用、远程开发、团队共享 |
| **TUI** | 社区 `dsh-tui` / `dst` | 终端党、SSH、低资源环境 |
| **Headless** | `dsh headless "任务"` | 脚本、CI、一次性任务 |
| **ACP / SDK** | `dsh --profile acp` / `sdk` | 编辑器/产品集成、自动化客户端 |

它们不是五个不同的产品，而是同一个 Harness 的不同入口。

## 核心概念

### Profile

`dsh` 启动的不是一个固定应用，而是一个 **profile**：

```text
dsh web       → web profile
dsh headless  → headless profile
dsh tui       → tui profile（需安装 TUI 插件）
dsh --profile acp
dsh --profile sdk
```

profile 目录默认在：

```text
$DSH_HOME/profiles/<profile>/
```

里面有 `package.json`、`dsh.profile`、`cordis.patch.yml`。插件、配置和用户覆盖都按 profile 隔离。

### 插件

DSH 的插件可以加：

- 模型 provider / 模型目录；
- 工具、命令、Hook；
- 客户端 UI 面板、侧栏、设置页；
- 搜索、抓取、记忆、MCP；
- 终端、浏览器、LSP、子代理；
- 主题、快捷键。

官方内置了很多插件，第三方也可以通过 npm、Git 或插件索引分发。

### 工作区

启动 `dsh` 时所在目录是默认工作区根目录。Agent 可以读写该目录、执行命令；权限策略决定是否需要审批。

### 会话

会话持久化保存，包含对话、工具调用、审批、计划、模型切换等。可以恢复、继续、查看轨迹。

## 能做什么

| 能力 | 说明 |
| --- | --- |
| **代码开发** | 读写文件、执行命令、跑测试、改 bug、重构 |
| **代码审查** | 读取 diff、检查风险、给出修改建议 |
| **计划模式** | 先计划再实现，计划需要确认 |
| **子代理** | 并行调研、审查、实现 |
| **Skills** | 按需加载的流程/知识 |
| **MCP** | 接外部工具和数据源 |
| **Web 搜索/抓取** | 搜索、读取网页 |
| **终端/浏览器** | 内置终端面板、浏览器操作插件 |
| **定时任务/Goal** | 会话内提醒、持久目标 |
| **插件市场** | 安装官方/第三方插件，或自己写 |
| **多模型** | DeepSeek 账号/API Key、第三方 provider、自定义兼容端点 |

具体可用能力取决于当前 profile 装了哪些插件。

## 和 Codex / Claude Code / Pi 的区别

| 维度 | DeepSeek Harness | Codex | Claude Code | Pi |
| --- | --- | --- | --- | --- |
| **厂商** | DeepSeek | OpenAI | Anthropic | Earendil |
| **架构** | 插件化 Harness + Cordis | 编码 Agent | 编码 Agent | 极简可编程 Agent |
| **主界面** | Desktop / Web / TUI / CLI | CLI/IDE/app/cloud | CLI/IDE/desktop/web | TUI/print/RPC/SDK |
| **扩展** | 插件 bundle + 插件市场 | Skills/Plugins/MCP | Skills/Plugins/Hooks/MCP | TypeScript 扩展/Packages |
| **模型** | DeepSeek + 多 provider | OpenAI + 自定义 | Claude | 任意 provider |
| **适合** | 想要完整 Harness/桌面体验 | OpenAI 生态 | Claude 生态 | 自己组装 Agent |

## 安全边界

官方安全说明明确：

- 项目可以执行模型生成的代码和命令；
- 会加载第三方插件；
- 能访问被开放的网络、进程、凭据和文件；
- 沙箱、审批和权限控制可以降低风险，但**不保证隔离**；
- 错误模型输出、插件或配置可能损坏文件、泄露数据。

建议：

- 只给最小权限；
- 不信任的仓库/插件在容器或虚拟机里跑；
- 备份重要文件；
- 不把敏感凭据暴露给不需要它的任务；
- 运行前检查插件、配置和命令。

## 建议阅读顺序

1. [安装 DSH](install.md)：Desktop、CLI、TUI，按平台选择。
2. [Desktop 使用](desktop.md)：图形界面、首次配置、插件、CLI 命令。
3. [CLI 与 TUI](cli-tui.md)：Web、Headless、ACP、SDK、社区 TUI。
4. [配置](configuration.md)：模型、代理、AGENTS.md、Skills、权限、插件。
5. [场景与排错](playbooks.md)：真实工作流和常见问题。

## 官方参考

- [DeepSeek Harness 官网](https://www.deepseek.com/harness/)
- [官方文档](https://deepseek-harness.github.io/deepseek-harness/)
- [GitHub](https://github.com/deepseek-ai/deepseek-harness)
- [安全说明](https://github.com/deepseek-ai/deepseek-harness/blob/master/SAFETY.zh.md)
- [CLI 帮助](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/cli-help.zh.md)
