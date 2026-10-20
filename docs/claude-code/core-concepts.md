# 核心机制：Claude Code 如何工作

Claude Code 的核心不是“补全一行代码”，而是一个会反复收集上下文、执行动作、验证结果，直到任务完成的 agentic loop。

理解它的上下文、权限和会话机制，比记住更多快捷键更重要。

## Agentic Loop：收集 → 行动 → 验证

一次任务通常会在这三个阶段之间循环：

1. **收集上下文**：读文件、搜索、看 git 状态、读日志、查文档。
2. **行动**：改文件、跑命令、装依赖、执行测试、操作浏览器。
3. **验证**：看测试结果、构建输出、diff、截图，再决定下一步。

循环会一直持续到 Claude 判断完成，或者你中断、纠正它。

这意味着：

- 你给的是目标和约束，不是逐行指令；
- 能验证的任务，结果明显更可靠；
- 你随时可以按 `Esc` 打断并转向。

## 工具：Claude 能“动手”的原因

Claude Code 的能力主要来自五类工具：

| 类别 | 能做什么 |
| --- | --- |
| 文件操作 | 读、写、编辑、创建、重命名文件 |
| 搜索 | 按文件名、内容、正则搜索整个仓库 |
| 执行 | 跑 shell、启动服务、运行测试、使用 git |
| Web | 搜索、抓取文档、查错误信息 |
| 代码智能 | 跳转定义、查引用、看类型错误（需要 code intelligence 插件） |

此外还有子代理、提问、任务列表等编排工具。

## 会话与上下文

### 一个任务一个会话

每次新会话都是全新的上下文窗口，不会自动继承上一次对话。跨会话保留信息靠：

- `CLAUDE.md`：你写的持久规则；
- auto memory：Claude 自动保存的学习；
- `/resume`：恢复旧会话。

不要把无关任务塞进同一个会话。上下文越脏，表现越差。

### 上下文快满时

Claude Code 会自动压缩较早的工具输出和对话。你也可以主动管理：

```text
/context        查看哪些内容占了上下文
/compact        压缩当前会话
/compact 重点保留 API 改动和测试命令
/clear          开始一个全新任务
/btw 这个问题只问一次，不要进入历史
```

!!! tip "重要规则不要只放在聊天里"
    对话会被压缩。构建命令、代码风格、不能改的接口，应该写进 `CLAUDE.md`，而不是只靠某次对话记住。

### 恢复、分叉与命名

```bash
claude --continue          # 继续当前目录最近会话
claude --resume            # 选择历史会话
claude --continue --fork-session  # 从当前会话分叉
```

会话内：

```text
/resume          恢复会话
/branch          从当前点分叉，尝试另一条路线
/rename oauth-migration  给会话起名，方便以后找
```

## 计划模式：先探索，再动手

复杂任务最怕“理解错问题，然后写得很努力”。

计划模式（Plan mode）让 Claude 只读、不改源文件，先收集信息并给出方案。进入方式：

- 按 `Shift+Tab`，直到状态栏显示 plan mode；
- 或启动时指定：

```bash
claude --permission-mode plan
```

推荐的四步：

1. **Explore**：读相关目录、入口、测试和文档。
2. **Plan**：让它给出改哪些文件、怎么验证、有什么风险。
3. **Implement**：确认计划后切出 Plan mode，开始实现。
4. **Commit**：跑测试、看 diff、提交并开 PR。

```text
先不要改代码。阅读 src/auth 和现有测试，给出添加 Google OAuth 的分阶段计划。
每一阶段列出文件、验证命令和回滚方式。
```

!!! tip "小任务不需要计划"
    改文案、重命名变量、加一行日志，直接做。计划模式适合跨模块、需求模糊、你还不熟悉代码的情况。

## 权限模式：控制 Claude 能做什么

按 `Shift+Tab` 可以在权限模式之间切换。常见模式：

| 模式 | 能做什么 | 适合 |
| --- | --- | --- |
| **Auto** | 分类器在后台审查工具调用，拦截高风险动作 | 大多数日常开发（新版默认起点） |
| **Manual** | 改文件和跑命令前询问 | 高风险仓库、初次使用 |
| **Accept edits** | 自动接受文件编辑和常见文件系统命令，其他命令仍询问 | 信任的编辑任务 |
| **Plan** | 只读探索，不改源文件 | 调研、方案设计 |
| **dontAsk** | 只允许预批准的工具，其余拒绝 | 无人值守批处理 |
| **bypassPermissions** | 跳过所有检查 | 仅限隔离环境，风险最高 |

相关命令：

```text
/permissions     管理 allow / ask / deny 规则
/sandbox         开关沙箱
```

沙箱限制文件系统和网络访问，权限规则决定何时询问。两者配合，才能在“少打扰”和“安全”之间取得平衡。

