# 日常使用：从打开终端到提交代码

这一页只讲配好 MiaoAPI 之后，日常怎么用 Claude Code。安装和网关配置见 [Claude Code 接入 MiaoAPI](../guide/claude-code.md)。

## 启动与恢复

```bash
claude                     # 在当前目录启动交互会话
claude "解释这个仓库"       # 带初始任务启动
claude -p "总结这次改动"    # 非交互，输出后退出
claude -c                  # 继续最近会话
claude -r                  # 选择历史会话
claude --model <model>     # 指定模型
```

会话内恢复和分叉：

```text
/resume       恢复历史会话
/branch       从当前点分叉
/rename oauth-migration  给会话命名
```

## 最值得记住的交互

| 操作 | 作用 |
| --- | --- |
| `@` | 引用文件或目录，让 Claude 先读再回答 |
| `!` | 直接执行 shell 命令，输出进入上下文 |
| `Esc` | 打断当前动作，保留上下文 |
| `Esc` `Esc` | 打开 rewind 菜单，回滚代码/对话 |
| `Shift+Tab` | 切换权限模式：Auto / Manual / Accept edits / Plan |
| `Ctrl+G` | 用编辑器写长提示或修改计划 |
| `Ctrl+R` | 搜索历史提示 |
| `/btw` | 问一个不进入会话历史的侧问题 |

## 常用 Slash Commands

| 命令 | 用途 |
| --- | --- |
| `/init` | 生成初始 `CLAUDE.md` |
| `/memory` | 查看/编辑记忆和指令文件 |
| `/context` | 查看上下文占用 |
| `/compact` | 压缩当前会话 |
| `/clear` | 开始全新任务，清空上下文 |
| `/rewind` | 回滚代码或对话到检查点 |
| `/model` | 切换模型 |
| `/effort` | 调整推理强度 |
| `/permissions` | 管理 allow / ask / deny 规则 |
| `/sandbox` | 开关沙箱 |
| `/diff` | 查看当前工作区 diff |
| `/code-review` | 审查当前 diff，可用 `--fix` 应用修复 |
| `/review` | `/code-review` 的别名 |
| `/security-review` | 审查 diff 中的安全问题 |
| `/goal` | 设置持续完成条件 |
| `/loop` | 在会话内重复执行提示 |
| `/schedule` | 创建云端定时任务 |
| `/mcp` | 管理 MCP 服务器 |
| `/plugin` | 浏览和安装插件 |
| `/doctor` | 检查安装、配置和 `CLAUDE.md` |
| `/usage` | 查看用量和成本 |
| `/status` | 查看版本、模型、账号和连接状态 |

不同版本和账号下命令列表会有差异；输入 `/` 看实际可用项。

## 标准工作流：探索 → 计划 → 实现 → 验证

### 1. 探索

进入 Plan mode：

```text
/plan
```

或者启动时：

```bash
claude --permission-mode plan
```

```text
阅读 src/auth 和现有测试，说明登录流程、token 刷新和权限校验分别在哪里。
先不要改代码。
```

### 2. 计划

```text
我要增加 Google OAuth。列出需要修改的文件、session 流程、测试点和回滚方案。
```

复杂计划可以按 `Ctrl+G` 在编辑器里直接改。

### 3. 实现

确认计划后切出 Plan mode：

```text
按计划实现。先做 callback handler，再接入 session，最后补测试。
每完成一步运行相关测试。
```

### 4. 验证

```text
运行相关测试、lint 和类型检查。
把命令、输出和失败原因一起贴出来。
```

### 5. 提交

```text
总结改动，提交一个清晰的 commit，并创建 PR。
```

如果安装了 `gh` CLI，Claude 可以直接创建 Issue、PR、读取评论：

```bash
gh auth status
```

## 看 diff 和做审查

### `/diff`

```text
/diff
```

检查：

- 有没有超出任务范围的改动；
- 有没有调试代码、密钥、日志；
- 测试是否覆盖了行为变化；
- 有没有误删文件。

