# 日常使用：从打开终端到提交代码

这一页不讲安装和 API 配置，只讲装好之后每天怎么用。重点是 CLI，因为它是所有工作流的基线；IDE、桌面端和 Cloud 的差异会单独说明。

## 启动与恢复

```bash
# 在当前目录启动交互式 TUI
codex

# 带一个初始任务启动
codex "解释这个仓库的请求链路"

# 指定工作目录和额外可写目录
codex --cd apps/frontend --add-dir ../shared

# 指定模型
codex -m <model-name>
```

恢复之前的会话：

```bash
codex resume                 # 打开最近会话选择器
codex resume --last          # 直接恢复当前目录的最近一次
codex resume --all           # 显示所有目录的会话
codex resume <SESSION_ID>    # 恢复指定会话
```

非交互模式也可以恢复：

```bash
codex exec resume --last "继续修复刚才发现的竞态问题"
```

## TUI 里最值得记住的快捷键

| 操作 | 作用 |
| --- | --- |
| 输入 `@` | 模糊搜索并插入文件路径 |
| 行首输入 `!` | 直接执行本地 shell 命令，输出会进入上下文 |
| `Tab` | 把跟进内容排到下一轮，不打断当前任务 |
| `Esc` `Esc` | 回退并编辑之前的提示，可从某一点分叉 |
| `Ctrl + G` | 用 `$VISUAL` / `$EDITOR` 打开完整编辑器写长提示 |
| `Ctrl + O` | 复制最近一次完成的输出 |
| `Ctrl + L` | 清屏，但不新开会话 |
| `Ctrl + C` | 中断当前任务 |
| `/exit` | 退出 |

!!! tip "运行中也能补充要求"
    任务正在跑时，按 `Enter` 可以把新指令注入当前轮；按 `Tab` 则排到下一轮。前者适合纠正方向，后者适合“做完这个再做那个”。

## 常用 Slash Commands

在输入框输入 `/` 会打开命令列表。不同版本会有差异，以实际列表为准。

| 命令 | 用途 | 典型场景 |
| --- | --- | --- |
| `/plan` | 进入计划模式 | 需求模糊、跨模块改动 |
| `/goal` | 设置持续目标 | 长任务、多阶段迁移 |
| `/model` | 切换模型和推理强度 | 任务变难或变简单 |
| `/fast` | 切换 Fast 档 | 模型支持时追求速度 |
| `/permissions` | 调整权限模式 | 从只读切到可写，或临时收紧 |
| `/review` | 审查工作区改动 | 提交前第二双眼睛 |
| `/diff` | 查看 Git diff | 确认改了哪些文件 |
| `/compact` | 压缩上下文 | 长会话快满时 |
| `/new` | 新开对话 | 切换到独立任务 |
| `/resume` | 恢复历史会话 | 继续之前的工作 |
| `/fork` | 分叉当前对话 | 尝试另一条实现路线 |
| `/side` | 临时侧聊 | 问个问题但不污染主线 |
| `/worktree` | 在新 worktree 中运行 | 并行任务，隔离改动 |
| `/agent` | 切换子代理线程 | 查看某个子代理的进展 |
| `/agents` | 查看 agent 总览 | 新版 CLI 的并行任务管理 |
| `/skills` | 选择技能 | 复用固定工作流 |
| `/mcp` | 查看 MCP 工具 | 确认外部工具是否可用 |
| `/plugins` | 浏览插件 | 安装团队或官方插件 |
| `/init` | 生成 `AGENTS.md` | 新仓库初始化 |
| `/memories` | 管理记忆 | 控制是否读写长期上下文 |
| `/status` | 查看会话状态 | 确认模型、权限、剩余上下文 |
| `/usage` | 查看额度使用 | 判断是否接近限额 |

## 提交前的标准动作：看 diff + 审查

### 1. 看 diff

```text
/diff
```

它会显示 Git diff，包括尚未被 Git 跟踪的文件。先自己扫一遍，重点看：

- 有没有不该改的文件；
- 有没有调试代码、密钥、日志；
- 改动范围是否超出任务；
- 测试是否真的覆盖了行为变化。

### 2. 运行审查

```text
/review
```

`/review` 会启动一个专门的审查 agent，读取你选择的 diff，给出按优先级排序的问题，并且**不会修改工作区**。常用预设：

- 相对某个 base branch 审查；
- 审查未提交改动；
- 审查某个 commit；
- 输入自定义审查要求，例如：

