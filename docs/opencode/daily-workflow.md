# 日常使用：从 TUI 到提交代码

这一页只讲配好 MiaoAPI 之后，OpenCode 每天怎么用。provider 配置见 [OpenCode 接入 MiaoAPI](../guide/opencode.md)。

## 启动

```bash
opencode                     # 当前目录启动 TUI
opencode /path/to/project    # 指定项目目录
```

常用启动参数：

```bash
opencode -c                  # 继续最近会话
opencode -s <session-id>     # 继续指定会话
opencode --fork              # 继续时分叉
opencode -m miaoapi/gpt-6.1-sol
opencode --agent plan
```

## TUI 交互

| 操作 | 作用 |
| --- | --- |
| `@` | 模糊搜索文件，插入路径 |
| `!` | 执行 shell 命令，输出进入对话 |
| `Tab` | 在 Build / Plan 主 agent 之间切换 |
| `ctrl+x` | 默认 leader key，后面接命令快捷键 |
| `ctrl+p` | 命令面板 |
| `ctrl+t` | 循环模型 variant |
| `ctrl+x n` | 新会话（`/new`） |
| `ctrl+x l` | 会话列表（`/sessions`） |
| `ctrl+x u` | 撤销（`/undo`） |
| `ctrl+x r` | 重做（`/redo`） |
| `ctrl+x c` | 压缩上下文（`/compact`） |
| `ctrl+x e` | 用外部编辑器写提示（`/editor`） |
| `ctrl+x q` | 退出 |

## 常用 Slash Commands

| 命令 | 用途 |
| --- | --- |
| `/connect` | 添加 provider 和 API Key |
| `/models` | 选择模型 |
| `/init` | 生成/更新 `AGENTS.md` |
| `/new` | 新会话，别名 `/clear` |
| `/sessions` | 会话列表，别名 `/resume`、`/continue` |
| `/compact` | 压缩上下文，别名 `/summarize` |
| `/undo` / `/redo` | 撤销/重做，依赖 Git |
| `/editor` | 打开外部编辑器 |
| `/export` | 导出会话为 Markdown |
| `/thinking` | 显示/隐藏 reasoning |
| `/themes` | 切换主题 |
| `/help` | 帮助 |

不同版本可能增加命令，输入 `/` 看实际列表。

## CLI 命令

```bash
opencode run "解释这个仓库"        # 非交互执行
opencode run --format json "..."   # JSON 事件输出
opencode models                    # 列出模型
opencode auth list                 # 查看已认证 provider
opencode stats                     # 用量和成本统计
opencode session list              # 会话列表
opencode export [sessionID]        # 导出会话 JSON
opencode serve                     # 无头 server
opencode web                       # 浏览器界面
opencode attach <url>              # 把 TUI 接到已有 server
opencode github install            # 安装 GitHub agent
opencode agent list                # 列出 agent
opencode mcp list                  # 查看 MCP
```

`opencode run` 适合脚本和 CI：

```bash
opencode run --model miaoapi/gpt-6.1-sol "总结最近的提交"
```

需要机器可读输出时加 `--format json`；需要附加文件时用 `-f`。

## 标准工作流：Plan → Build → Verify → Commit

### 1. Plan

按 `Tab` 切到 Plan，或者：

```bash
opencode --agent plan
```

```text
我要给设置页增加“批量归档项目”。先不要改代码。
阅读相关页面、API 和权限逻辑，给出：
- 需要改哪些文件
- 数据模型和接口
- 失败回滚
- 测试计划
```

### 2. Build

确认计划后按 `Tab` 切回 Build：

```text
按计划实现。先做后端和权限，再做 UI，最后补测试。
每完成一步运行相关测试。
```

### 3. Verify

```text
运行 lint、类型检查和相关测试。
把命令、输出和失败原因一起贴出来。
```

### 4. Review diff

OpenCode 没有 Claude Code 那样内置的 `/code-review` 命令。可以直接让 agent 审查：

```text
审查当前工作区 diff，重点看正确性、边界条件、权限和回归风险。
只报告影响正确性或需求的问题，给出文件和行号。
```

也可以接 GitHub/GitLab 做 PR 级审查。

### 5. Undo / commit

```text
/undo
```

撤销上一轮对话和文件改动。项目必须是 Git 仓库。确认后：

```text
总结改动并提交，写一个清晰的 commit message。
```

## Web 界面

```bash
opencode web
```

默认只监听 `127.0.0.1`，自动打开浏览器。需要局域网访问时：

```bash
OPENCODE_SERVER_PASSWORD=secret opencode web --hostname 0.0.0.0
```

!!! warning "不要裸露到公网"
    `opencode web` 默认没有认证。设 `OPENCODE_SERVER_PASSWORD`，并只在可信网络使用；不要直接暴露到公网。

浏览器和终端可以连接同一个 server：

```bash
opencode web --port 4096
opencode attach http://localhost:4096
```

## IDE 集成

在 VS Code、Cursor、Windsurf、VSCodium 的集成终端里运行：

```bash
opencode
```

扩展会自动安装。常用快捷键：

- `Cmd+Esc` / `Ctrl+Esc`：打开或聚焦 OpenCode；
- `Cmd+Shift+Esc` / `Ctrl+Shift+Esc`：新会话；
- `Cmd+Option+K` / `Alt+Ctrl+K`：插入文件引用；
- 自动共享当前选区或标签页。

JetBrains、Zed 和 Neovim 可以通过 ACP 接入。

## GitHub 与 GitLab

### GitHub

在仓库里：

```bash
opencode github install
```

安装后，在 Issue 或 PR 评论里：

```text
/oc 解释这个 issue 并给出修复方案
/opencode 实现这个功能并提 PR
```

OpenCode 在 GitHub Actions runner 中执行，使用新分支并提交 PR。

### GitLab

可以通过 GitLab CI 组件运行，或在评论里 `@opencode` 使用 GitLab Duo 集成。具体配置取决于你的 GitLab 版本和 runner。

## 一个 10 分钟的任务

```text
目标：给 list 命令增加 --json 参数，输出机器可读 JSON。

上下文：
- 入口 src/cli/list.ts
- 格式化 src/formatters/list.ts
- 测试 test/cli/list.test.ts

约束：
- 不改变默认人类可读输出
- 不新增运行时依赖
- 保持退出码语义

完成标准：
- list --json 输出合法 JSON
- 覆盖空列表、正常列表、错误情况
- lint、类型检查、测试通过

先给计划，确认后再改。
```

然后：

1. 看计划；
2. 实现并跑测试；
3. 让 agent 审查 diff；
4. `/undo` 回滚不满意的尝试；
5. 提交。

## 官方参考

- [TUI](https://opencode.ai/docs/tui/)
- [CLI](https://opencode.ai/docs/cli/)
- [Web](https://opencode.ai/docs/web/)
- [IDE](https://opencode.ai/docs/ide/)
- [GitHub](https://opencode.ai/docs/github/)
- [GitLab](https://opencode.ai/docs/gitlab/)
- [ACP](https://opencode.ai/docs/acp/)
