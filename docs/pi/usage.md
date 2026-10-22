# 日常使用：TUI、会话与命令

Pi 的主界面是终端 TUI，但它同时支持 print、JSON、RPC 和 SDK。先把 TUI 用顺，其他接口都是同一套 Agent/会话机制的不同外壳。

## 启动

```bash
cd /path/to/project
pi
```

Pi 用当前目录发现文件、指令和配置，并按目录组织会话。首次进入可能询问是否信任该目录。

```bash
pi --model <model>          # 指定模型
pi --continue               # 继续当前目录最近会话
pi --resume                 # 选择历史会话
pi --fork                   # 从已有会话分叉
pi --no-session             # 不保存会话
```

## 输入

| 操作 | 作用 |
| --- | --- |
| `Enter` | 发送 |
| `Shift+Enter` | 换行 |
| `Ctrl+G` | 用外部编辑器写长提示 |
| `@` | 搜索并插入文件 |
| `Tab` | 补全路径 |
| 粘贴/拖入图片 | 图片输入（终端支持时） |
| `!command` | 执行 shell，输出进入上下文 |
| `!!command` | 执行 shell，输出不进入上下文 |

`!!` 适合跑不需要模型看的命令，例如打开编辑器、启动一个已经知道的进程。

## 运行中的控制

| 操作 | 作用 |
| --- | --- |
| `Enter` | 把消息作为“steering”插入当前任务 |
| `Alt+Enter` | 作为当前任务完成后的 follow-up |
| `Alt+Up` | 把排队的消息拿回编辑器 |
| `Esc` | 停止当前任务 |
| `Ctrl+O` | 展开/折叠工具输出 |
| `Ctrl+T` | 显示/隐藏 thinking |
| `Ctrl+L` | 打开模型选择器 |
| `Shift+Tab` | 循环 thinking level |

Pi 不会在每个工具调用前询问。执行命令、改文件前，自己看清 transcript 和 diff。

## 模型与思考强度

```text
/model
/thinking
```

`/model` 选择模型；`/thinking` 选择推理档位。也可以在启动时：

```bash
pi --model sonnet:high
```

或：

```bash
pi --model <model> --thinking high
```

可选档位通常包括 `off`、`minimal`、`low`、`medium`、`high`、`xhigh`、`max`，实际会被模型能力裁剪。

## 常用 Slash Commands

| 命令 | 用途 |
| --- | --- |
| `/settings` | 打开设置 |
| `/model [provider/model]` | 选择模型 |
| `/thinking [level]` | 设置思考强度 |
| `/login [provider]` | 登录 provider |
| `/logout` | 退出 provider |
| `/new` | 新会话 |
| `/resume` | 恢复历史会话 |
| `/name [name]` | 命名当前会话 |
| `/session` | 当前会话信息、token、成本 |
| `/tree` | 在会话树中导航 |
| `/fork` | 从更早的用户消息分叉 |
| `/clone` | 复制当前会话 |
| `/compact [instructions]` | 压缩上下文 |
| `/import <path>` | 导入 JSONL 会话 |
| `/copy` | 复制最后一条助手消息 |
| `/export [path]` | 导出 HTML/JSONL |
| `/share` | 上传会话并获取链接 |
| `/trust` | 保存项目 trust 决定 |
| `/reload` | 重新加载扩展、skills、模板、主题、配置 |
| `/hotkeys` | 查看当前快捷键 |
| `/bug [description]` | 生成私有 bug 报告 |
| `/quit` | 退出 |

扩展、Prompt Template 和 Skill 还会增加自己的命令。输入 `/` 看当前会话实际可用项。

## 会话是一棵树

Pi 的会话保存为 JSONL，消息按树组织。当前分支决定模型看到的历史。

### 继续

```bash
pi --continue
pi --resume
```

会话内：

```text
/resume
```

### 分叉

| 操作 | 结果 |
| --- | --- |
| `/tree` | 在当前会话文件内切换分支 |
| `/fork` | 从更早的用户消息创建新会话 |
| `/clone` | 复制当前分支到新会话 |

回到更早的消息、修改后重新提交，会形成另一条分支；原来的分支不会被删除。

### 压缩

当上下文接近上限，Pi 会自动压缩较早历史。也可以手动：

```text
/compact
/compact 保留 OAuth 方案和测试命令
```

压缩会插入摘要条目，原始消息仍保留在会话树中，但后续请求只带摘要和近期消息。

### 导出

```text
/export
/share
```

导出/分享前先审查内容：会话可能包含提示词、工具参数、命令输出、文件内容，甚至凭据。

## 工具

默认工具通常包括：

- `read`：读文件；
- `bash`：执行命令；
- `edit`：修改文件；
- `write`：写文件。

可以限制工具：

```bash
pi --tools read,grep,find,ls --print "审查这个项目"
```

或用：

```bash
pi --tools +codemode,-write
```

`--tools` 只影响模型可见工具，不会移除扩展或 MCP 的工具注册；详见官方 CLI。

## CLI / 自动化模式

### Print

```bash
pi -p "总结这个仓库"
pi --print "审查这次改动"
```

只输出最终助手文本，适合脚本。

### JSON

```bash
pi --mode json "分析这个仓库" > events.jsonl
```

输出 JSONL 事件流，适合程序处理。

### RPC / SDK

```bash
pi --mode rpc
```

RPC 从 stdin 读 JSONL 命令、stdout 写响应和事件。TypeScript SDK 可以在进程内创建和控制会话。

所有接口共享同一套 Agent 和会话机制，不是两套实现。

## 项目指令：AGENTS.md

Pi 会自动读取：

- 当前目录及父目录的 `AGENTS.md`；
- `AGENTS.override.md`（同目录替换）；
- `CLAUDE.md`（兼容）；
- `~/.pi/agent/AGENTS.md`（全局）。

建议写：

- 构建/测试/lint 命令；
- 目录结构和架构约定；
- 不能改的接口；
- 完成标准。

`AGENTS.md` 不需要项目 trust 就会加载；`.pi` 下的扩展/技能等资源才需要 trust。

## 诊断

```text
/hotkeys    当前快捷键
/session    会话、token、成本
/debug      写终端渲染和会话诊断到 pi-debug.log
/bug        生成 bug 报告
```

`pi-debug.log` 可能包含提示词、模型输出、工具结果和文件内容；分享前脱敏。

## 安全提醒

Pi 没有内置沙箱，工具和扩展以你的用户权限运行。TUI 里看到命令和 diff 再放行；对不信任的仓库、扩展和自动化任务，使用容器/虚拟机隔离。

## 官方参考

- [Usage](https://pi.dev/docs/latest/usage)
- [Sessions](https://pi.dev/docs/latest/sessions)
- [Slash Commands](https://pi.dev/docs/latest/slash-commands)
- [CLI](https://pi.dev/docs/latest/cli)
- [Security](https://pi.dev/docs/latest/security)
