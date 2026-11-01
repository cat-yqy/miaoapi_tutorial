# CLI 与 TUI：Web、Headless、ACP、SDK

`dsh` 是唯一的官方 Node 应用启动器。它按 profile 启动不同的应用，而不是提供一堆互不相干的命令。

## 先确认 CLI 可用

```bash
dsh --version
dsh --help
```

如果还没有安装：

```bash
npm install -g @deepseek-ai/dsh
# 或临时运行
npx @deepseek-ai/dsh web
```

要求 Node.js 24.9+。安装细节见[安装页](install.md)。

## 命令总览

```text
dsh web                  启动 Web UI
dsh headless "任务"       执行一个任务并退出
dsh --profile acp        通过 ACP stdio 服务自动化客户端
dsh --profile sdk        通过 SDK JSON-RPC 服务客户端
dsh --profile sdk-minimal
dsh plugin --profile <name> <pnpm args>
dsh --profile tui        启动已安装的 TUI profile
```

`dsh tui` 不是官方内置。要使用 TUI，需要安装社区插件（见下文）。

## Web UI

### 启动

```bash
cd /path/to/project
dsh web
```

默认：

- 监听 `127.0.0.1`；
- 端口 `3080`；
- 本机启动时打开默认浏览器；
- 通过 SSH 启动时只打印宿主机 URL。

### 常用参数

```bash
dsh web --no-open                  # 不自动开浏览器
dsh web --port 8080                # 换端口
dsh web --port 0                   # 让系统分配空闲端口
dsh web --host 192.168.1.20        # 绑定一个具体网卡地址
dsh web --help                     # 查看完整参数
```

`--host` 只接受一个具体的本地 IPv4/IPv6 地址，不接受通配地址。不要把端口直接暴露到公网。

### 远程/反向代理

需要外部访问时，官方建议放在反向代理后面：

```bash
dsh web \
  --public-url https://app.example/ui/ \
  --trusted-host app.example
```

代理必须保留 `Host`、剥离挂载前缀、转发 WebSocket upgrade，并处理 cookie 作用域。需要 TLS 时可以直接：

```bash
dsh web --tls-cert ./server-chain.pem --tls-key ./server-key.pem
```

不要把带启动 token 的 URL 随意发给别人。

## Headless

适合脚本、CI、一次性任务：

```bash
cd /path/to/project
dsh headless "run the tests and summarize failures"
```

从 stdin 读取：

```bash
echo "review this diff" | dsh headless
```

输出 JSONL 事件：

```bash
dsh headless --json "inspect this repository" > events.jsonl
```

### 会话

```bash
dsh headless --session-id <id> "continue"
```

未知 session id 会报错，不会静默新建。

### 权限

```bash
dsh headless \
  --permission-mode workspace-write \
  --approval-policy never \
  --add-dir /repo/.git \
  "commit the fix"
```

- `--permission-mode`：选择已配置的权限预设；
- `--approval-policy ask|never`：审批行为；
- `--add-dir`：在 `workspace-write` 下额外允许一个绝对目录，可重复。

!!! warning "无人值守"
    `--approval-policy never` 只适合你信任的任务和隔离环境。它会跳过人工审批，但不能保证沙箱绝对安全。

## ACP

ACP（Agent Client Protocol）用于编辑器/自动化客户端：

```bash
dsh --profile acp
```

通过 stdio 提供 ACP 服务，直到客户端断开。适合把 DSH 接到支持 ACP 的编辑器或工具。

## SDK

```bash
dsh --profile sdk
dsh --profile sdk-minimal
```

通过 stdio JSON-RPC 为 SDK 客户端提供服务。`sdk-minimal` 是独立极简 agent 配置树，适合嵌入和示例。

