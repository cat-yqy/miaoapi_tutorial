# 进阶能力：并行、复用与自动化

当日常的“提问 → 修改 → 验证”已经顺手，下一步通常不是换更强的模型，而是把工作流做得更可复用、更可并行、更少重复劳动。

这一页按“从个人到团队”的顺序介绍：子代理、Skills、Plugins、MCP、Automations、Memories、Hooks/Rules，以及 `codex exec`、GitHub Action 和 SDK。

## 子代理：把噪声移出主线程

子代理（Subagents）是 Codex 在主线程之外启动的并行 agent。每个子代理有自己的上下文，最后只把摘要返回主线程。

适合：

- 大范围代码探索；
- 多角度代码审查；
- 并行跑测试或分析日志；
- 把长文档拆成多个片段处理；
- 安全、性能、可维护性分别审查。

不适合：

- 多个子代理同时改同一批文件；
- 任务很小、依赖强顺序；
- 你还没有明确分工。

Codex **不会自动启动子代理**，需要你明确要求。例如：

```text
用三个并行子代理审查这个分支：
1. 安全风险
2. 测试缺口
3. 可维护性

等三个都完成后，按类别汇总，并给出文件和行号。
```

权衡：

- 子代理能减少主线程的上下文污染；
- 但每个子代理都消耗模型和工具调用，总 token 通常更高；
- 写操作并行容易冲突，读多写少更适合并行。

在 CLI 中用 `/agent` 切换子代理线程，新版还可以用 `/agents` 查看总览。

## Skills：把重复流程固化成技能

Skill 是一个带 `SKILL.md` 的目录，用来告诉 Codex“这类任务应该怎么做”。它适合稳定的、可重复的工作流，而不是一次性提示。

### 一个 Skill 的最小结构

```text
my-skill/
├── SKILL.md          # 必需：说明和步骤
├── scripts/          # 可选：确定性脚本
├── references/       # 可选：参考资料
└── assets/           # 可选：模板、素材
```

`SKILL.md` 至少要有 `name` 和 `description`：

```markdown
---
name: release-notes
description: 根据最近提交生成发布说明。当用户要求写 release notes、更新日志或发版说明时使用。
---

1. 读取最近 N 个提交和合并的 PR。
2. 按 feat / fix / refactor / docs 分类。
3. 面向用户描述变化，不要直接复制 commit message。
4. 输出 Markdown，并列出需要人工确认的破坏性变更。
```

### 放在哪里

| 范围 | 位置 | 适合 |
| --- | --- | --- |
| 仓库 | `.agents/skills` | 团队共享、跟代码一起版本化 |
| 个人 | `~/.agents/skills` | 跨项目个人习惯 |
| 管理员 | `/etc/codex/skills` | 机器或容器级默认 |
| 系统 | Codex 内置 | 官方提供的基础能力 |

### 怎么触发

- 显式：输入 `$skill-name`，或用 `/skills` 选择；
- 隐式：Codex 根据 `description` 判断当前任务是否匹配。

因此 `description` 非常关键。要写清楚“做什么”和“什么时候用”，并包含用户真实会说的触发词。

### 什么时候该做成 Skill

一个很实用的判断标准：

> 如果你第三次复制同一段提示词，或者第三次纠正 Codex 的同一个流程，它就应该变成 Skill。

适合做成 Skill 的：

- 发布说明；
- PR 审查清单；
- 日志排障；
- 迁移计划；
- 事故总结；
- 固定的数据/报表流程。

可以用内置的 `$skill-creator` 生成初版，也可以手动创建。先用一个真实任务把流程跑通，再抽象成 Skill；不要一开始就设计“大而全”的技能。

## Plugins：把 Skills、Apps 和 MCP 打包分发

Plugin 是 Codex 的可安装分发单位，可以包含：

- 一个或多个 Skills；
- 外部 App 连接（GitHub、Slack、Gmail、Google Drive 等）；
- MCP 服务器；
- 展示素材和配置。

