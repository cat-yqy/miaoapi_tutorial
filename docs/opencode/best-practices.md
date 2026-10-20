# 最佳实践：把 OpenCode 用成可靠的队友

OpenCode 的限制和优势都很鲜明：模型自由、扩展灵活、server 可编程；但默认权限偏宽松，MCP 和子代理会快速消耗上下文，undo 依赖 Git。

下面这些原则，比“万能提示词”更重要。

## 十条最重要的原则

### 1. 给 Agent 一个能自己跑的验证

测试、构建、lint、类型检查、截图对比，都是验证。没有验证时，“看起来完成”就是唯一信号。

```text
实现 validateEmail。用例：
- user@example.com -> true
- invalid -> false
- user@.com -> false
实现后运行测试，失败就继续修。
```

### 2. 复杂任务先 Plan，再 Build

OpenCode 内置 Plan/Build 两个主 agent：

- 按 `Tab` 切到 Plan；
- 或 `opencode --agent plan`。

Plan 适合跨模块、需求模糊、你不熟悉代码的情况。小修小改直接 Build，不必为一行改动写计划。

### 3. 用 `AGENTS.md` 固化长期规则

构建命令、测试命令、代码风格、目录结构、不能改的接口，放 `AGENTS.md`。

- 项目根 `AGENTS.md`：团队共享；
- `~/.config/opencode/AGENTS.md`：个人全局；
- `CLAUDE.md`：Claude Code 兼容 fallback；
- `instructions`：引用额外规则文件。

重复犯错两次，再补规则。不要写百科。

### 4. 默认权限是宽松的，先收紧

OpenCode 大多数权限默认 `allow`，`.env` 默认拒绝，`external_directory` 和 `doom_loop` 默认 `ask`。

在敏感仓库：

```json
{
  "permission": {
    "*": "ask",
    "shell": {
      "*": "ask",
      "git *": "allow",
      "npm *": "allow",
      "rm *": "deny"
    }
  }
}
```

`--auto` 只自动批准 `ask`，显式 `deny` 仍然生效，但不要把 `--auto` 当成默认。

### 5. 一个任务一个 Session

- 新任务：`/new`；
- 长会话：`/compact`；
- 恢复：`/sessions`；
- 分叉：`--fork`；
- 并行：多个 session 或 Git worktree。

上下文越脏，表现越差。不要把项目初始化、三个月后的重构和今天的 bug 都塞进一个 session。

### 6. 研究交给子代理，写操作保持克制

- `@explore`：只读代码库探索；
- `@scout`：只读外部文档/依赖；
- `@general`：多步任务，可写文件。

子代理适合隔离噪声，但每个都独立消耗 token。多个 agent 同时写同一批文件通常不划算。

### 7. 重复动作做成 Command，重复知识做成 Skill

- **Command**：显式 `/test`、`/deploy`，固定动作；
- **Skill**：模型可按描述自动加载的知识/流程；
- **Custom Tool**：需要真实执行逻辑；
- **Plugin**：需要订阅生命周期事件。

判断标准：

- 只是提示词 → Command；
- 需要自动选择 → Skill；
- 需要执行代码 → Custom Tool；
- 需要改 OpenCode 行为 → Plugin。

### 8. MCP 只接真实需要的

MCP 工具会进入上下文。GitHub 等大型 MCP 很容易吃掉大量 token。

原则：

- 只启用当前任务需要的服务器；
- 优先用 CLI 工具（`gh`、`aws`、`sentry-cli`）处理外部系统；
- 对大型 MCP 设置超时和工具范围；
- 不要把“工具多”当成“能力强”。

### 9. LSP 不是越多越好

LSP 默认关闭。它能提供诊断和符号导航，但可能占内存、变慢、和项目版本不同步。

很多项目让 agent 直接跑 `tsc`、`eslint`、`pytest`、`cargo check` 更可控。把这些命令写进 `AGENTS.md`。

### 10. 自动化和平台化放在最后

顺序：

1. 单会话 Plan/Build；
2. `AGENTS.md`；
3. Commands / Skills；
4. MCP / Custom Tools；
5. 子代理；
6. `opencode run` / GitHub Actions；
7. Server / SDK。

流程不稳定时，不要急着做 server 或 CI。

## 按任务选择方式

| 任务 | 建议 |
| --- | --- |
| 改文案、重命名 | 直接 Build |
| 单文件 bug | 复现 → 失败测试 → 修复 |
| 普通功能 | Plan + 垂直切片 |
| 跨模块重构 | Plan + Git worktree + 分阶段验证 |
| 大范围调研 | `@explore` 子代理 |
| 重复动作 | Command |
| 可复用知识 | Skill |
| 需要执行逻辑 | Custom Tool |
| 多仓库/长任务 | 多 session + Git worktree |
| CI/自动化 | `opencode run` / GitHub Actions |

## 上下文管理

OpenCode 没有 Claude Code 的 `/context` 可视化，需要更主动：

- 一个任务一个 session；
- `/compact` 压缩长会话；
- `/new` 换任务；
- 大范围探索交给 Explore；
- 不启用无关 MCP；
- 长日志先截取关键部分。

