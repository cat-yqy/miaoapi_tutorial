# OpenCode 功能全景

OpenCode 是开源的 AI 编码代理，提供终端 TUI、桌面端、浏览器界面、IDE 集成、GitHub/GitLab 自动化和可编程 Server/SDK。它不绑定单一模型厂商，可以通过 75+ provider 或任意 OpenAI 兼容端点接入模型。

!!! note "版本说明"
    官方目前同时维护 OpenCode 1 和 OpenCode 2。两者都使用 `opencode` 命令，默认不能并排安装。本指南以 **OpenCode 1.x** 为基线，因为 [MiaoAPI 接入页](../guide/opencode.md) 使用的 `opencode.json` 写法属于 1.x；文末会说明 V2 的主要变化。

## 七个入口

| 入口 | 适合做什么 | 关键特点 |
| --- | --- | --- |
| **TUI** | 终端里的日常开发 | 全屏界面、Plan/Build、子代理、`@` 文件引用、`!` shell |
| **Desktop** | 多会话、可视化工作 | 桌面应用，适合不依赖终端的日常使用；V2 提供官方桌面构建 |
| **Web** | 浏览器里用同一套会话 | V1 用 `opencode web`；V2 使用 `opencode pair` |
| **IDE** | 在编辑器里边看边改 | VS Code / Cursor / Windsurf / VSCodium，自动共享选区 |
| **ACP** | 接入其他编辑器 | Zed、JetBrains、Neovim（Avante、CodeCompanion） |
| **GitHub / GitLab** | PR、Issue、CI 自动化 | 评论 `/oc` 或 `/opencode`，在 runner 里执行任务并提 PR |
| **Server / SDK** | 自建客户端、产品集成 | HTTP OpenAPI 3.1、TypeScript SDK、`serve`、`attach` |

## 能做什么

| 能力 | 说明 |
| --- | --- |
| **理解代码库** | 探索目录、搜索符号、解释调用链和数据流 |
| **写代码 / 改代码** | 跨文件实现、重构、迁移、修 lint |
| **执行与验证** | 跑 shell、测试、构建、格式化、类型检查 |
| **多 Agent** | Build / Plan 主 agent，General / Explore / Scout 子代理 |
| **模型选择** | 75+ provider、自定义 provider、模型 variant、不同推理强度 |
| **规则与记忆** | `AGENTS.md`、全局规则、Claude Code `CLAUDE.md` 兼容 |
| **扩展** | Skills、自定义 commands、JS/TS plugins、custom tools、MCP |
| **代码智能** | LSP 诊断、formatter、类型检查 |
| **自动化** | `opencode run`、GitHub Actions、GitLab CI、Server/SDK |
| **会话管理** | 恢复、分叉、undo/redo、导出/导入、用量统计 |

## OpenCode 的架构特点

OpenCode 不是“一个巨大的终端程序”，而是 **client/server** 架构：

- TUI 是客户端；
- 本地启动一个 OpenCode server；
- server 暴露 OpenAPI 3.1 接口；
- Web、IDE、SDK 都可以连接同一个 server；
- `opencode serve` 可以单独启动无头 server；
- `opencode attach` 可以把终端接到已有 server。

这带来的好处：

- 终端和浏览器可以共享同一会话；
- 可以在远程机器跑 server，本地 attach；
- 可以用 SDK/HTTP API 做自动化；
- IDE 插件可以驱动 TUI/server。

## 2026 年值得关注的变化

- **OpenCode 2 已发布**：新的安装方式、官方桌面端、`opencode pair` 网页入口和新的文档结构。V1 仍然可用。
- **Agent 体系更完整**：Build / Plan 两个主 agent，General / Explore / Scout 子代理，支持自定义 agent。
- **Skills 与 Commands 分离**：Skills 是按需加载的知识/流程，Commands 是显式 `/命令`。
- **插件系统**：用 JS/TS 订阅工具、文件、会话、权限、LSP、TUI 等事件。
- **Custom Tools**：用 TypeScript/JavaScript 定义模型可调用的新工具。
- **Server/SDK/ACP**：从“终端工具”扩展成可嵌入的 agent 平台。
- **GitHub/GitLab 集成**：评论触发、CI 执行、自动提 PR。