在 CLI 里输入 `/plugins`，或在桌面端的 Plugins 页面浏览和安装。典型插件包括：

- **GitHub**：读 PR、Issue、Review；
- **Slack**：总结频道、起草回复；
- **Gmail / Google Drive**：处理邮件和文档；
- **Codex Security**：扫描代码、验证漏洞、准备修复；
- **Sites**：创建并部署网站。

安装插件不等于绕过权限。插件里的 App 仍然受各自认证和隐私策略约束，Codex 的审批设置也仍然生效。不要把“安装方便”当成“可以无脑授权”。

## MCP：把外部系统和上下文接进来

MCP（Model Context Protocol）让 Codex 连接外部工具和数据源。适合：

- 需要实时文档，而不是模型记忆；
- 需要访问浏览器、Figma、Sentry、GitHub 等系统；
- 需要让 Codex 调用一个真实工具，而不是靠你复制粘贴。

### 用 CLI 添加

```bash
codex mcp add context7 -- npx -y @upstash/context7-mcp
```

### 用 config.toml 配置

```toml
[mcp_servers.context7]
command = "npx"
args = ["-y", "@upstash/context7-mcp"]
```

HTTP 服务器可以这样配置：

```toml
[mcp_servers.figma]
url = "https://mcp.figma.com/mcp"
bearer_token_env_var = "FIGMA_OAUTH_TOKEN"
```

在 CLI 里用 `/mcp` 查看当前可用的 MCP 工具。

!!! warning "MCP 也是信任边界"
    MCP 服务器能读数据、调工具，某些还能执行操作。只安装你信任的服务器，限制可用工具，敏感环境优先使用只读或按次审批。不要因为“工具很多”就全部接进来。

## Automations：把稳定流程按计划跑起来

Automations 目前在 Codex app 中使用，适合重复性任务：

- 每天总结最近提交；
- 定期扫描潜在 bug；
- 检查 CI 失败；
- 生成发布说明；
- 汇总 standup；
- 定时做安全/依赖巡检。

两种形态：

- **独立/项目自动化**：每次启动新的运行，结果进入 Triage 收件箱；
- **Thread 自动化**：附着在当前线程上，按分钟、每天或每周唤醒，保留线程上下文。

关键限制和建议：

- 项目级自动化要求机器开机、Codex 运行、项目仍在磁盘上；
- Git 仓库可以选择在本地项目或独立 worktree 中运行；
- 无人值守任务沿用你的沙箱设置，Full Access 下风险更高；
- **先在普通对话里手动跑通，再设置定时**；
- 先用 Skill 固化方法，再用 Automation 设定频率。

一句话：**Skill 定义“怎么做”，Automation 定义“什么时候做”。**

## Memories：记住偏好，而不是记住秘密

Memories 默认关闭。开启后，Codex 可以把过去线程中的稳定偏好、技术栈、项目约定和常见坑带到后续工作里。

```toml
[features]
memories = true
```

也可以用 `/memories` 按线程控制。

使用建议：

- 适合记：个人偏好、常用命令、稳定约定；
- 不适合记：密钥、临时决定、必须严格生效的团队规则；
- 必须生效的规范仍然放在 `AGENTS.md` 或仓库文档里；
- 共享 `~/.codex` 目录前先检查 memory 文件。

## Hooks 与 Rules：更细的控制

- **Hooks**：在会话开始、工具调用前后、压缩前后等生命周期节点执行自动化逻辑。适合团队审计、通知、策略检查。
- **Rules**：对命令前缀设置允许、询问或禁止，适合“只放行某几个命令”，而不是整域放开权限。

这两项更偏团队和平台工程。个人使用时，先把权限模式、`AGENTS.md` 和 `codex exec` 用好。

## `codex exec`：把 Codex 接进脚本

`codex exec` 是非交互模式，适合 CI、定时任务和管道。

