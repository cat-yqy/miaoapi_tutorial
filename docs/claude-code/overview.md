# Claude Code 功能全景

Claude Code 是 Anthropic 的 agentic 编码工具。它能在终端、IDE、桌面端和浏览器里读取仓库、修改文件、执行命令、运行测试、审查代码，并协调多个 agent 一起工作。

!!! note "阅读前提"
    本站通过 [MiaoAPI 接入 Claude Code](../guide/claude-code.md)。终端 CLI、IDE 扩展和本地会话通常可用；Cloud、Remote Control、Ultrareview、Artifacts 等依赖 claude.ai 账号的能力，在第三方网关模式下通常不可用或需要额外验证。具体以 `/status` 和客户端实际显示为准。

## 九个入口

| 入口 | 适合做什么 | 关键特点 |
| --- | --- | --- |
| **Terminal CLI** | 日常开发、调试、脚本化 | `claude`，完整功能基线，支持交互和非交互 |
| **VS Code / Cursor** | 边看代码边改 | 内联 diff、`@` 引用、选中代码上下文、计划审阅 |
| **JetBrains** | IntelliJ、PyCharm、WebStorm 等 | 依赖 CLI，提供交互式 diff 和选区上下文 |
| **Desktop app** | 多会话、可视化审查 | diff 面板、内置浏览器、worktree、Computer Use、定时任务 |
| **Web / 手机** | 长时间、远程、无需本地环境 | `claude.ai/code`，云环境执行，可把会话 teleport 回终端 |
| **Remote Control** | 在手机上继续本地会话 | 代码仍在本机执行，浏览器/手机只是控制端 |
| **Slack** | 从 bug 报告直接产出 PR | `@Claude` 接入团队聊天 |
| **CI / GitHub Actions** | PR 审查、Issue 转 PR、定时任务 | `anthropics/claude-code-action@v1`，响应 `@claude` |
| **Agent SDK** | 把 Claude Code 嵌进自己的产品 | TypeScript / Python，控制编排、工具和权限 |

## 能做什么

| 能力 | 说明 |
| --- | --- |
| **理解代码库** | 解释架构、调用链、数据模型、历史设计 |
| **写代码 / 改代码** | 跨文件实现功能、补类型、修 lint、迁移 |
| **调试** | 读日志、复现问题、定位根因、验证修复 |
| **测试与校验** | 写测试、跑测试、构建、类型检查、截图比对 |
| **Git 操作** | 查看 diff、提交、开分支、创建 PR |
| **代码审查** | `/code-review` 本地审查、Ultrareview 云端深度审查、GitHub PR 审查 |
| **多 agent** | 子代理、Agent View 后台会话、Agent Teams、动态工作流 |
| **外部工具** | MCP 接数据库、浏览器、Figma、Sentry、Slack 等 |
| **可复用流程** | Skills、Plugins、Hooks、自定义 subagents |
| **自动化** | `claude -p`、GitHub Actions、`/loop`、云端/桌面定时任务 |
| **视觉与桌面** | 图片输入、Claude in Chrome、Computer Use、Artifacts |
| **安全与回滚** | 权限模式、沙箱、检查点、rewind |

## 2026 年值得关注的能力

截至 2026 年 10 月，Claude Code 的几个重点方向是“并行、长期运行和可验证”：

- **Auto mode 成为默认起点**：由分类器在后台审查工具调用，拦截高风险动作，减少逐步确认。仍可切回 Manual。
- **Agent View**：`claude agents` 打开后台会话总览，可派发任务、查看状态、附加到某个会话。研究预览。
- **Agent Teams**：一个 lead 带多个 teammate，共享任务列表并互相通信。实验功能，默认关闭。
- **动态工作流**：Claude 写脚本编排大量子代理，适合全仓库审计、迁移和交叉验证。
- **Ultrareview**：`/code-review ultra` 在云端跑多 agent 深度审查。研究预览，依赖 claude.ai。
- **定时任务**：云端 scheduled tasks（`/schedule`）、桌面端定时任务、会话内 `/loop`。
- **Computer Use**：让 Claude 看屏幕、点击、输入，适合 GUI-only 的验证。CLI 版目前以 macOS 为主。
- **Artifacts**：把会话结果发布成可交互页面，适合图表、对比、演示。
- **Remote Control**：从手机或浏览器继续本地会话；但第三方网关模式下通常不可用。