### `/code-review`

```text
/code-review
```

默认审查当前 diff，查找正确性问题。常用参数：

```text
/code-review high              # 提高审查强度
/code-review --fix             # 应用发现的问题
/code-review 1234              # 审查 PR #1234
/code-review --comment         # 把发现发到 PR
/security-review               # 专看安全问题
```

`/review` 是别名。

!!! warning "Ultrareview 依赖 claude.ai"
    `/code-review ultra` 会在 Anthropic 云端跑多 agent 深度审查，需要 claude.ai 登录和用量额度。通过 MiaoAPI 网关使用 Claude Code 时，通常不能用 Ultrareview；本地 `/code-review` 不受影响。

## IDE 扩展

VS Code、Cursor 和 JetBrains 里的 Claude Code 共享同一个引擎，但多了编辑器上下文：

- 当前打开的文件；
- 选中代码范围；
- 内联 diff；
- 计划审阅；
- 会话历史；
- 从 IDE 委托到 Cloud，再拉回结果。

适合：

- 边读边问；
- 选中函数写测试；
- 在编辑器里逐条看 diff；
- 把长任务丢到 Cloud，本地继续改别的。

不要让几十个无关标签页一直开着；自动上下文太多会稀释注意力。

## Desktop app

桌面端适合“同时推进多个任务”：

- 多个会话并排；
- 可视化 diff 和逐行评论；
- 内置浏览器预览本地页面；
- worktree 隔离；
- Computer Use；
- 定时任务；
- 从终端 `/desktop` 把当前会话接到桌面端继续。

如果你在终端里已经有一个会话，可以：

```text
/desktop
```

注意：这个交接需要 claude.ai 订阅，MiaoAPI 网关模式下不一定可用。

## Web 与 Remote Control

### Web

在 [claude.ai/code](https://claude.ai/code) 里可以跑云端会话，适合：

- 本地没有仓库；
- 长时间任务；
- 同时跑多个任务；
- 从手机查看进度。

从终端把任务送到云端：

```bash
claude --cloud
```

把云端会话拉回终端：

```bash
claude --teleport
```

这些能力依赖 claude.ai 账号，第三方网关通常不可用。

### Remote Control

Remote Control 让代码继续在本地执行，你从手机或浏览器控制会话：

```bash
claude remote-control
# 或
claude --remote-control
```

限制很重要：Remote Control 要求 `ANTHROPIC_BASE_URL` 指向 Anthropic 官方地址。使用 MiaoAPI 时，Remote Control 通常不可用。

## 一个 10 分钟的标准任务

假设要给 CLI 增加 `--json`：

```text
目标：给 list 命令增加 --json 参数，输出机器可读 JSON。

上下文：
- 入口在 src/cli/list.ts
- 格式化在 src/formatters/list.ts
- 测试参考 test/cli/list.test.ts

约束：
- 不改变默认的人类可读输出
- 不新增运行时依赖
- 保持现有退出码语义

完成标准：
- list --json 输出合法 JSON
- 覆盖空列表、正常列表和错误情况
- lint、类型检查、相关测试通过

先给简短计划，确认后再改。
```

然后：

1. 看计划；
2. 让它实现并跑测试；
3. `/diff` 看改动；
4. `/code-review` 审查；
5. 修正后提交。

## 官方参考

- [Quickstart](https://code.claude.com/docs/en/quickstart)
- [Common workflows](https://code.claude.com/docs/en/common-workflows)
- [Interactive mode](https://code.claude.com/docs/en/interactive-mode)
- [Commands](https://code.claude.com/docs/en/commands)
- [VS Code](https://code.claude.com/docs/en/vs-code)
- [Desktop](https://code.claude.com/docs/en/desktop)
- [Claude Code on the web](https://code.claude.com/docs/en/claude-code-on-the-web)
- [Remote Control](https://code.claude.com/docs/en/remote-control)