```bash
codex exec "总结这个仓库的结构，列出 5 个高风险区域"
```

默认是只读沙箱。需要改文件时显式指定：

```bash
codex exec --sandbox workspace-write "修复 CI 失败并运行相关测试"
```

机器可读输出：

```bash
codex exec --json "分析仓库结构" | jq
```

结构化输出：

```bash
codex exec "提取项目元数据" \
  --output-schema ./schema.json \
  -o ./project-metadata.json
```

其他常用参数：

- `--ephemeral`：不持久化会话文件；
- `--ignore-user-config`：不加载用户配置；
- `--ignore-rules`：跳过用户/项目规则；
- `CODEX_API_KEY`：只对单次 `exec` 生效。

!!! warning "CI 里不要随便暴露 Key"
    不要把 API Key 设成整个 job 的环境变量，尤其当 job 会检出或执行仓库里的代码时。优先使用 Codex GitHub Action，或只对单次 `codex exec` 注入密钥。

## GitHub Action 与 SDK

如果要在 GitHub 里自动审查 PR、跑质量门禁或生成发布说明，优先使用官方 Action：

```yaml
- uses: openai/codex-action@v1
  with:
    openai-api-key: ${{ secrets.OPENAI_API_KEY }}
    prompt-file: .github/codex/prompts/review.md
    output-file: codex-output.md
```

安全要点：

- 限制谁能触发 workflow；
- 对 PR 文本、Issue 内容保持警惕，防止 prompt injection；
- 使用 `safety-strategy: drop-sudo` 或非特权用户；
- 把 Codex 放在 job 的最后一步；
- 怀疑泄露时立即轮换 Key。

如果要把 Codex 嵌进自己的产品或内部平台，可以用 TypeScript / Python SDK，而不是解析 CLI 的自然语言输出。

## 浏览器与 Computer Use

Codex app 内置浏览器可以预览本地页面、标注元素并让 Codex 修改；Computer Use 可以看屏幕、点击和输入，适合：

- 测试桌面应用；
- 复现 GUI-only 的 bug；
- 操作没有 API 的内部系统；
- 检查模拟器或浏览器流程。

限制：

- 内置浏览器不支持登录态、Cookie、扩展和已有标签页；
- Computer Use 会影响项目之外的状态；
- 任务要窄，权限提示要逐条看；
- 不要把“自动化整个桌面”当成默认工作方式。

## 安全扫描

Codex Security 提供两种形态：

- **Security 插件**：在本地线程里扫描授权代码、验证发现、准备修复；
- **Security Cloud**：连接 GitHub 仓库做定时扫描、去重和修复准备，目前是研究预览。

它适合作为安全审查的补充，不替代 SAST、依赖扫描、渗透测试和人工安全评审。

## 能力采用顺序

不要一次把所有功能都打开。推荐顺序：

1. **提示词 + 验证**：先把单任务做稳。
2. **`AGENTS.md`**：把重复规则固化。
3. **Skills**：把稳定流程固化。
4. **MCP / Plugins**：接入真实外部上下文。
5. **Subagents / Worktrees**：需要并行时再加。
6. **Automations / `exec` / Action**：流程稳定后再自动化。

## 官方参考

- [Subagents](https://developers.openai.com/codex/concepts/subagents)
- [Agent Skills](https://developers.openai.com/codex/skills)
- [Plugins](https://developers.openai.com/codex/plugins)
- [MCP](https://developers.openai.com/codex/mcp)
- [Automations](https://developers.openai.com/codex/app/automations)
- [Memories](https://developers.openai.com/codex/memories)
- [Hooks](https://developers.openai.com/codex/hooks)
- [Rules](https://developers.openai.com/codex/rules)
- [Non-interactive mode](https://developers.openai.com/codex/noninteractive)
- [GitHub Action](https://developers.openai.com/codex/github-action)
- [SDK](https://developers.openai.com/codex/sdk)
- [Codex Security](https://developers.openai.com/codex/security)
