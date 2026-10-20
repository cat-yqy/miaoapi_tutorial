# 核心机制：Codex 是如何工作的

理解 Codex 的工作方式，比记住更多命令更重要。Codex 不是“输入一句话，直接吐出最终代码”的补全工具，而是一个会反复读取、修改、执行和验证的 agent。

## 一次任务的基本循环

一次典型的 Codex 任务会经历这样的循环：

1. 读取你的提示词和上下文。
2. 制定计划，决定下一步读哪个文件、跑什么命令。
3. 调用工具：读文件、搜索、编辑、执行测试。
4. 根据工具输出修正判断。
5. 重复以上步骤，直到满足完成条件。
6. 输出总结、diff 和验证结果。

这意味着两件事：

- **你给的是目标和约束，不是逐行指令。** 不需要告诉它每一行怎么改，但要说清楚“什么算完成”。
- **验证必须进入循环。** 如果 Codex 不能跑测试、看日志或复现问题，它就只能“看起来正确”。

## Thread：一个任务一个线程

Thread 是一次完整的工作会话，包含你的提示、模型的回复、工具调用和中间结果。一个 thread 里可以有多轮对话。

常见操作：

| 操作 | 作用 | 什么时候用 |
| --- | --- | --- |
| `/new` | 在同一终端里开新对话 | 换一个独立任务 |
| `/resume` | 恢复之前的会话 | 昨天没做完，今天继续 |
| `/fork` | 从当前对话分叉 | 想尝试另一条路线，但不想丢掉原记录 |
| `/side`（别名 `/btw`） | 开一个临时侧聊 | 只想问个问题，不想污染主任务上下文 |
| `/compact` | 压缩早期上下文 | 长任务上下文快满时 |
| `/worktree` | 在独立 Git worktree 中运行 | 并行任务，避免改同一份工作区 |
| `/agent` | 切换子代理线程 | 查看或继续某个子代理的工作 |
| `/agents` | 查看 agent 总览 | 新版 CLI 中管理并行任务 |

### 不要一个项目只开一个线程

这是最常见、也最影响效果的习惯问题。一个 thread 从项目初始化聊到三个月后的重构，早期无关信息会不断挤占上下文，导致：

- 重要约束被埋没；
- 模型开始遗忘之前的决定；
- 无关的工具输出越来越多；
- 回答变慢、变贵，而且更容易跑偏。

**推荐做法：一个连贯任务一个 thread。** 任务真正分叉时用 `/fork`，不要把所有事情塞进同一个会话。

## 上下文与压缩

模型能处理的内容有上限。Codex 会监控剩余上下文，并在长任务中自动压缩较早的内容。你也可以手动 `/compact`。

但压缩不是无损的。被摘要掉的信息可能包括：

- 你早期说过的边界条件；
- 已经排除的方案；
- 某个失败尝试的原因；
- 团队约定。

所以重要的规则不要只靠对话传递。把它们写进 `AGENTS.md`、`README` 或任务文件，让 Codex 每次都能重新读到。

## 先计划，再执行：Plan 与 Goal

### `/plan`：把模糊任务变清楚

适合：

- 需求还不明确；
- 改动跨多个模块；
- 需要先做技术选型；
- 你不想让 Codex 一上来就改代码。

用法：

```text
/plan 把支付模块从回调模式迁移到轮询模式。先不要改代码，给出分阶段计划、风险点和回滚方案。
```

Plan 模式会先收集上下文、提出方案，必要时反问你。计划确认后再进入实现。

### `/goal`：给长任务一个持续目标

Goal 模式适合需要多步、长时间推进的任务。目标本身既是起始提示，也是完成标准。

```text
/goal 把首页首屏加载时间降到 1 秒以内，保持现有功能和测试全部通过。
```

常用操作：

```text
/goal              查看当前目标
/goal pause        暂停
/goal resume       继续
/goal clear        清除
```

如果 `/goal` 不存在，可能需要在配置里启用：

```toml
[features]
goals = true
```

也可以运行：

```bash
codex features enable goals
```

!!! tip "目标要可验证"
    不要写“优化一下性能”。写成“LCP 低于 1.5 秒，现有测试通过，不新增运行时依赖”。Codex 才能判断什么时候算完成。

### 什么时候不需要计划

- 改一个文案；
- 修一个明显的拼写错误；
- 已经有明确复现步骤和修复方案的小 bug；
- 你只想问一个代码问题。

小任务直接做，反复确认计划反而浪费时间。

## 模型与推理强度

在 CLI 里用 `/model` 切换模型和推理强度；在 IDE 和 app 里有对应的模型选择器。

推理强度一般分为低、中、高，部分模型还支持更高档位。选择原则：

| 任务 | 建议 |
| --- | --- |
| 改文案、简单重命名、格式化 | 低推理，快模型 |
| 普通功能开发、单文件 bug | 中推理 |
| 跨模块重构、复杂调试、架构设计 | 高推理 |
| 长链路 agent 任务、疑难问题 | 最高可用档位 |

!!! warning "更强不等于更适合"
    高推理会增加等待时间和 token 消耗。一个清晰的提示词 + 中推理，通常比模糊提示 + 最高推理更有效。先用默认档位，遇到明显瓶颈再升。