!!! warning "Auto 不是无限制"
    Auto 只是把大部分审批交给分类器，仍然会拦截越权、未知基础设施和可疑内容驱动的操作。不要把 Auto 当成 Full Access。

## 检查点：可以回滚的实验

Claude Code 会在每次提交提示词、开始一轮工作时，对文件做快照。你可以：

- 按 `Esc` 两次，或输入 `/rewind`；
- 恢复代码、恢复对话、两者都恢复；
- 从某个点开始总结，释放上下文。

限制：

- 只跟踪 Claude 文件编辑工具做的改动；
- 通过 Bash 执行的 `rm`、`mv`、`cp` 不会被恢复；
- 子代理的编辑通常不在当前会话检查点里；
- 不是 Git 的替代品。

!!! tip "先提交，再让 agent 冒险"
    重要改动前先 commit。检查点适合快速试错，Git 才是长期可追溯的边界。

## CLAUDE.md：项目级持久规则

`CLAUDE.md` 会在每次会话开始时加载，适合放“永远适用”的内容：

- 构建、测试、lint 命令；
- 代码风格和架构约定；
- 目录结构和重要入口；
- 不能改的接口或文件；
- PR 规范和完成标准。

位置：

| 位置 | 作用 |
| --- | --- |
| `~/.claude/CLAUDE.md` | 个人全局规则 |
| `<repo>/CLAUDE.md` | 团队共享规则 |
| 子目录 `CLAUDE.md` | 更局部的规则 |
| `.claude/rules/*.md` | 按路径或主题拆分的规则 |

在项目里运行：

```text
/init
```

生成初始文件，然后手动精简。

好的 `CLAUDE.md`：

- 短，最好控制在 200 行以内；
- 写 Claude 猜不到的东西；
- 不写“写干净的代码”这种空话；
- 重复犯错两次后再加规则；
- 用 `@path/to/file` 导入更长文档，而不是全文粘贴。

如果仓库里已经有 `AGENTS.md`，Claude Code 也可以在缺少 `CLAUDE.md` 时读取它，方便和其他工具共享规则。具体加载行为以 `/memory` 和官方文档为准。

### Auto memory

Claude Code 还会自动保存一些跨会话的学习，例如你的偏好和项目习惯。它不能替代 `CLAUDE.md`：

- 必须始终生效的规则：放 `CLAUDE.md`；
- 可以自动积累的个人偏好：交给 auto memory；
- 分享 `~/.claude` 前先检查 memory 内容。

## 设置层级

| 层级 | 位置 | 用途 |
| --- | --- | --- |
| 用户 | `~/.claude/settings.json` | 个人默认、网关、模型 |
| 项目共享 | `.claude/settings.json` | 团队权限、hooks、MCP |
| 项目本地 | `.claude/settings.local.json` | 个人项目覆盖 |
| 托管 | managed settings / MDM | 组织强制策略 |

项目共享文件可以提交，但不要放 API Key。网关密钥放用户级设置或环境变量。

## Goal：让 Claude 持续追一个完成条件

`/goal` 用来设置一个可检查的完成条件。每轮结束后，会有一个独立评估模型判断条件是否满足；没满足就继续下一轮。

```text
/goal test/auth 下的测试全部通过，lint 没有错误
```

适合：

- 长任务；
- 需要多轮修复直到测试通过；
- 无人值守目标明确的场景。

不适合：

- 目标本身还很模糊；
- 需要频繁人工判断审美或产品取舍；
- 没有可验证条件。

`/goal` 不会改变权限模式。要让它无人值守地跑，通常需要配合 Auto mode。

## 一个高质量任务的写法

至少写清：

1. **目标**：要改变什么；
2. **上下文**：文件、错误、复现步骤、设计稿；
3. **约束**：不能改什么、必须遵守什么；
4. **完成标准**：测试、构建、行为；
5. **验证方式**：让 Claude 能自己检查结果。

示例：

```text
目标：修复设置页开关保存后刷新复位的问题。

上下文：
- 页面在 src/pages/Settings.tsx
- 保存在 src/api/settings.ts
- 复现：npm run dev，进入 /settings，打开 Enable alerts，保存，刷新后复位

约束：
- 不改变 API 形状
- 保持现有 UI 风格
- 先写一个能复现问题的失败测试

完成标准：
- 复现步骤不再出现
- 测试、lint、类型检查通过
- 最后给出修改文件和验证命令
```

## 官方参考

- [How Claude Code works](https://code.claude.com/docs/en/how-claude-code-works)
- [Best practices](https://code.claude.com/docs/en/best-practices)
- [Permissions](https://code.claude.com/docs/en/permission-modes)
- [Sandboxing](https://code.claude.com/docs/en/sandboxing)
- [Checkpointing](https://code.claude.com/docs/en/checkpointing)
- [Memory / CLAUDE.md](https://code.claude.com/docs/en/memory)
- [Goal](https://code.claude.com/docs/en/goal)
