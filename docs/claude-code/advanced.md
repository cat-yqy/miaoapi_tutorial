# 进阶能力：并行、复用与自动化

当单会话工作流已经顺手，下一步通常是把“研究、验证、重复劳动”移出主上下文，或者让多个会话并行推进。

这一页介绍 Claude Code 的扩展层：子代理、Agent View、Agent Teams、动态工作流、Skills、Plugins、MCP、Hooks、worktree 和自动化。

## 子代理：把噪声移出主上下文

子代理（Subagents）是在独立上下文里工作的专门 agent。它读很多文件、跑很多搜索，最后只把摘要返回主会话。

内置子代理：

| 类型 | 用途 | 特点 |
| --- | --- | --- |
| **Explore** | 代码库搜索、文件发现 | 只读，不能写文件 |
| **Plan** | 计划阶段的研究 | 只读，用于收集上下文 |
| **General-purpose** | 复杂多步任务 | 工具更完整，可探索和修改 |

适合：

- 大范围代码调研；
- 多角度审查；
- 并行验证不同假设；
- 把日志、搜索结果等噪声隔离掉。

自定义子代理放在：

```text
.claude/agents/<name>.md      # 项目级
~/.claude/agents/<name>.md    # 个人级
```

一个最小定义：

```markdown
---
name: security-reviewer
description: 审查代码中的安全漏洞。用于提交前或 PR 审查。
tools: Read, Grep, Glob, Bash
model: sonnet
---

你是资深安全工程师。重点检查注入、鉴权、密钥泄露和不安全的数据处理。
给出文件、行号和可执行的修复建议。
```

使用：

```text
Use the security-reviewer agent to review this diff.
```

注意：

- 子代理有自己的上下文和 token 消耗；
- 主会话只看到摘要；
- 子代理的编辑通常不能通过主会话的 rewind 恢复；
- 读多写少的任务最适合并行。

## Agent View：后台会话总览

`claude agents` 打开一个总览界面，管理多个后台会话：

```bash
claude agents
```

你可以：

- 派发独立任务；
- 查看哪些会话在运行、等待输入或已完成；
- 打开某个会话的最近输出；
- 附加到会话继续对话；
- 把当前会话放到后台（`/bg`）；
- 用 `/fork` 复制当前会话到后台。

适合：

- 同时修 bug、看 PR、查 flaky test；
- 长任务后台跑，自己继续做别的事；
- 不想开一堆终端窗口。

Agent View 目前是研究预览。后台会话会独立消耗额度，派发前先确认任务边界。

## Agent Teams：让多个会话协作

Agent Teams 是一个 lead 带多个 teammate，共享任务列表，teammate 之间可以直接通信。

默认关闭。启用：

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

或设置同名环境变量。

适合：

- 多角度研究后互相质疑；
- 新模块拆成互不重叠的部分；
- 用竞争假设并行调试；
- 前端、后端、测试各由不同 teammate 负责。

不适合：

- 强顺序任务；
- 多个 agent 改同一批文件；
- 任务很小，协调成本高于收益；
- 预算敏感的场景，teammate 是独立 Claude 实例，token 消耗明显更高。

Agent Teams 仍是实验功能，会话恢复、任务协调和关闭行为都有已知限制。先用子代理或 Agent View，确实需要“团队协作”再开。

## 动态工作流：脚本编排大量子代理

当任务大到“十几个子代理”也不够，或者需要交叉验证发现，Claude 可以写一个 workflow 脚本，在后台批量运行子代理，最后只返回一个结果。

适合：

- 全仓库审计；
- 大规模迁移；
- 多角度生成方案并互相比对；
- 让第二组 agent 验证第一组的发现。

和普通子代理的区别：

- 子代理由主 agent 逐轮决定；
- 动态工作流由脚本决定编排。

这一步的协调成本较高，先用普通子代理跑通，再考虑工作流。

## Skills：把重复流程固化

Skill 是一个带 `SKILL.md` 的目录，用来教 Claude 一类任务怎么做。

位置：

| 范围 | 位置 |
| --- | --- |
| 项目 | `.claude/skills/<name>/SKILL.md` |
| 个人 | `~/.claude/skills/<name>/SKILL.md` |
| 插件 | `<plugin>/skills/<name>/SKILL.md` |

最小示例：