### 什么时候开新会话

- 换到不相关任务；
- 同一问题已经纠正两次；
- 会话里堆了大量失败尝试；
- 需要重新按计划实现。

## 权限与安全

### 权限矩阵

| 场景 | 建议 |
| --- | --- |
| 读代码、调研 | Plan + `shell: ask` |
| 日常改代码 | `*: ask` + 常用命令 allow |
| 信任的批量编辑 | 明确 allowlist |
| 无人值守 | 最小 allowlist，谨慎 `--auto` |
| CI | secret + 最小 runner 权限 |

### 外部内容的信任边界

以下内容可能包含 prompt injection：

- 网页和搜索结果；
- MCP 返回值；
- 插件；
- PR/Issue/commit message；
- 粘贴的日志。

OpenCode 应把外部内容当数据，不是指令。自动化时尤其不要让不可信文本驱动高权限任务。

### 密钥与隐私

- 不要把 API Key 写进项目 `opencode.json`；
- 用环境变量或 `opencode auth`；
- `opencode web` 默认无认证，必须设 `OPENCODE_SERVER_PASSWORD`；
- 共享 `~/.config/opencode` 前检查 auth、插件和 MCP 配置；
- 插件和 Custom Tools 在你的权限下运行，只装可信来源。

## 模型与成本

### 模型选择

- 复杂任务用更强模型；
- 探索、扫描、轻量任务用快模型；
- 用 variant 调整推理强度；
- 不要所有任务都用最高档；
- 用 `opencode stats` 看实际消耗。

### 成本控制

- 子代理独立消耗 token；
- MCP 工具定义占上下文；
- 长会话压缩或重开；
- 大范围扫描先缩小范围；
- 不要用 `--auto` 跑不信任的任务；
- 多 session 并行会成倍消耗额度。

## 常见反模式

| 反模式 | 后果 | 更好的做法 |
| --- | --- | --- |
| 一个 session 干所有事 | 上下文污染 | 一个任务一个 session |
| 默认 `*: allow` | 误删、越权 | 先 `ask`，再逐项 allow |
| 不跑测试就接受 | 表面正确、边界崩溃 | 先写失败测试 |
| 多个 agent 改同一批文件 | 冲突、覆盖 | 读写分离或用 Git worktree |
| MCP 全开 | 上下文爆炸 | 只接当前需要的 |
| LSP 全开 | 变慢、误报 | 用 CLI 诊断或按需开启 |
| 流程没稳定就上 CI | 无人值守地重复犯错 | 先 Command/Skill，再自动化 |
| 把密钥写进配置 | 泄露 | 环境变量 / auth |
| 用 `opencode web` 裸奔 | 局域网/公网可访问 | 设密码，限制 host |
| 盲目相信 agent 审查 | 漏掉真实风险 | 看证据、行号、触发条件 |

## 团队落地

### 从一条真实工作流开始

- PR 审查；
- 新人代码库导览；
- 发布说明；
- CI 失败排查；
- 测试补全。

先手动跑通，再固化。

### 团队资产分层

| 资产 | 位置 | 作用 |
| --- | --- | --- |
| 项目规范 | `AGENTS.md` | 构建、测试、风格、禁区 |
| 固定动作 | `.opencode/commands/` | `/test`、`/deploy` |
| 可复用知识 | `.opencode/skills/` | 流程和参考材料 |
| 专门 agent | `.opencode/agents/` | 审查、调试、数据任务 |
| 外部系统 | MCP / Custom Tools | 数据库、Issue、监控 |
| 自动触发 | GitHub Actions / GitLab CI | PR 审查、巡检 |

### 度量效果

- PR 从提交到合并的时间；
- 审查发现的问题类型；
- CI 失败恢复时间；
- 重复劳动减少了多少；
- 返工和回滚是否增加。

如果 diff 变大、review 变多、返工变多，先检查任务拆分、`AGENTS.md` 和权限，而不是简单换更强模型。

## 提交前检查清单

- [ ] 任务目标能用一句话说清
- [ ] 相关文件、错误和复现步骤已提供
- [ ] 约束和“不要做什么”已写明
- [ ] 完成标准可验证
- [ ] 复杂任务先看过 Plan
- [ ] 测试/构建/lint 真的跑过
- [ ] diff 看过
- [ ] 没有密钥、日志、调试代码
- [ ] 文档、类型、配置同步更新
- [ ] 需要人工确认的点已列出

## 官方参考

- [Agents](https://opencode.ai/docs/agents/)
- [Models](https://opencode.ai/docs/models/)
- [Permissions](https://opencode.ai/docs/permissions/)
- [Rules](https://opencode.ai/docs/rules/)
- [Commands](https://opencode.ai/docs/commands/)
- [Skills](https://opencode.ai/docs/skills/)
- [Plugins](https://opencode.ai/docs/plugins/)
- [MCP servers](https://opencode.ai/docs/mcp-servers/)
- [LSP](https://opencode.ai/docs/lsp/)
- [CLI](https://opencode.ai/docs/cli/)
