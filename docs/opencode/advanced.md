# 进阶能力：Agent、扩展与平台化

单会话的 Plan/Build 用顺之后，OpenCode 的下一步是把研究、重复流程和外部系统接进来：自定义 agent、Skills、Commands、Plugins、MCP、Custom Tools，以及 Server/SDK。

!!! note "版本差异"
    这一页以 OpenCode 1.x 的扩展接口为准。OpenCode 2 的安装、桌面/网页入口和部分 API 有变化，迁移前先看 V2 文档。

## 自定义 Agent

OpenCode 的 agent 可以理解为“带独立提示词、模型、工具和权限的助手”。

两种类型：

- **Primary**：你直接对话的主 agent，用 `Tab` 切换；
- **Subagent**：主 agent 自动委派，或你用 `@name` 手动调用。

配置方式：

- `opencode.json` 的 `agent` 字段；
- `.opencode/agents/<name>.md`（项目）；
- `~/.config/opencode/agents/<name>.md`（全局）。

最小 Markdown agent：

```markdown
---
description: 审查代码中的安全问题
mode: subagent
model: miaoapi/gpt-6.1-sol
tools:
  write: false
  edit: false
---

你是资深安全工程师。检查注入、鉴权、密钥泄露和不安全的数据处理。
给出文件、行号和可执行的修复建议。
```

也可以用命令生成：

```bash
opencode agent create
opencode agent list
```

关键选项：

- `description`：主 agent 决定何时委派的依据；
- `model`：给这个 agent 指定模型；
- `tools`：限制可用工具；
- `permission`：更细的 allow / ask / deny；
- `mode`：`primary`、`subagent` 或 `all`。

## 子代理与并行

内置子代理：

- **General**：完整工具，适合多步任务；
- **Explore**：只读，适合代码库搜索；
- **Scout**：只读，适合外部文档和依赖研究。

用法：

```text
@explore 找出所有处理 OAuth 回调的文件
@general 并行检查这三个模块的测试覆盖
```

子代理会创建子会话。用 leader 快捷键在父子会话之间切换，避免把大量探索输出塞进主会话。

适合：

- 大范围代码调研；
- 多角度审查；
- 并行研究不同假设；
- 把日志和搜索噪声隔离掉。

不适合：

- 多个子代理同时改同一批文件；
- 强顺序任务；
- 小任务。

## Skills

Skill 是按需加载的 `SKILL.md`，适合放可复用知识、流程和参考材料。

位置：

```text
.opencode/skills/<name>/SKILL.md       # 项目
~/.config/opencode/skills/<name>/SKILL.md  # 全局
.claude/skills/<name>/SKILL.md         # Claude 兼容
.agents/skills/<name>/SKILL.md         # agent 兼容
```

最小示例：

```markdown
---
name: release-notes
description: 根据最近的 PR 和提交生成发布说明
metadata:
  opencode/slash: "true"
  opencode/autoinvoke: "false"
---

1. 读取最近的合并记录。
2. 按 feat / fix / perf / docs 分类。
3. 面向用户描述影响，不复制 commit message。
4. 单独列出破坏性变更。
```

字段：

- `name`：小写、连字符分隔，必须和目录名一致；
- `description`：1–1024 字符，决定何时自动使用；
- `metadata["opencode/slash"]`：设为 `"true"` 时暴露为 `/name`；
- `metadata["opencode/autoinvoke"]`：设为 `"false"` 时不让模型自动选择。

Skills 适合“知识 + 流程”，不是传统意义的可执行插件。需要执行逻辑时，用 Custom Tools 或 Plugins。

## Commands：显式斜杠命令

Command 是一个可复用的提示词模板，适合“我明确要跑这个流程”的场景。

位置：

```text
.opencode/commands/test.md
~/.config/opencode/commands/test.md
```

示例：

```markdown
---
description: 运行测试并分析失败
agent: build
---

运行完整测试套件，展示失败用例。
只分析失败原因，不要直接修改代码。
```

使用：

```text
/test
```

支持 `$ARGUMENTS` 参数：

```markdown
---
description: 创建组件
---

创建一个名为 $ARGUMENTS 的 React 组件，使用 TypeScript。
```

Command vs Skill：

- Command：你显式触发，适合固定动作；
- Skill：模型也可以按描述自动加载，适合知识和可复用流程。

## Plugins：JS/TS 事件钩子

OpenCode 的 Plugin 是可执行扩展，不是“技能包”。它是 JavaScript/TypeScript 模块，可以订阅生命周期事件并修改行为。

加载方式：

- 本地：`.opencode/plugins/` 或 `~/.config/opencode/plugins/`；
- npm：在 `opencode.json` 的 `plugin` 数组里声明，启动时自动安装。

```json
{
  "plugin": ["opencode-helicone-session", "@my-org/custom-plugin"]
}
```

插件可以监听：

- 工具执行前后；
- 文件变化；
- 会话和消息；
- 权限请求；
- LSP；
- 命令执行；
- TUI 事件。

典型用途：

- 通知；
- 保护 `.env`；
- 注入环境变量；
- 自定义工具；
- 压缩钩子；
- 审计和日志。

!!! warning "插件在你的权限下运行"
    插件可以执行命令、读文件和改行为。只安装可信来源，安装前看它的 hooks 和依赖。