Python SDK 通过同一个 CLI 运行时工作；详见[官方 Python 指南](https://deepseek-harness.github.io/deepseek-harness/guide/python-sdk)。

## TUI（社区）

!!! warning "社区组件"
    `@deepseek-harness-tui/dsh-tui` 由社区维护，不是 DeepSeek 官方包。它提供交互式终端界面，安装前请查看源码和发布记录。

### 安装

```bash
npm install -g pnpm
npm install -g @deepseek-ai/dsh @deepseek-harness-tui/dsh-tui
```

启动：

```bash
dsh-tui
# 短别名
dst
```

首次启动会自动初始化 `dsh-tui` profile，需要 pnpm。

### 手动挂载

```bash
dsh plugin --profile dsh-tui add @deepseek-harness-tui/dsh-tui
dsh --profile dsh-tui
```

### 常用命令

```bash
dsh-tui --resume        # 恢复会话
dsh-tui update          # 更新
dsh-tui doctor          # 环境检查
dsh-tui safe            # 只读诊断/修复
dsh-tui version
```

TUI 的设置通常在：

```text
~/.dsh-tui/settings.json
```

它和 `$DSH_HOME` 下的 profile 配置是分开的。

### TUI 的自动化 CLI

```bash
dst -p "总结这个仓库"
dst run --json "检查测试失败"
dst session list --json
```

`dst` 是 TUI 的短别名；`dst dsh <argv...>` 可以转发给原生 DSH。

## 选 Web 还是 TUI

| 需求 | 建议 |
| --- | --- |
| 图形界面、文件树、浏览器面板 | Web/Desktop |
| 纯终端、SSH、低资源 | TUI |
| 脚本/CI | Headless |
| 编辑器集成 | ACP |
| 嵌入产品 | SDK |
| 远程访问 | Web + 反向代理 |

## 会话与工作区

- 启动目录是默认工作区；
- 会话持久化，可在 Web/TUI/Headless 之间按 id 恢复（能力取决于客户端）；
- Headless 用 `--session-id`；
- TUI 用 `--resume`；
- Web 在侧栏切换会话；
- 不要在两个客户端同时改同一个工作区而不协调。

## 进程管理

`dsh web` 是前台进程。长期运行可以用：

```bash
tmux new -s dsh
dsh web --no-open
```

或 systemd/容器。不要用 `nohup` 后忘记 token/端口；正式部署应配反向代理和访问控制。

## 权限与沙箱

DSH 有权限预设、审批和沙箱，但不是绝对隔离：

- Web/Desktop 在需要时弹审批；
- Headless 可用 `--approval-policy` 控制；
- 文件/命令沙箱按 profile 配置；
- 不要把不信任仓库直接放进有凭据的环境跑。

## 配置文件

- 用户配置：`$DSH_HOME/`（默认 `~/.dsh`）；
- profile：`$DSH_HOME/profiles/<name>/cordis.patch.yml`；
- 代理/环境：`$DSH_HOME/.env`；
- 凭据：`$DSH_HOME/.credentials.yaml`；
- 共享 AGENTS.md：`~/.agents/AGENTS.md`；
- 共享 skills：`~/.agents/skills/`。

详见[配置页](configuration.md)。

## 常见问题

| 现象 | 处理 |
| --- | --- |
| `dsh: command not found` | 检查 npm 全局 bin/PATH；用 `npx @deepseek-ai/dsh` |
| Node 版本错误 | 升级到 Node 24.9+ |
| 3080 被占用 | `--port 0` 或 `--port 8080` |
| 浏览器打不开 | 看终端打印的 URL；SSH 用端口转发 |
| TUI 安装失败 | 先装 pnpm；`dsh-tui doctor`；看 pnpm 的 ignored builds |
| TUI 更新后异常 | `dsh-tui safe` 或回退版本 |
| 模型 401 | 检查 Key、provider、Base URL |
| 代理不生效 | 设置 `HTTPS_PROXY`/`HTTP_PROXY`，DSH 不读系统代理 |
| 远程访问 403 | 配 `--trusted-host`，保留 Host，处理 cookie Path |

## 官方参考

- [CLI README](https://github.com/deepseek-ai/deepseek-harness/blob/master/apps/cli/README.zh.md)
- [CLI 帮助](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/cli-help.zh.md)
- [Web 部署](https://deepseek-harness.github.io/deepseek-harness/guide/public-deployments)
- [Python SDK](https://deepseek-harness.github.io/deepseek-harness/guide/python-sdk)
- [TUI 项目](https://github.com/ccch1mneyyy/dsh-TUI)