`/fast` 可以在模型支持时切换 Fast 服务层，用速度换成本或额度。不是所有模型都有 Fast 档，命令不存在就是当前模型不支持。

## 权限与沙箱

Codex 的安全模型由两个独立但配合的机制组成：

- **沙箱（Sandbox）**：技术上限制能读哪些目录、能写哪些文件、能不能联网。
- **审批（Approval）**：当操作要越过沙箱边界时，是否停下来问你。

### 三种常见权限模式

| 模式 | 能做什么 | 适合 |
| --- | --- | --- |
| **Auto / 默认** | 在当前工作区内读、写、跑命令；越界或联网前询问 | 大多数日常开发 |
| **Read-only** | 只读，不改文件、不跑有副作用的命令 | 代码调研、审查、陌生仓库 |
| **Full Access** | 可跨目录、联网，不再逐步询问 | 可信仓库、隔离环境、明确知道风险时 |

对应到 `config.toml`：

```toml
sandbox_mode = "workspace-write"   # read-only | workspace-write | danger-full-access
approval_policy = "on-request"     # untrusted | on-request | never
```

- `workspace-write` + `on-request` 是低摩擦且相对安全的组合。
- `danger-full-access` + `never` 等于完全放权，只应在可信目录或隔离容器中使用。
- 本站的 MiaoAPI 推荐配置为了兼容特定工作流使用了完全放权，如果你更看重安全，可以改回上面的组合。

### 规则（Rules）

如果只是某几个命令需要越界，不要直接开 Full Access。可以用 rules 对命令前缀设置允许、询问或禁止。这样比“全域放开”更细。

## AGENTS.md：把规则变成长期记忆

`AGENTS.md` 是给 agent 看的项目说明书。它会自动进入上下文，适合放“每次都应该遵守”的内容。

一个好的 `AGENTS.md` 通常包含：

- 仓库结构和重要目录；
- 安装、构建、测试、lint 命令；
- 代码风格和架构约定；
- 不允许改动的文件或接口；
- PR 要求；
- “完成”的定义和验证方式。

可以在不同层级放置：

- `~/.codex/AGENTS.md`：个人全局默认；
- 仓库根目录 `AGENTS.md`：团队共享规范；
- 子目录 `AGENTS.md`：更局部、更具体的规则，离当前目录更近的优先。

在仓库里运行：

```text
/init
```

可以生成一个初始 `AGENTS.md`，但生成后一定要手动修改，不要直接提交模板。

!!! tip "短而准，比长而全更有用"
    不要把所有文档都塞进 `AGENTS.md`。只写那些“Codex 不知道就一定会做错”的内容。重复犯同一个错误两次，就把它补进去。

### 代码审查规则

如果团队使用 Codex 做 PR 审查，可以在 `AGENTS.md` 里增加 `## Code Review Rules` 段落，写明重点关注的行为、风险级别和不应由 Codex 判断的机械检查。机械检查交给 CI，Codex 负责需要理解上下文的审查。

## 配置分层

Codex 的配置大致分为三层：

| 层级 | 位置 | 用途 |
| --- | --- | --- |
| 个人默认 | `~/.codex/config.toml` | 模型、权限、MCP、个人偏好 |
| 仓库配置 | `.codex/config.toml` | 团队共享的项目级设置 |
| 一次性覆盖 | 命令行参数、`/model`、`/permissions` | 临时调整，不写回配置 |

Profile 可以理解为配置预设，适合区分“日常开发”“深度审查”“CI”等场景。

不要把 API Key、token、密码写进仓库里的 `.codex/config.toml`。密钥放在环境变量、系统钥匙串或 CI secret 中。

## 一个高质量任务的四要素

写提示词时，至少说清楚这四件事：

1. **Goal**：要改变什么、构建什么。
2. **Context**：相关文件、错误日志、复现步骤、设计稿。
3. **Constraints**：不能改什么、必须遵守什么、用什么技术栈。
4. **Done when**：哪些测试必须过、行为必须怎样、什么算完成。

示例：

```text
目标：修复设置页开关“保存后刷新又恢复”的问题。

上下文：
- 入口在 src/pages/Settings.tsx
- 保存接口在 src/api/settings.ts
- 复现：npm run dev，进入 /settings，打开 Enable alerts，保存，刷新后开关复位

约束：
- 不改变 API 形状
- 保持现有 UI 风格
- 加一个回归测试

完成标准：
- 复现步骤不再出现
- 相关测试和 lint 通过
- 最后给出修改文件和验证命令
```

## 官方参考

- [Prompting](https://developers.openai.com/codex/prompting)
- [Best practices](https://developers.openai.com/codex/learn/best-practices)
- [AGENTS.md](https://developers.openai.com/codex/guides/agents-md)
- [Sandbox](https://developers.openai.com/codex/concepts/sandboxing)
- [Codex CLI features](https://developers.openai.com/codex/cli/features)
- [Slash commands](https://developers.openai.com/codex/cli/slash-commands)
- [Config basics](https://developers.openai.com/codex/config-basic)