## Custom Tools：给模型加新工具

Custom Tools 是用 TypeScript/JavaScript 定义的模型可调用函数。

位置：

```text
.opencode/tools/database.ts
~/.config/opencode/tools/database.ts
```

示例：

```ts
import { tool } from "@opencode-ai/plugin"

export default tool({
  description: "Query the project database",
  args: {
    query: tool.schema.string().describe("SQL query"),
  },
  async execute(args) {
    return `Executed: ${args.query}`
  },
})
```

文件名会成为工具名。一个文件导出多个工具时，名称是 `<文件名>_<导出名>`。

适合：

- 内部 API；
- 数据库查询；
- 公司内部 CLI；
- 需要确定性逻辑的重复操作。

## MCP

MCP 用来接外部工具和数据源。OpenCode 支持本地和远程服务器。

```json
{
  "mcp": {
    "context7": {
      "type": "local",
      "command": ["npx", "-y", "@upstash/context7-mcp"]
    },
    "remote-example": {
      "type": "remote",
      "url": "https://mcp.example.com",
      "headers": {
        "Authorization": "Bearer TOKEN"
      }
    }
  }
}
```

管理：

```bash
opencode mcp add
opencode mcp list
opencode mcp auth <name>
opencode mcp debug <name>
```

!!! warning "MCP 会占上下文"
    每个 MCP 工具的定义都会进入上下文。GitHub 等大型 MCP 很容易吃掉大量 token。只启用当前任务需要的服务器。

## LSP

OpenCode 内置很多语言的 LSP，但 **默认关闭**。

启用：

```json
{
  "lsp": true
}
```

LSP 能提供：

- 类型错误和警告；
- 跳转定义；
- 查找引用；
- 编辑后的即时诊断。

但也有代价：

- 语言服务器可能占内存；
- 版本和项目不一致时会误报；
- 大型仓库可能变慢。

很多项目让 agent 直接跑 `tsc`、`eslint`、`pytest`、`cargo check` 更稳定。把这些命令写进 `AGENTS.md`，比依赖 LSP 更可控。

## Formatters

OpenCode 可以在文件修改后自动运行格式化工具。方便，但也会让 diff 变大。

建议：

- 团队统一 formatter 配置；
- 不要让 formatter 在没有约定的情况下重排整个文件；
- 提交前单独检查格式化产生的 diff。

## Server、SDK 与 ACP

### Server

```bash
opencode serve --port 4096
```

提供 OpenAPI 3.1 接口，默认文档：

```text
http://localhost:4096/doc
```

可以设置 `OPENCODE_SERVER_PASSWORD` 启用 Basic Auth。

### SDK

TypeScript SDK：

```bash
npm install @opencode-ai/sdk
```

```ts
import { createOpencode } from "@opencode-ai/sdk"

const opencode = await createOpencode()
```

适合：

- 自建客户端；
- 内部平台集成；
- 自动化流水线；
- 自定义 UI。

### ACP

OpenCode 支持 Agent Client Protocol，可以接入：

- Zed；
- JetBrains IDE；
- Neovim（Avante、CodeCompanion）。

这让 OpenCode 不只是一个终端程序，而是可以被编辑器/客户端调用的 agent backend。

## GitHub 与 GitLab

### GitHub

```bash
opencode github install
```

评论触发：

```text
/oc 解释这个 issue
/opencode 修复这个 bug 并提 PR
```

支持 Issue 评论、PR 评论、PR review comment、schedule、workflow_dispatch 等事件。

### GitLab

可以通过 GitLab CI 组件运行，或在评论里 `@opencode` 使用 GitLab Duo 集成。凭据和 runner 配置由 GitLab 侧管理。

## Policies（实验）

Policies 控制 OpenCode 是否能使用某个资源，目前主要用于 provider：

```json
{
  "experimental": {
    "policies": [
      { "effect": "deny", "action": "provider.use", "resource": "*" },
      { "effect": "allow", "action": "provider.use", "resource": "miaoapi" }
    ]
  }
}
```

它和 permissions 不同：permissions 控制工具能做什么，policies 控制能不能使用某个 provider。

## 采用顺序

1. **Plan/Build + `AGENTS.md`**：先把单会话做稳。
2. **Commands**：固化显式动作。
3. **Skills**：固化可复用知识。
4. **MCP**：接真实外部上下文。
5. **Subagents**：隔离研究噪声。
6. **Custom Tools / Plugins**：需要确定性逻辑时再加。
7. **Server/SDK/CI**：流程稳定后平台化。

## 官方参考

- [Agents](https://opencode.ai/docs/agents/)
- [Skills](https://opencode.ai/docs/skills/)
- [Commands](https://opencode.ai/docs/commands/)
- [Plugins](https://opencode.ai/docs/plugins/)
- [Custom Tools](https://opencode.ai/docs/custom-tools/)
- [MCP servers](https://opencode.ai/docs/mcp-servers/)
- [LSP](https://opencode.ai/docs/lsp/)
- [Server](https://opencode.ai/docs/server/)
- [SDK](https://opencode.ai/docs/sdk/)
- [ACP](https://opencode.ai/docs/acp/)
- [GitHub](https://opencode.ai/docs/github/)
- [GitLab](https://opencode.ai/docs/gitlab/)