## OpenCode 1 与 2

| 项目 | OpenCode 1.x | OpenCode 2.x |
| --- | --- | --- |
| 安装脚本 | `https://opencode.ai/install` | `https://opencode.ai/v2/install` |
| npm 包 | `opencode-ai` | `@opencode/cli` |
| Web | `opencode web` | `opencode pair` |
| 桌面端 | 有桌面应用 | 官方桌面构建更完整 |
| 文档 | `/docs` | `/v2/docs` |
| 配置 | `opencode.json` | 以 V2 文档为准，部分字段变化 |

两者默认都占用 `opencode` 命令，不能并排安装。升级前先看 V2 的迁移文档。

## 和 Codex / Claude Code 怎么选

| 维度 | OpenCode | Codex | Claude Code |
| --- | --- | --- | --- |
| **模型** | 任意 provider / 自定义端点 | 主要 OpenAI，也支持自定义 | 只支持 Claude |
| **开源** | 是 | CLI 开源 | 否 |
| **配置** | `opencode.json` + `AGENTS.md` | `config.toml` + `AGENTS.md` | `settings.json` + `CLAUDE.md` |
| **主 Agent** | Build / Plan | 单一主 agent + 模式 | 单一主 agent + 权限模式 |
| **子代理** | General / Explore / Scout | 显式子代理 | Explore / Plan / General |
| **扩展** | Skills、Commands、JS/TS Plugins、Custom Tools、MCP | Skills、Plugins、MCP | Skills、Plugins、Hooks、MCP |
| **服务化** | Server + SDK + ACP | App Server / SDK | Agent SDK |
| **云端托管** | 主要靠自建 server / GitHub Actions | Codex Cloud | Claude Code Web / Cloud |

简单选择：

- 想自由切换模型、自己掌控 server 和插件：OpenCode。
- 深度使用 OpenAI 生态和 Codex Cloud：Codex。
- 深度使用 Claude 生态、Agent Teams 和 Claude Code hooks：Claude Code。

## 本地能力与外部服务

| 类型 | 例子 | 依赖 |
| --- | --- | --- |
| **本地能力** | TUI、文件编辑、shell、子代理、Skills、MCP、Server | 模型接口和本机环境 |
| **托管/第三方** | OpenCode Zen / Go / Console、GitHub Actions、GitLab CI | OpenCode 账号或对应平台 |
| **MiaoAPI** | 自定义 provider，走 `https://miaoapi.site/v1` | MiaoAPI Key 和模型权限 |

使用 MiaoAPI 时，TUI、Server、SDK 和大部分本地扩展都可用；OpenCode Zen/Go 不需要开启。GitHub/GitLab 集成需要把 MiaoAPI 的凭据配置进对应 runner 的 secret。

## 建议的阅读顺序

1. [核心机制](core-concepts.md)：client/server、sessions、agents、permissions、rules。
2. [日常使用](daily-workflow.md)：TUI、CLI、Plan/Build、Web、IDE。
3. [进阶能力](advanced.md)：子代理、Skills、Commands、Plugins、MCP、LSP、Server/SDK。
4. [场景实战](playbooks.md)：真实任务模板。
5. [最佳实践](best-practices.md)：上下文、权限、成本和团队落地。

## 官方参考

- [OpenCode Intro](https://opencode.ai/docs/)
- [OpenCode V2 Docs](https://opencode.ai/v2/docs/)
- [TUI](https://opencode.ai/docs/tui/)
- [CLI](https://opencode.ai/docs/cli/)
- [Agents](https://opencode.ai/docs/agents/)
- [Config](https://opencode.ai/docs/config/)
- [Server](https://opencode.ai/docs/server/)
- [SDK](https://opencode.ai/docs/sdk/)
- [GitHub](https://opencode.ai/docs/github/)