```text
/review 重点看并发、错误处理和回归风险
```

审查发现的问题可以交给主 agent 修，修完再跑一次 `/review`。

!!! note "审查不是验收"
    Codex 的审查能发现不少问题，但它不能替代测试、类型检查、CI 和人工判断。把它当作“提交前的额外一层”，不是“可以闭眼合并”。

### 3. GitHub 上的审查

如果仓库接入了 Codex Code Review，可以在 PR 里评论：

```text
@codex review
```

也可以指定重点：

```text
@codex review 重点检查权限校验和错误分支
```

新版桌面端还提供 PR 审查面板，能同时看描述、diff、评论和检查结果。GitHub 审查已正式可用，GitLab MR 支持处于预览阶段。具体是否可用取决于账号和工作区配置。

## IDE 扩展：把“当前看到的东西”直接交给 Codex

IDE 扩展的核心优势是**自动上下文**：

- 当前打开的文件；
- 当前选中的代码范围；
- 通过 `@file` 引用的文件；
- 编辑器里的诊断和终端输出（取决于版本和设置）。

适合：

- 边读边问：“这个函数被谁调用？”
- 选中一段代码写测试；
- 在编辑器里看 diff、逐条评论；
- 把任务委托到 Cloud，再在本地应用结果。

使用时注意：自动上下文是双刃剑。打开太多无关文件会稀释注意力，建议只保留当前任务相关的标签页。

## Codex app：多任务工作台

桌面端最适合“同时推进几件事”：

- **Local**：直接在当前项目目录工作；
- **Worktree**：创建独立 Git worktree，改动与主工作区隔离；
- **Cloud**：在云端隔离环境运行。

其他实用能力：

- 内置 diff 面板，可以按行评论，让 Codex 针对具体改动修改；
- 内置终端，能跑测试和 Git 命令；
- 内置浏览器，可预览本地页面并直接在页面上评论；
- Computer Use，可操作桌面应用，适合 GUI-only 的复现和测试；
- 语音输入、浮窗、任务侧栏和产物预览；
- Automations，把稳定流程按计划跑起来。

!!! warning "Computer Use 风险更高"
    它能操作项目之外的应用和系统状态。只做窄任务，仔细看权限提示；不要把“操作整个电脑”的权限随手交给一个模糊任务。

## Codex Cloud：把长任务放到后台

适合：

- 任务会跑很久；
- 你想同时开多个方向；
- 你要离开电脑；
- 你希望直接产出 PR。

基本流程：

1. 在 Cloud 里连接 GitHub 仓库；
2. 配置环境（依赖、启动命令、网络访问）；
3. 选择仓库和分支，提交任务；
4. 查看进度和 diff；
5. 直接在云端开 PR，或把改动拉回本地继续。

Cloud 的关键不是“更聪明”，而是“并行”和“不占本地”。环境配置好之后，任务之间的隔离和复用会省很多时间。使用第三方 API 时，Cloud 可能不可用，先确认账号能力。

## 一个 10 分钟的标准工作流

假设要给 CLI 增加一个 `--json` 输出选项：

```text
目标：给 `list` 命令增加 `--json` 参数，输出机器可读的 JSON。

上下文：
- 命令入口在 src/cli/list.ts
- 现有输出格式在 src/formatters/list.ts
- 参考 test/cli/list.test.ts 的写法

约束：
- 不改变默认的人类可读输出
- 不新增运行时依赖
- 保持现有退出码语义

完成标准：
- `list --json` 输出合法 JSON
- 新增测试覆盖空列表、正常列表和错误情况
- lint、类型检查和相关测试通过

先读代码并给出简短计划，确认后再改。
```

接下来：

1. 看计划，确认没有理解偏差；
2. 让 Codex 实现并运行测试；
3. `/diff` 检查改动；
4. `/review` 做一次审查；
5. 修正后提交。

这个循环适用于大多数中小任务。复杂任务把“计划”和“实现”拆成两轮，长任务再考虑 Goal 或 Cloud。

## 官方参考

- [Codex CLI features](https://developers.openai.com/codex/cli/features)
- [Slash commands](https://developers.openai.com/codex/cli/slash-commands)
- [Codex IDE extension](https://developers.openai.com/codex/ide)
- [Codex app features](https://developers.openai.com/codex/app/features)
- [Codex Cloud](https://developers.openai.com/codex/cloud)
- [Workflows](https://developers.openai.com/codex/workflows)
