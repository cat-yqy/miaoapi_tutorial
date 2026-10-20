# 最佳实践：把 Claude Code 用成可靠的队友

Claude Code 的上限不只取决于模型，还取决于上下文、验证、权限和会话管理。下面这些原则，比任何“万能提示词”都更重要。

## 十条最重要的原则

### 1. 给 Claude 一个能自己跑的验证

测试、构建、lint、截图对比、脚本断言，都是验证。没有验证时，“看起来完成了”就是唯一信号，你只能人工兜底。

```text
实现 validateEmail 函数。
用例：user@example.com 为 true；invalid 为 false；user@.com 为 false。
实现后运行测试，失败就继续修。
```

### 2. 先探索，再计划，再写代码

复杂任务先用 Plan mode：

- `Shift+Tab` 切到 Plan；
- 或 `claude --permission-mode plan`。

让 Claude 读代码、给方案、列风险和验证步骤，确认后再实现。

小任务不需要计划。改文案、重命名变量、加一行日志，直接做。

### 3. 提供具体上下文

至少说清：

- 目标；
- 相关文件和路径；
- 错误和复现步骤；
- 约束和不能改的东西；
- 完成标准；
- 怎么验证。

用 `@` 引用文件、粘贴截图、把日志 pipe 进来，比长篇描述更有效。

### 4. 把长期规则写进 `CLAUDE.md`

构建命令、测试命令、代码风格、架构约定、PR 规范，放 `CLAUDE.md`，不要每次在对话里重复。

保持短：

- 最好 200 行以内；
- 只写 Claude 猜不到的内容；
- 重复犯错两次后再加规则；
- 长参考资料放 Skill 或 `.claude/rules/`。

### 5. 先收紧权限，再按需放开

默认从 Auto 或 Manual 开始，理解行为后再放宽。Full bypass 只用于隔离环境。

```text
/permissions     管理 allow / ask / deny
/sandbox         开关沙箱
```

不要把密钥写进项目 `.claude/settings.json`，也不要把 `bypassPermissions` 当成日常默认。

### 6. 用检查点做实验，用 Git 做边界

`Esc` `Esc` 或 `/rewind` 可以恢复代码、对话或两者。适合：

- 尝试不同实现；
- 从错误修改中恢复；
- 压缩一段冗长对话。

但检查点只跟踪 Claude 文件编辑工具的改动：

- Bash 执行的 `rm`、`mv`、`cp` 不会被恢复；
- 子代理的编辑通常不在主会话检查点里；
- 外部改动不会自动捕获。

重要改动前先 commit。检查点不是 Git 的替代品。

### 7. 一个任务一个会话

不要把一个会话从项目初始化聊到三个月后。上下文越脏，表现越差。

- 新任务：`/clear`；
- 长会话：`/compact`；
- 侧问题：`/btw`；
- 分叉：`/branch`；
- 并行：`--worktree`。

如果同一个问题纠正了两次还不对，通常不是再解释一次，而是 `/clear` 后用更准确的提示重来。

### 8. 研究交给子代理，写操作保持克制

子代理适合：

- 大范围代码探索；
- 多角度审查；
- 日志和测试分析；
- 并行验证假设。

不适合：

- 多个 agent 同时改同一批文件；
- 强顺序迁移；
- 任务很小、协调成本更高。

子代理有独立上下文和独立 token 消耗。主会话只收到摘要，这是优点，也是成本。

### 9. 稳定流程先做成 Skill，再自动化

顺序：

1. 手动跑通；
2. 把步骤写成 `SKILL.md`；
3. 确认可重复；
4. 再加 `/loop`、定时任务或 CI。

不要把一个还在频繁调整的流程直接设成无人值守任务。

### 10. 用独立审查对抗“自说自话”

实现和审查分开：

- 用 `/code-review` 审查当前 diff；
- 用全新子代理对照 `SPEC.md` 找缺口；
- 用 `/security-review` 做安全专项；
- 用 Writer/Reviewer 双会话模式。

但不要盲目追求“零 findings”。审查者被要求找问题，就一定会找出一些问题；只为正确性和明确需求买单。

## 按任务选择方式

| 任务 | 建议 |
| --- | --- |
| 改文案、重命名、小修 | 直接做，低 effort |
| 单文件 bug | 复现 → 失败测试 → 修复 → 回归 |
| 普通功能 | 简短计划 + 垂直切片 |
| 跨模块重构 | Plan mode + worktree + 分阶段验证 |
| 需求模糊的新功能 | 访谈 → SPEC.md → 新会话实现 |
| 大范围调研 | 子代理 / Agent View |
| 重复流程 | Skill |
| 多仓库/长任务 | Cloud（如可用）或后台会话 |
| 需要协作的复杂任务 | Agent Teams（实验） |
| 定时任务 | `/loop` / 桌面定时 / `/schedule` |

## 上下文管理

上下文是 Claude Code 最关键的资源。上下文越满，早期规则越容易丢失。

### 主动管理

```text
/context        看什么占了空间
/compact        压缩
/clear          换任务
/btw            只问一次，不进历史
```

### 什么时候该开新会话

- 换了一个不相关的任务；
- 同一问题已经纠正两次；
- 会话里堆了大量失败尝试；
- 需要按 `SPEC.md` 重新实现。

### 什么该持久化

| 内容 | 放在哪里 |
| --- | --- |
| 构建/测试命令、代码风格 | `CLAUDE.md` |
| 按路径生效的规则 | `.claude/rules/` |
| 可复用流程 | Skill |
| 外部实时数据 | MCP |
| 必须每次执行的检查 | Hook |