```markdown
---
name: fix-issue
description: 根据 GitHub Issue 修复问题并创建 PR
disable-model-invocation: true
---

分析并修复 GitHub Issue：$ARGUMENTS。

1. 用 `gh issue view` 读取详情
2. 定位相关代码
3. 实现最小修复
4. 补测试并运行
5. 跑 lint 和类型检查
6. 创建 commit 和 PR
```

使用：

```text
/fix-issue 1234
```

要点：

- `description` 决定 Claude 什么时候自动使用它；
- `disable-model-invocation: true` 用于有副作用的流程，只允许手动触发；
- Skill 可以包含脚本、参考资料和模板；
- 旧的 `.claude/commands/*.md` 仍然可用，并已合并到 Skill 体系；
- 如果同一段提示词或流程反复出现，就把它做成 Skill。

## Plugins：打包分发 Skills、Agents、Hooks 和 MCP

Plugin 是 Claude Code 的可安装单位，可以包含：

- Skills；
- Subagents；
- Hooks；
- MCP servers；
- 其他配置和资源。

使用：

```text
/plugin
```

从 marketplace 浏览、安装、启用或禁用。常见用途：

- 团队统一审查流程；
- 安装 code intelligence 插件，获得符号跳转和实时类型错误；
- 打包一组 MCP 集成；
- 分发部署、发布、迁移流程。

安装前要意识到：**插件在你的权限下运行**。它能让 Claude 执行命令、读文件、调外部服务。只安装信任来源，必要时先看插件的 hooks 和 MCP 配置。

## MCP：连接外部系统

MCP 让 Claude 访问仓库之外的上下文和工具：

- 数据库查询；
- Slack / Notion / Linear；
- Figma 设计稿；
- Sentry 日志；
- 浏览器自动化；
- 内部 API。

添加远程 HTTP 服务器：

```bash
claude mcp add --transport http notion https://mcp.notion.com/mcp
```

查看和管理：

```text
/mcp
```

MCP 支持本地 stdio、远程 HTTP/SSE/WebSocket、OAuth、项目级 `.mcp.json` 和用户级配置。团队可以把项目级 MCP 配置提交到仓库，但凭据要放在环境变量或 OAuth 流程里。

!!! warning "MCP 是信任边界"
    工具能读数据、执行操作。只安装可信服务器，限制工具范围，敏感环境优先只读或逐次审批。

## Hooks：必须每次都发生的自动化

`CLAUDE.md` 是建议，Hooks 是确定性执行。Hooks 在生命周期事件上运行脚本、HTTP 请求、MCP 工具、提示词或子代理。

常见事件：

- `PreToolUse`：工具执行前，可阻止危险命令；
- `PostToolUse`：编辑后自动格式化或跑 lint；
- `SessionStart`：会话开始时注入环境信息；
- `Stop`：任务结束时跑检查，未通过则阻止结束；
- `PreCompact` / `PostCompact`：压缩前后保留关键信息。

配置在 `.claude/settings.json` 或用户设置中。运行：

```text
/hooks
```

查看和信任当前 hooks。

适合：

- 每次编辑后格式化；
- 提交前强制 lint；
- 阻止写入 migrations 目录；
- 在任务结束前跑测试。

!!! warning "Hooks 会在你的机器上执行命令"
    只信任你读过的 hook。团队成员共享的 `.claude/settings.json` 也要审查，不要盲目接受。

## 安全审查

Claude Code 的安全能力分几层：

- `/security-review`：检查当前 diff 中的安全问题；
- **security-guidance 插件**：在 Claude 写代码时提示漏洞并尝试在同一会话修复；
- **Claude Security 插件**：扫描代码库、验证发现、生成可审查的补丁；
- **Hooks**：在提交前强制跑安全或合规检查。

安全扫描是补充，不替代 SAST、依赖扫描、渗透测试和人工安全评审。

## 其他协作入口

| 能力 | 用途 | 备注 |
| --- | --- | --- |
| **Cross-session messaging** | 让并行会话互相传递发现 | 适合你自己管理的多个会话 |
| **Channels** | 从 Telegram、Discord、iMessage 或 webhook 推事件进会话 | 研究预览 |
| **Projects** | 在云端协调多个并行会话 | 依赖 claude.ai |

这些能力都建立在“会话”之上。先用好单会话和子代理，再考虑跨会话协作。

## Worktree：并行会话的文件隔离

在独立 Git worktree 中启动会话：

```bash
claude --worktree feature-auth
# 或
claude -w feature-auth
```

默认位置：

```text
.claude/worktrees/<name>/
```

每个 worktree 有自己的文件和分支，适合：

