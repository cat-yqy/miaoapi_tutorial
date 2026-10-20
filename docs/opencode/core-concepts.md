# 核心机制：OpenCode 如何工作

OpenCode 的核心是一个 client/server 架构下的编码 agent。理解它如何组织会话、agent、上下文和权限，比记住更多快捷键更重要。

## Client / Server 架构

运行 `opencode` 时，终端 TUI 会连接一个本地 OpenCode server。server 负责模型调用、工具执行、会话状态和事件流，TUI 只是其中一个客户端。

这带来几个直接结果：

- Web、IDE、SDK 可以连接同一个 server；
- `opencode serve` 可以单独启动无头 server；
- `opencode attach` 可以把终端接到已有 server；
- 一个会话可以在终端和浏览器之间切换；
- 可以通过 HTTP/OpenAPI 或 SDK 驱动 OpenCode。

```bash
# 启动无头 server
opencode serve --port 4096

# 另一个终端接入
opencode attach http://localhost:4096
```

server 的 OpenAPI 文档默认在：

```text
http://localhost:4096/doc
```

## Session：工作单元

每次对话是一个 session。OpenCode 会把会话保存到本地，可以恢复、分叉、导出。

```bash
opencode -c                 # 继续最近会话
opencode -s <session-id>    # 继续指定会话
opencode --fork             # 从会话分叉
opencode session list       # 列出会话
```

TUI 内：

```text
/sessions      会话列表，别名 /resume、/continue
/new           新会话，别名 /clear
/export        导出为 Markdown
```

### Undo / Redo 依赖 Git

```text
/undo     撤销上一轮对话和文件改动
/redo     重做
```

它们内部使用 Git 管理文件变化，所以项目**必须是 Git 仓库**。这不是通用备份机制；重要改动仍然应该 commit。

## Agent：Build、Plan 和子代理

### 主 Agent

OpenCode 内置两个 primary agent：

| Agent | 用途 | 工具 |
| --- | --- | --- |
| **Build** | 默认开发 agent | 全部工具 |
| **Plan** | 分析、规划、不改代码 | 默认拒绝编辑，shell 仍受权限控制 |

用 `Tab` 在 primary agent 之间切换。

!!! note "Plan 不是绝对只读"
    Plan 默认拒绝文件编辑，但它并不自动禁止所有 shell 命令。要严格只读，需要在 `permission` 里把 `shell` 设为 `ask` 或 `deny`。

### 子代理

内置子代理：

| Agent | 用途 | 特点 |
| --- | --- | --- |
| **General** | 复杂多步任务 | 完整工具，可修改文件 |
| **Explore** | 代码库探索 | 只读，快速搜索 |
| **Scout** | 外部文档和依赖研究 | 只读，可克隆依赖到缓存 |

调用方式：

- 主 agent 根据描述自动委派；
- 手动 `@general`、`@explore`、`@scout`；
- 子会话创建后，用 leader 快捷键在父子会话之间切换。

子代理适合并行研究和隔离噪声，但每个子代理都独立消耗 token。

## Model 与 Variant

OpenCode 不绑定模型厂商。模型 ID 的格式是：

```text
provider/model
```

例如：

```text
miaoapi/gpt-6.1-sol
```

常用命令：

```bash
opencode models                 # 列出可用模型
opencode models --refresh       # 刷新模型缓存
```

TUI 内：

```text
/models        选择模型
ctrl+t         循环模型 variant
```

Variant 是同一模型的不同推理配置，例如 `low`、`high`、`max` 或 provider 自定义的档位。它们适合在不切换模型的情况下调整速度、成本和质量。

## Permission：默认比你想象的宽松

OpenCode 的权限规则有三种结果：

- `allow`：直接执行；
- `ask`：询问；
- `deny`：阻止。

默认值偏宽松：

- 大多数工具是 `allow`；
- `.env` 文件默认拒绝读取；
- `external_directory` 默认 `ask`；
- `doom_loop`（同一工具调用重复 3 次）默认 `ask`。

```json
{
  "permission": {
    "*": "ask",
    "shell": {
      "*": "ask",
      "git *": "allow",
      "npm *": "allow",
      "rm *": "deny"
    },
    "edit": "ask"
  }
}
```

规则按顺序匹配，**最后一条匹配的规则生效**。通常把 `*` 放前面，具体规则放后面。

### Auto 模式

