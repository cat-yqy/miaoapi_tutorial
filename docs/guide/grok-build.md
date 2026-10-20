# Grok Build 接入 MiaoAPI

Grok Build 是 xAI 的终端编码代理，命令是 `grok`，**不是** `grok build`。它和 Codex、Claude Code 的定位类似：在终端里读仓库、改文件、跑命令、搜索网页和处理长任务。

它的配置逻辑也和 Codex 很像：**安装 → 认证 → 在 `config.toml` 定义自定义模型 → 选模型 → 验证 → 写项目规则**。

!!! note "先确认 MiaoAPI 是否提供 Grok 模型"
    Grok Build 对 Grok 模型的工具调用适配最好。如果 MiaoAPI 没有 Grok 系列，也可以填它提供的任意 OpenAI 兼容模型，但工具调用和提示词效果需要实测。下面的模型名是示例，请换成控制台里的真实 ID。

## 1. 安装

macOS / Linux / Git Bash：

```bash
curl -fsSL https://x.ai/cli/install.sh | bash
grok --version
```

Windows PowerShell：

```powershell
irm https://x.ai/cli/install.ps1 | iex
```

安装后可以用以下命令更新：

```bash
grok update
```

## 2. 首次启动与登录

直接运行：

```bash
grok
```

首次启动通常会打开浏览器登录 xAI。完成一次登录即可；凭据保存在 `~/.grok/auth.json`。

如果你在无浏览器的环境里，可以：

```bash
grok login --device-auth
```

!!! tip "自定义模型仍然使用 MiaoAPI Key"
    xAI 登录只解决首次启动和 Grok Build 自身的认证。真正调用 MiaoAPI 自定义模型时，会优先使用 `[model.*]` 里的 `env_key` 或 `api_key`。两者不要混淆。

## 3. 创建 MiaoAPI 密钥

在 MiaoAPI 控制台新建密钥。如果控制台有 Grok 分组就选 Grok；没有就选通用分组，并确认该分组能访问你要用的模型。

把密钥放进环境变量，而不是直接写进配置文件：

```bash
export MIAOAPI_API_KEY="sk-你的密钥"
```

Windows PowerShell：

```powershell
$env:MIAOAPI_API_KEY = "sk-你的密钥"
```

## 4. 配置自定义模型

编辑全局配置：

- macOS / Linux：`~/.grok/config.toml`
- Windows：`%USERPROFILE%\.grok\config.toml`

!!! note "模型配置放全局"
    项目里的 `.grok/config.toml` 主要支持 MCP、插件和权限规则，不适合放完整的模型/provider 配置。自定义模型请写在全局 `~/.grok/config.toml`。

加入：

```toml
[models]
default = "miaoapi-grok"

[model.miaoapi-grok]
model = "grok-4.6"                    # 换成 MiaoAPI 实际模型 ID
base_url = "https://miaoapi.site/v1"
name = "MiaoAPI · Grok"
env_key = "MIAOAPI_API_KEY"
api_backend = "responses"
context_window = 200000
```

关键字段：

| 字段 | 作用 |
| --- | --- |
| `model` | 发给 MiaoAPI 的模型 ID，不是配置里的显示名 |
| `base_url` | OpenAI 兼容端点，要带 `/v1` |
| `name` | 模型选择器里显示的名字 |
| `env_key` | 从哪个环境变量读取 MiaoAPI Key |
| `api_backend` | `responses`、`chat_completions` 或 `messages` |
| `context_window` | 告诉 Grok Build 何时触发自动压缩；不填默认 200k |

`api_backend` 的选择：

- MiaoAPI 支持 `/v1/responses`：用 `"responses"`；
- 只支持 `/v1/chat/completions`：改成 `"chat_completions"`；
- 接 Anthropic Messages 协议：用 `"messages"`，通常还需要 `extra_headers` 传 `x-api-key` 和 `anthropic-version` 等头。

如果 Responses 返回 404，先换成：

```toml
api_backend = "chat_completions"
```

## 5. 选择并验证模型

列出模型：

```bash
grok models
```

查看当前目录最终加载了哪些配置、skills、MCP 和规则：

```bash
grok inspect
```

在 TUI 里切换：

```text
/model miaoapi-grok
```

或者从命令行指定：

```bash
grok -m miaoapi-grok "解释这个仓库的目录结构"
```