!!! warning "功能可用性差异很大"
    Agent View、Agent Teams、Ultrareview、Artifacts、Cloud、Remote Control 等功能的开放范围不同。使用 MiaoAPI 网关时，先看 `/status` 和功能入口是否出现，不要默认所有 claude.ai 能力都可用。

## 和 Codex 怎么选

| 维度 | Claude Code | Codex |
| --- | --- | --- |
| **模型** | 只支持 Claude 模型 | 主要使用 OpenAI 模型，也支持自定义 provider |
| **项目规则** | `CLAUDE.md`，兼容 `AGENTS.md` | `AGENTS.md` |
| **计划** | Plan mode（`Shift+Tab` / `--permission-mode plan`） | `/plan` |
| **权限** | Auto / Manual / Accept edits / Plan | Auto / Read-only / Full Access |
| **回滚** | 检查点 + `/rewind` | Git diff + worktree |
| **并行** | 子代理、Agent View、Agent Teams、worktree | 子代理、worktree、Cloud |
| **审查** | `/code-review`、Ultrareview、GitHub Code Review | `/review`、GitHub Code Review |
| **自动化** | `-p`、GitHub Actions、`/schedule`、`/loop` | `exec`、GitHub Action、Automations |
| **扩展** | Skills、Plugins、Hooks、MCP | Skills、Plugins、Hooks、MCP |

一个简单的选择：

- 团队和模型都在 Claude 生态：优先 Claude Code。
- 团队使用 OpenAI 或需要混合 provider：优先 Codex。
- 两者都可以接 MiaoAPI，但**不要用 Claude Code 跑非 Claude 模型**。

## 本地能力与账号能力

使用 MiaoAPI 时，最重要的是分清两类能力：

| 类型 | 例子 | 依赖 |
| --- | --- | --- |
| **本地能力** | CLI 读写文件、跑测试、子代理、Skills、MCP、Hooks、worktree | 主要依赖模型接口和本机环境 |
| **账号/云能力** | Cloud、Remote Control、Ultrareview、Artifacts、云端定时任务 | 依赖 claude.ai 登录、套餐和 Anthropic 云服务 |

Remote Control 官方明确要求 `ANTHROPIC_BASE_URL` 指向 Anthropic 官方地址；指向 MiaoAPI 时会不可用。Ultrareview 和云端任务也需要 claude.ai 账号。使用第三方网关时，先把本地工作流用好，再逐项验证云能力。

## 建议的阅读顺序

1. [核心机制](core-concepts.md)：理解 agentic loop、上下文、权限、检查点和 `CLAUDE.md`。
2. [日常使用](daily-workflow.md)：掌握 CLI、计划模式、审查、IDE 和桌面端。
3. [进阶能力](advanced.md)：子代理、Agent Teams、Skills、MCP、Hooks、worktree 和自动化。
4. [场景实战](playbooks.md)：照着真实任务模板练习。
5. [最佳实践](best-practices.md)：把 Claude Code 用得更稳、更省上下文。

## 官方参考

- [Claude Code Overview](https://code.claude.com/docs/en/overview)
- [How Claude Code works](https://code.claude.com/docs/en/how-claude-code-works)
- [Extend Claude Code](https://code.claude.com/docs/en/features-overview)
- [Best practices](https://code.claude.com/docs/en/best-practices)
- [Common workflows](https://code.claude.com/docs/en/common-workflows)
- [Agent teams](https://code.claude.com/docs/en/agent-teams)
- [Ultrareview](https://code.claude.com/docs/en/ultrareview)
- [Claude Code GitHub Actions](https://code.claude.com/docs/en/github-actions)
- [Agent SDK](https://code.claude.com/docs/en/agent-sdk/overview)