```bash
opencode --auto
opencode run --auto "重构这个模块"
```

Auto 只自动批准那些本来会 `ask` 的请求；显式 `deny` 仍然生效。

!!! warning "默认宽松，不等于不需要审查"
    如果你在敏感仓库工作，先把 `*` 改成 `ask`，再逐项 allow。不要把 `--auto` 当成默认工作方式。

### 按 Agent 配置权限

可以为某个 agent 单独设置权限，例如让 Plan 更严格：

```json
{
  "agent": {
    "plan": {
      "permission": {
        "edit": {
          "*": "deny",
          "~/.opencode/plan/*": "allow"
        },
        "shell": "ask"
      }
    }
  }
}
```

## Rules：AGENTS.md

OpenCode 用 `AGENTS.md` 保存项目规则：

- 项目根目录 `AGENTS.md`：团队共享；
- `~/.config/opencode/AGENTS.md`：个人全局；
- 子目录 `AGENTS.md`：更局部的规则；
- `CLAUDE.md`：Claude Code 兼容 fallback，没有 `AGENTS.md` 时使用；
- `opencode.json` 的 `instructions`：引用额外的 Markdown 或远程规则。

运行：

```text
/init
```

会根据仓库生成或更新 `AGENTS.md`。建议提交到 Git。

适合写：

- 构建、测试、lint 命令；
- 目录结构和架构约定；
- 代码风格；
- 不能改的接口；
- 完成标准。

不要把 `AGENTS.md` 写成百科。只写 OpenCode 猜不到、而且每次都应该遵守的内容。

## 配置层级

OpenCode 的配置会**合并**，不是互相替换：

| 层级 | 位置 | 用途 |
| --- | --- | --- |
| 远程 | `.well-known/opencode` | 组织默认值 |
| 全局 | `~/.config/opencode/opencode.json` | 个人 provider、模型、权限 |
| 自定义 | `OPENCODE_CONFIG` | 特殊环境覆盖 |
| 项目 | 项目根目录 `opencode.json` | 团队共享设置 |
| 内联 | `OPENCODE_CONFIG_CONTENT` | 运行时覆盖 |
| 托管 | 系统 managed config | 组织强制策略 |

后面的配置会覆盖前面的冲突键，但不会丢掉未冲突的设置。

TUI 另有 `tui.json`，控制主题、键位、滚动、通知和声音，与运行时的 `opencode.json` 分开。

## LSP 与 Formatter

OpenCode 支持 LSP 诊断，但 **LSP 默认关闭**。启用：

```json
{
  "lsp": true
}
```

LSP 能提供类型错误、跳转定义和引用，但语言服务器可能占内存、变慢或和项目版本不同步。很多项目直接让 agent 跑 `tsc`、`eslint`、`pytest` 更稳定。

Formatter 可以在文件修改后自动格式化。方便，但也会扩大 diff；团队要统一配置。

## 上下文与会话管理

OpenCode 不像 Claude Code 那样有 `/context` 可视化命令；你需要主动管理：

- 一个任务一个 session；
- 长会话用 `/compact`；
- 换任务用 `/new`；
- 大范围调研交给 Explore 子代理；
- 不让无关 MCP 工具占满上下文。

MCP 工具会进入上下文；GitHub 等大型 MCP 尤其容易消耗大量 token。只启用当前任务需要的服务器。

## 一个高质量任务的结构

```text
目标：给 list 命令增加 --json 参数。

上下文：
- 入口在 src/cli/list.ts
- 格式化在 src/formatters/list.ts
- 测试参考 test/cli/list.test.ts

约束：
- 不改变默认输出
- 不新增运行时依赖
- 保持退出码语义

完成标准：
- list --json 输出合法 JSON
- 覆盖空列表、正常列表、错误情况
- lint、类型检查、测试通过
```

先按 `Tab` 进入 Plan，确认方案后再切回 Build 实现。

## 官方参考

- [Intro](https://opencode.ai/docs/)
- [TUI](https://opencode.ai/docs/tui/)
- [Agents](https://opencode.ai/docs/agents/)
- [Models](https://opencode.ai/docs/models/)
- [Permissions](https://opencode.ai/docs/permissions/)
- [Rules](https://opencode.ai/docs/rules/)
- [Config](https://opencode.ai/docs/config/)
- [Server](https://opencode.ai/docs/server/)