如果模型没有出现在列表里：

- 确认 `[models] default` 和 `[model.<name>]` 的 key 一致；
- 确认 TOML 没有语法错误；
- 确认 `grok inspect` 能看到你的配置来源；
- 确认 `MIAOAPI_API_KEY` 在启动 `grok` 的同一个终端里。

## 6. 项目规则：AGENTS.md 和 CLAUDE.md

Grok Build 会自动读取：

- `~/.grok/AGENTS.md`：个人全局规则；
- `<repo>/AGENTS.md`：仓库规则；
- `<cwd>/AGENTS.md`：当前目录规则，优先级更高；
- `CLAUDE.md` 及其常见变体：兼容 Claude Code 项目规则；
- `.grok/rules/` 下的 Markdown 规则。

建议把构建、测试、lint、目录结构、禁止事项和完成标准写进 `AGENTS.md`。和 Codex 一样，短而准比长而全更有用。

## 7. 常用操作

```bash
grok                         # 交互式 TUI
grok "修复登录测试"           # 带初始任务
grok -c                      # 继续最近会话
grok --resume <session-id>   # 恢复指定会话
grok -m miaoapi-grok         # 指定模型
grok -p "总结这次改动"        # 非交互，输出后退出
```

TUI 内常用：

```text
/model miaoapi-grok    切换模型
/compact               压缩上下文
/new                   新会话
/always-approve        切换自动批准
```

权限相关：

- 默认会在执行命令或改文件前询问；
- `Ctrl+O` 切换 always-approve；
- `--yolo` 启动时自动批准，风险较高，只在可信环境使用。

## 8. 无头模式与自动化

```bash
grok -p "检查最近的提交，列出风险点" --output-format json
```

`--output-format streaming-json` 可以输出 NDJSON 事件流，适合接脚本或 CI。和 `codex exec` 一样，自动化场景要显式控制权限，不要把 `--yolo` 当成默认。

## 9. 常见问题

| 现象 | 可能原因 | 处理 |
| --- | --- | --- |
| `401` / 认证失败 | MiaoAPI Key 没读到，或分组无权限 | 在同一终端 export；检查 `grok inspect` |
| `404` | `base_url` 路径不对，或 `api_backend` 选错 | 用 `https://miaoapi.site/v1`；Responses 404 时换 `chat_completions` |
| 模型不存在 | `model` 不是 MiaoAPI 的真实 ID | 在控制台查看模型 ID，更新 `[model.*].model` |
| 配置不生效 | 改的是项目配置，但全局默认没变 | 检查 `~/.grok/config.toml` 和 `grok inspect` |
| 首次启动一直要求登录 | 还没完成 xAI 登录 | 运行 `grok login`，或使用 `grok login --device-auth` |
| 工具调用不稳定 | 用非 Grok 模型跑 Grok Build | 换 Grok 模型，或接受兼容性测试成本 |

调试时打开日志：

```bash
RUST_LOG=debug GROK_LOG_FILE=/tmp/grok.log grok
tail -f /tmp/grok.log
```

重点看 `model`、`sampling`、请求 URL 和 HTTP 状态码。

## 10. 和 Codex / Claude Code 的区别

| 项目 | Codex | Claude Code | Grok Build |
| --- | --- | --- | --- |
| 命令 | `codex` | `claude` | `grok` |
| 全局配置 | `~/.codex/config.toml` | `~/.claude/settings.json` | `~/.grok/config.toml` |
| 项目规则 | `AGENTS.md` | `CLAUDE.md` | `AGENTS.md` / `CLAUDE.md` |
| 自定义端点 | `[model_providers.*]` | `ANTHROPIC_BASE_URL` | `[model.*].base_url` |
| 推荐协议 | Responses | Anthropic Messages | Responses / Chat Completions |

## 官方参考

- [Grok Build 官方文档](https://docs.x.ai/build/overview)
- [Grok Build 安装与快速开始](https://docs.x.ai/build/overview)
- [Custom Models](https://github.com/xai-org/grok-build/blob/main/crates/codegen/xai-grok-pager/docs/user-guide/11-custom-models.md)
- [Grok Build GitHub](https://github.com/xai-org/grok-build)
- [Grok Build 发布说明](https://x.ai/news/grok-build-cli)