- 同时开发两个功能；
- 一个会话重构，另一个修 bug；
- 让 subagent 在隔离环境里改代码；
- 长任务不污染主工作区。

建议把 `.claude/worktrees/` 加进 `.gitignore`。worktree 是新的 checkout，依赖和环境要重新初始化。

## 定时与长期任务

三种常见方式：

| 方式 | 在哪里跑 | 适合 |
| --- | --- | --- |
| `/loop` | 当前 CLI 会话 | 轮询部署、反复检查、短周期任务 |
| 桌面定时任务 | 你的电脑 | 需要本地文件和工具的任务 |
| 云端 scheduled tasks | Anthropic 云 | 电脑关机也要跑、定时 PR 审查、依赖审计 |

云端任务用 `/schedule` 创建：

```text
/schedule 每天 9 点审查昨天合并的 PR
```

云端定时任务依赖 claude.ai 账号和 GitHub 连接；使用 MiaoAPI 网关时通常不可用。`/loop` 是本地的，更适合当前环境。

## Computer Use、Chrome 和 Artifacts

### Computer Use

让 Claude 看屏幕、点击和输入，适合：

- 测试桌面应用；
- 复现 GUI-only bug；
- 操作没有 API 的内部系统；
- 检查模拟器流程。

CLI 版目前主要在 macOS 可用，需要开启 `computer-use` MCP；桌面端在 macOS / Windows 提供图形化设置。只做窄任务，逐条看权限提示。

### Chrome

Claude in Chrome 让 Claude 操作浏览器，适合调试 Web 应用、读取页面、复现前端问题。它和 Computer Use 是不同入口：Chrome 更聚焦浏览器，Computer Use 更通用。

### Artifacts

Artifacts 把会话结果发布成可交互的私有网页，适合：

- 带注释的 diff；
- 图表和对比；
- 方案展示；
- 事故时间线；
- 幻灯片或文档。

需要 claude.ai 登录；MiaoAPI 网关模式下通常不可用。

## CI、GitHub Actions 和 Agent SDK

### 非交互模式

```bash
claude -p "解释这个项目"
claude -p "列出所有 API 端点" --output-format json
claude -p "分析日志" --output-format stream-json --verbose
```

常用参数：

- `--allowedTools`：预批准特定工具；
- `--permission-mode dontAsk`：无人值守时只允许预批准工具；
- `--no-session-persistence`：不保存会话。

### GitHub Actions

官方 Action 可以在 Issue/PR 里响应 `@claude`：

```yaml
- uses: anthropics/claude-code-action@v1
  with:
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

也支持通过 `prompt` 在任意 GitHub 事件上运行。安全要点：

- 保护 API Key；
- 限制谁能触发；
- 不要把不可信的 PR 文本直接喂给高权限任务；
- 让 Action 只拥有完成任务所需的最小权限。

### Agent SDK

如果要把 Claude Code 的能力嵌进自己的产品、内部平台或自动化系统，用 Agent SDK：

- TypeScript；
- Python；
- 可控制工具、权限、会话、流式输出和结构化结果。

适合做自定义 agent、批处理平台、内部代码审查服务。不要用解析 CLI 自然语言的方式做产品集成。

## 采用顺序

1. **CLAUDE.md + 验证**：先把单会话做稳。
2. **Skills**：把重复流程固化。
3. **MCP / Plugins**：接真实外部上下文。
4. **Subagents**：隔离研究噪声。
5. **Worktree / Agent View**：需要并行时再加。
6. **Agent Teams / 动态工作流**：任务确实需要协作时再用。
7. **CI / SDK / 定时任务**：流程稳定后自动化。

## 官方参考

- [Subagents](https://code.claude.com/docs/en/sub-agents)
- [Agent view](https://code.claude.com/docs/en/agent-view)
- [Agent teams](https://code.claude.com/docs/en/agent-teams)
- [Skills](https://code.claude.com/docs/en/skills)
- [Plugins](https://code.claude.com/docs/en/plugins/overview)
- [MCP](https://code.claude.com/docs/en/mcp)
- [Hooks](https://code.claude.com/docs/en/hooks)
- [Worktrees](https://code.claude.com/docs/en/worktrees)
- [Scheduled tasks](https://code.claude.com/docs/en/routines)
- [Computer use](https://code.claude.com/docs/en/computer-use)
- [GitHub Actions](https://code.claude.com/docs/en/github-actions)
- [Agent SDK](https://code.claude.com/docs/en/agent-sdk/overview)