`CLAUDE.md` 是“每次都要知道”的内容，不是知识库。太长会让重要规则被淹没。

## 权限与安全

### 权限矩阵

| 场景 | 建议 |
| --- | --- |
| 读代码、调研 | Plan mode / 只读 |
| 日常改代码 | Auto 或 Manual |
| 信任的批量编辑 | Accept edits |
| 无人值守批处理 | `dontAsk` + 明确 allowlist |
| 隔离容器/一次性环境 | 才考虑 bypass |
| CI | 最小权限 + 保护 secret |

### 外部内容的信任边界

以下内容可能包含 prompt injection：

- 网页和搜索结果；
- MCP 返回值；
- 插件；
- PR/Issue/commit message；
- 粘贴的日志和文档。

Claude 应该把外部内容当数据，不是指令。自动化时尤其要避免把不可信文本直接拼进高权限任务。

### 密钥与隐私

- 不要把 API Key 写进项目配置、`CLAUDE.md`、截图或提交；
- CI 用 secret，并限制触发者；
- 共享 `~/.claude` 前检查会话、memory 和凭据；
- 使用 MCP/插件前审查它能访问什么；
- Computer Use 只给窄任务，逐条看权限提示。

## 并行与成本

### 什么时候值得并行

适合：

- 多个独立 bug；
- 多角度审查；
- 大范围只读调研；
- 不同模块的独立改动；
- 长任务后台跑。

不适合：

- 同文件并行编辑；
- 强依赖的顺序任务；
- 小任务；
- 预算敏感且没有明确收益。

### 成本控制

- 上下文越短，通常越便宜也越准；
- 子代理、Agent View、Agent Teams 都独立消耗额度；
- Agent Teams 的 token 成本明显高于单会话；
- Ultrareview 使用云端额度；
- 图片、长日志、全仓库扫描会快速推高消耗；
- 用 `/usage` 看用量，用 `/context` 看上下文。

**更强模型 + 更高 effort 不是默认最优。清晰任务 + 合适模型 + 验证通常更划算。**

## 常见反模式

| 反模式 | 后果 | 更好的做法 |
| --- | --- | --- |
| 一个会话干所有事 | 上下文污染、越来越笨 | 一个任务一个会话 |
| `CLAUDE.md` 写成百科 | 重要规则被忽略 | 控制在 200 行内，细节放 Skill |
| 不跑测试就接受 | 表面正确、边界崩溃 | 先写失败测试再修 |
| 默认 bypass 权限 | 误删、越权、供应链风险 | Auto/Manual + sandbox |
| 两个 agent 改同一批文件 | 冲突、互相覆盖 | worktree 隔离 |
| 流程没稳定就自动化 | 无人值守地重复犯错 | 先 Skill，再定时 |
| 盲目追零 findings | 过度防御、抽象膨胀 | 只为正确性和需求买单 |
| 把密钥/日志粘进会话 | 泄露 | 脱敏，用 secret |
| 用 Claude Code 跑非 Claude 模型 | 不兼容 | 用 Claude 模型 |
| 把 Cloud/Remote Control 当默认 | 网关模式下不可用 | 先确认账号能力 |

## 团队落地

### 先从一条工作流开始

选团队真正重复的流程：

- PR 审查；
- 新人代码库导览；
- 发布说明；
- CI 失败排查；
- 迁移计划。

先手动跑通，再固化。

### 团队资产分层

| 资产 | 位置 | 作用 |
| --- | --- | --- |
| 项目规范 | `CLAUDE.md` | 构建、测试、风格、禁区 |
| 路径规则 | `.claude/rules/` | 按目录/文件生效 |
| 可复用流程 | `.claude/skills/` | 团队技能 |
| 专门 agent | `.claude/agents/` | 审查、调试、数据任务 |
| 确定性检查 | Hooks | 格式化、lint、阻断危险操作 |
| 外部系统 | MCP / Plugins | 文档、数据库、Issue、监控 |
| 自动触发 | GitHub Actions / 定时任务 | PR 审查、巡检、汇总 |

### 度量效果

观察：

- PR 从提交到合并的时间；
- 审查发现的问题类型；
- CI 失败恢复时间；
- 重复劳动减少了多少；
- 返工和回滚是否增加。

如果 review 变多、diff 变大、返工变多，先检查任务拆分、`CLAUDE.md` 和权限，而不是简单换更强模型。

## 提交前检查清单

- [ ] 任务目标能用一句话说清
- [ ] 相关文件、错误和复现步骤已提供
- [ ] 约束和“不要做什么”已写明
- [ ] 完成标准可验证
- [ ] 复杂任务先看过计划
- [ ] 测试/构建/lint 真的跑过
- [ ] `/diff` 看过
- [ ] `/code-review` 跑过
- [ ] 没有密钥、日志、调试代码
- [ ] 文档、类型、配置同步更新
- [ ] 需要人工确认的点已列出

## 官方参考

- [Best practices](https://code.claude.com/docs/en/best-practices)
- [How Claude Code works](https://code.claude.com/docs/en/how-claude-code-works)
- [Memory / CLAUDE.md](https://code.claude.com/docs/en/memory)
- [Permission modes](https://code.claude.com/docs/en/permission-modes)
- [Sandboxing](https://code.claude.com/docs/en/sandboxing)
- [Checkpointing](https://code.claude.com/docs/en/checkpointing)
- [Subagents](https://code.claude.com/docs/en/sub-agents)
- [Agent teams](https://code.claude.com/docs/en/agent-teams)
- [Costs](https://code.claude.com/docs/en/costs)
