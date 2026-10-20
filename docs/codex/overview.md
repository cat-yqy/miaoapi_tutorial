# Codex 功能全景

Codex 是 OpenAI 的编码智能体。它不只是补全代码，而是能在一个任务里读取仓库、制定计划、修改文件、执行命令、运行测试、审查 diff，最后把结果交给你验收。

!!! note "阅读前提"
    本站通过 MiaoAPI 接入 Codex。CLI、IDE 扩展和本地编辑等能力通常可用；Codex Cloud、GitHub Code Review、Automations、Security Cloud 等与 OpenAI 账号绑定的托管功能，是否可用取决于你的登录方式和套餐。具体以客户端实际显示为准。

## Codex 的六个入口

| 入口 | 适合做什么 | 关键特点 |
| --- | --- | --- |
| **Codex CLI** | 终端里的日常开发、调试、脚本化 | 开源、Rust 编写，能读写文件、跑命令，支持 TUI 和 `codex exec` |
| **IDE 扩展** | 在编辑器里边看代码边改 | 自动带入打开的文件和选中代码，支持 VS Code、Cursor、Windsurf、JetBrains |
| **Codex app** | 多任务并行、看 diff、跑自动化 | 桌面端，支持 Local / Worktree / Cloud 三种模式、内置终端和浏览器 |
| **Codex Cloud** | 长时间、可并行的后台任务 | 在隔离环境里克隆仓库、执行任务并产出 PR，适合离开电脑时继续工作 |
| **GitHub / Slack / Linear** | 团队协作和 PR 流程 | 在 Issue、PR 或聊天里直接 `@codex`，把任务接入现有工作流 |
| **`codex exec` / Action / SDK** | CI、脚本、产品集成 | 非交互执行、JSON 输出、结构化 schema、GitHub Action、TypeScript/Python SDK |

## 2026 年 10 月值得关注的变化

截至 2026 年 10 月，Codex 的新版能力主要集中在“并行、复用和自动化”：

- **GPT-6.1 Sol 成为 CLI 默认模型**（0.159.1 起）。实际可用性取决于套餐、管理员设置和接入方式。
- **CLI 增加语音、`/agents` 视图、提示词回退编辑、`/worktree`**。语音默认开启，`F8` 切换，`/voice settings` 选择声音。
- **Worktree 更适合并行**：每个任务可以在独立 Git worktree 中运行，避免两个 agent 同时修改同一份文件。
- **Code Review 进入桌面端**：支持 GitHub PR（已正式可用）和 GitLab MR（预览），也可以在 GitHub 评论 `@codex review`。
- **Cloud 环境可复用**：仓库、依赖、工具和网络访问可以配置成环境，之后每个任务在隔离工作区中复用。
- **Security Cloud 研究预览**：对已连接的 GitHub 仓库做定时扫描、去重和修复准备。

!!! warning "功能会分批开放"
    新功能可能按账号、套餐、地区或客户端版本逐步开放。如果某个命令不存在，先在客户端输入 `/` 查看实际命令列表，或升级到最新版。

## 能力地图

| 能力 | 具体能做什么 | 典型入口 |
| --- | --- | --- |
| **理解代码库** | 解释模块职责、请求链路、数据模型、历史遗留逻辑 | CLI、IDE、Cloud |
| **写代码 / 改代码** | 按现有风格实现功能、补类型、改接口、迁移 | CLI、IDE、app |
| **调试** | 复现问题、读日志、定位根因、给出最小修复 | CLI、IDE |
| **测试与校验** | 写测试、跑 lint/typecheck、验证行为 | CLI、app、`exec` |
| **代码审查** | 审查未提交改动、某个 commit、相对分支的 diff | `/review`、GitHub |
| **并行任务** | 子代理、多个线程、worktree、Cloud 任务 | CLI、app、Cloud |
| **外部上下文** | 通过 MCP 接文档、浏览器、Figma、Sentry、GitHub 等 | CLI、IDE、app |
| **可复用流程** | 用 Skills 固化步骤，用 Plugins 分发 | CLI、IDE、app |
| **自动化** | 定时任务、CI 审查、脚本化执行 | app Automations、`exec`、Action |
| **多模态** | 读截图、生成/编辑图片、浏览器预览、桌面操作 | CLI、app |
| **安全控制** | 沙箱、审批、规则、Security 扫描 | 全入口 |

## 怎么选入口

- **正在终端里工作**：优先 CLI。改完可以直接跑测试、看 diff、提交。
- **正在编辑器里读代码**：优先 IDE 扩展。选中代码和打开文件会自动成为上下文。
- **同时推进多个任务**：优先 Codex app。用 Local / Worktree / Cloud 分开，减少互相干扰。
- **任务会跑很久，或你想离开电脑**：用 Cloud。让它后台跑，回来再看 diff 和 PR。
- **要把 Codex 接进团队流程**：用 GitHub / Slack / Linear 集成，或 `codex exec` / GitHub Action。
- **要批处理、生成结构化结果**：用 `codex exec --json` 或 `--output-schema`，不要硬解析自然语言。

## 本地能力与托管能力

这是使用第三方 API 时最容易混淆的一点：

| 类型 | 例子 | 依赖 |
| --- | --- | --- |
| **本地能力** | CLI 读写文件、跑测试、Skills、MCP、子代理、`codex exec` | 主要依赖模型接口和本机环境 |
| **托管能力** | Codex Cloud、GitHub Code Review、Automations、Security Cloud | 依赖 OpenAI 账号、套餐、GitHub 连接和工作区配置 |

如果使用 MiaoAPI 的自定义 provider，本地 CLI 工作流通常不受影响；但云端任务、自动审查、Security Cloud 等功能可能无法使用，或者需要另外登录 OpenAI 账号。不要默认认为“能调用模型”就等于“所有云端功能都可用”。

## 建议的阅读顺序

1. [核心机制](core-concepts.md)：先理解线程、上下文、计划、权限和 `AGENTS.md`。
2. [日常使用](daily-workflow.md)：掌握 CLI、IDE、app 和 Code Review 的常用操作。
3. [进阶能力](advanced.md)：子代理、Skills、Plugins、MCP、自动化和 `exec`。
4. [场景实战](playbooks.md)：照着真实任务模板练习。
5. [最佳实践](best-practices.md)：把 Codex 用得更稳、更安全、更省额度。

## 官方参考

- [Codex 官方文档](https://developers.openai.com/codex/)
- [Codex CLI](https://developers.openai.com/codex/cli)
- [Codex IDE extension](https://developers.openai.com/codex/ide)
- [Codex app](https://developers.openai.com/codex/app)
- [Codex Cloud](https://developers.openai.com/codex/cloud)
- [DevDay 2026 更新](https://openai.com/index/devday-2026-recap/)
- [Codex What's New](https://learn.chatgpt.com/docs/whats-new)
- [Codex Security](https://developers.openai.com/codex/security)
- [Codex GitHub Action](https://developers.openai.com/codex/github-action)
- [Codex SDK](https://developers.openai.com/codex/sdk)
