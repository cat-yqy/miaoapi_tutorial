# Pi Agent 功能全景

Pi 是 Earendil 推出的终端 AI 编码 Agent。它的定位和 Codex、Claude Code、OpenCode 不太一样：**核心刻意保持极简，把很多能力交给扩展、技能和包来做**。

官方把它称为 “a hardened, minimal, extensible agent harness”。你可以把它理解成一个可编程的 Agent 内核：自带文件、shell、编辑、会话和模型接入，其他能力按需安装。

!!! note "本文基准"
    本文基于 Pi 1.x（2026 年 10 月官方文档）。Pi 迭代很快，安装命令、扩展 API 和 UI 可能变化；以 `pi --help`、`/help` 和 [官方文档](https://pi.dev/docs/latest)为准。

## Pi 的核心设计

| 原则 | 含义 |
| --- | --- |
| **Minimal core** | 核心只保留 Agent loop、工具、会话、模型接入 |
| **Extensible** | 用 TypeScript 扩展、Skills、Prompt Templates、Packages 增加能力 |
| **Terminal-first** | 主界面是 TUI；同时提供 print、JSON、RPC、SDK |
| **Bring your own model** | 订阅、API Key、本地模型、兼容端点都可以 |
| **Transparent sessions** | 会话是 JSONL 树，可分支、压缩、导出 |

## Pi 1.0 带来的能力

2026 年 10 月发布的 Pi 1.0 加入了：

- **Codemode**：模型写脚本编排工具；
- **原生 MCP 支持**：不用额外适配器也能接 MCP；
- **Deferred tool loading**：工具定义延迟加载，减少上下文占用；
- **Virtual models**：用扩展定义“虚拟模型”路由；
- **Image models**：图片模型接入；
- **Cache warming**：Anthropic 模型缓存预热；
- **全屏 TUI** 和新主题。

此外还有实验性的 **Pi Durable**，面向上层应用构建长运行 Agent。普通终端使用不需要安装它。

## 五个入口

| 入口 | 适合做什么 | 说明 |
| --- | --- | --- |
| **Interactive TUI** | 日常开发 | `pi`，全屏终端界面 |
| **Print mode** | 一次性任务、脚本 | `pi -p "..."` |
| **JSON 模式** | 事件流、自动化 | `pi --mode json "..."` |
| **RPC 模式** | 自建客户端/编辑器集成 | `pi --mode rpc` |
| **SDK** | 把 Pi 嵌进产品 | TypeScript SDK |

## 能做什么

| 能力 | 说明 | 默认/扩展 |
| --- | --- | --- |
| **读写文件** | read / edit / write | 内置 |
| **执行命令** | bash（Windows 用 Git Bash，可选 PowerShell） | 内置 |
| **代码搜索** | grep / find / ls 等 | 内置 |
| **会话与分支** | JSONL 会话树、fork、clone、compact | 内置 |
| **多模型** | 订阅、API Key、本地模型、兼容端点 | 内置 |
| **MCP** | 接外部工具和数据源 | 内置 |
| **Web 搜索/抓取** | 搜索、网页提取、视频理解 | 扩展，如 `pi-web-access` |
| **子代理** | 并行研究、审查、实现 | 扩展，如 `pi-subagents` |
| **计划模式** | 先计划再改代码 | 扩展，如 `@narumitw/pi-plan-mode` |
| **权限门禁** | allow / ask / deny | 扩展，如 `@gotgenes/pi-permission-system` |
| **长期记忆** | 跨会话记住偏好和决定 | 扩展，如 `pi-memory` |
| **后台任务** | 长命令、后台子代理 | 扩展，如 `pi-background-tasks` |
| **可观测性** | 把会话送到 Langfuse/Braintrust | 扩展 |

## Pi 的资源模型

Pi 把“可定制的东西”拆得很细：

| 资源 | 作用 | 位置 |
| --- | --- | --- |
| **AGENTS.md** | 项目/目录级长期指令 | 仓库、父目录、`~/.pi/agent` |
| **SYSTEM.md / APPEND_SYSTEM.md** | 替换/追加系统提示 | `~/.pi/agent` 或 `.pi` |
| **Prompt Templates** | 可复用 `/命令` | `prompts/` |
| **Skills** | 按需加载的知识和流程 | `skills/` |
| **Extensions** | TypeScript 可执行扩展 | `extensions/` |
| **Packages** | 打包分发以上资源 | npm / git / local |
| **MCP** | 外部工具服务器 | `mcp.json` |
| **Themes** | TUI 主题 | `themes/` |

### 什么时候用哪种

- 只是重复一段提示词 → Prompt Template；
- 需要一整套流程/参考资料 → Skill；
- 需要新工具、命令、事件钩子、TUI → Extension；
- 要把多个资源分发给团队 → Package；
- 要接外部系统 → MCP 或 Extension。

## 和 Codex / Claude Code / OpenCode 的区别

| 维度 | Pi | Codex | Claude Code | OpenCode |
| --- | --- | --- | --- | --- |
| **定位** | 极简可编程 Agent | OpenAI 编码 Agent | Anthropic 编码 Agent | 开源多模型 Agent |
| **模型** | 任意 provider/端点 | OpenAI + 自定义 | Claude | 任意 provider |
| **扩展** | TypeScript 扩展 + Packages | Skills/Plugins/MCP | Skills/Plugins/Hooks/MCP | JS/TS Plugins + Skills |
| **默认能力** | 很精简 | 较完整 | 较完整 | 较完整 |
| **沙箱** | 无内置沙箱 | 有沙箱/审批 | 有权限/沙箱 | 权限系统 |
| **界面** | TUI + print/JSON/RPC/SDK | CLI/IDE/app/cloud | CLI/IDE/desktop/web | TUI/web/IDE/server |
| **适合** | 想自己组装 Agent 的人 | OpenAI 生态 | Claude 生态 | 多模型/可编程 |

Pi 的最大优势是“可控和可编程”，最大代价是“很多能力要自己装、自己审”。

## 安全边界

Pi 官方明确说明：

- **没有内置沙箱**；
- 工具和扩展以 Pi 进程的权限运行；
- 项目 trust 只控制加载哪些项目资源，不隔离工具调用；
- Pi 不会在每次工具调用前都询问；
- 扩展可以读提示词、文件、凭据和会话历史。

所以：

- 不要在不信任的仓库里直接跑 Pi；
- 安装扩展前看源码；
- 敏感任务用容器/虚拟机隔离；
- 不要把密钥放在会进入上下文或会话的文件里。

## 建议的阅读顺序

1. [安装 Pi](install.md)：macOS、Linux、Windows、WSL。
2. [日常使用](usage.md)：TUI、会话、命令、CLI 模式。
3. [配置与定制](configuration.md)：AGENTS.md、模型、Skills、MCP。
4. [扩展与生态](extensions.md)：安装扩展、写扩展、推荐清单。
5. [场景实战](playbooks.md)：真实任务和最佳实践。

## 官方参考

- [Pi 文档](https://pi.dev/docs/latest)
- [Quickstart](https://pi.dev/docs/latest/quickstart)
- [How Pi Works](https://pi.dev/docs/latest/how-pi-works)
- [Pi Packages](https://pi.dev/packages)
- [GitHub](https://github.com/earendil-works/pi)
- [Pi 1.0 发布说明](https://earendil.com/posts/pi-1-0/)
