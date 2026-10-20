# Claude Code 接入 MiaoAPI

Claude Code 是 Anthropic 的终端编码代理。它和 Codex 一样，能读仓库、改文件、跑命令、审查 diff；配置思路上也类似：**安装 → 配模型端点 → 配密钥 → 选模型 → 验证 → 写项目规则**。

!!! warning "Claude Code 只接 Claude 模型"
    Claude Code 官方只支持 Claude 模型。不要把 GPT、Grok 等非 Claude 模型填进 `ANTHROPIC_MODEL`。使用 MiaoAPI 时，请确认你的分组里有可用的 Claude 模型。

## 1. 安装

macOS / Linux：

```bash
curl -fsSL https://claude.ai/install.sh | bash
claude --version
```

macOS 也可以用 Homebrew：

```bash
brew install --cask claude-code
```

Windows PowerShell：

```powershell
irm https://claude.ai/install.ps1 | iex
```

安装后如果提示 `claude: command not found`，先重开终端；仍然找不到就检查 PATH，或运行 `claude doctor` 查看安装状态。

## 2. 创建 MiaoAPI 密钥

登录 MiaoAPI 控制台，进入密钥管理：

1. 新建密钥；
2. 分组选择 Claude（如果控制台没有单独分组，选通用分组，并确认该分组有 Claude 模型权限）；
3. 复制 `sk-...` 密钥。

不要把密钥写进要提交到 Git 的文件。下面的示例都使用占位符。

## 3. 先做一次临时连接

在终端里设置环境变量，然后启动 Claude Code：

```bash
export ANTHROPIC_BASE_URL="https://miaoapi.site"
export ANTHROPIC_AUTH_TOKEN="sk-你的密钥"
export ANTHROPIC_MODEL="claude-sonnet-4-6"
export ANTHROPIC_DEFAULT_HAIKU_MODEL="claude-haiku-4-5"

claude
```

三个地址和变量的含义：

| 变量 | 作用 | 注意 |
| --- | --- | --- |
| `ANTHROPIC_BASE_URL` | 指向 MiaoAPI 网关 | 填 `https://miaoapi.site`，**不要**加 `/v1` |
| `ANTHROPIC_AUTH_TOKEN` | 以 `Authorization: Bearer` 发送密钥 | MiaoAPI 通常用这个 |
| `ANTHROPIC_API_KEY` | 以 `x-api-key` 发送密钥 | 如果 Bearer 返回 401，再改用这个 |
| `ANTHROPIC_MODEL` | 主模型 ID | 换成 MiaoAPI 实际提供的 Claude 模型 |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL` | 后台小任务模型 | 建议设一个便宜、可用的 Claude 模型 |

Claude Code 会在 base URL 后面拼接 `/v1/messages`，所以 `ANTHROPIC_BASE_URL` 只填到域名，不要重复写 `/v1`。

如果 Bearer 方式报 401，改用：

```bash
unset ANTHROPIC_AUTH_TOKEN
export ANTHROPIC_API_KEY="sk-你的密钥"
```

## 4. 写入 Claude Code 配置

临时环境变量只对当前终端生效。要长期使用，把配置写进用户级设置：

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "env": {
    "ANTHROPIC_BASE_URL": "https://miaoapi.site",
    "ANTHROPIC_AUTH_TOKEN": "sk-你的密钥",
    "ANTHROPIC_MODEL": "claude-sonnet-4-6",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "claude-haiku-4-5"
  }
}
```

保存到：

- macOS / Linux：`~/.claude/settings.json`
- Windows：`%USERPROFILE%\.claude\settings.json`

然后收紧权限：

```bash
chmod 600 ~/.claude/settings.json
```

!!! warning "不要把密钥提交到仓库"
    不要把带密钥的 `env` 写进项目的 `.claude/settings.json`。项目级文件是给团队共享的；个人密钥放在 `~/.claude/settings.json`，或项目里的 `.claude/settings.local.json`，并确保它被 `.gitignore` 忽略。

### 配置层级

Claude Code 的配置大致分为：

| 层级 | 位置 | 用途 |
| --- | --- | --- |
| 用户 | `~/.claude/settings.json` | 个人全局默认，适合放网关和模型 |
| 项目共享 | `.claude/settings.json` | 团队共享的权限、hooks、MCP；不要放密钥 |
| 项目本地 | `.claude/settings.local.json` | 只属于你的项目覆盖 |
| 托管 | `managed-settings.json` / MDM | 组织强制策略，优先级最高 |

同一项配置同时出现时，越靠近组织的层级优先级越高；网关和密钥优先放在用户级，团队规则放项目共享级。

## 5. 选择模型

Claude Code 的模型可以来自三处，优先级从高到低大致是：

1. 命令行 `--model` 或会话内 `/model`；
2. `ANTHROPIC_MODEL` 环境变量；
3. `settings.json` 里的 `model` 字段。

常用命令：

```bash
claude --model claude-sonnet-4-6
```

进入会话后：

```text
/model
/status
```

`/status` 会显示当前 base URL、认证来源和模型，适合确认“到底走了 MiaoAPI 还是官方登录”。

如果 MiaoAPI 的模型上下文和 Claude Code 内置假设不一致，可以显式指定：

```bash
export CLAUDE_CODE_MAX_CONTEXT_TOKENS=200000
```

具体数值以模型真实上下文为准，不要为了“看起来更大”随便填。

## 6. 验证连接

启动 Claude Code 后执行：

```text
/status
```

重点看两行：

- `Anthropic base URL` 是否为 `https://miaoapi.site`；
- `Auth token` / `API key` 是否显示你设置的环境变量来源。

也可以直接用 curl 验证网关：

```bash
curl -sS -w '\n%{http_code}\n' -X POST "$ANTHROPIC_BASE_URL/v1/messages" \
  -H "Authorization: Bearer $ANTHROPIC_AUTH_TOKEN" \
  -H "anthropic-version: 2023-06-01" \
  -H "content-type: application/json" \
  -d '{"model":"claude-sonnet-4-6","max_tokens":1,"messages":[{"role":"user","content":"."}]}'
```

判断方式：

- 返回 `{"id":"msg_...`：地址和密钥都通了；
- `401`：密钥或认证头不对，换 `ANTHROPIC_API_KEY` 再试；
- `404`：base URL 或路径不对，检查是否多写/漏写 `/v1`；
- 模型不存在：说明认证已通过，但模型 ID 需要换成 MiaoAPI 实际提供的名字。

## 7. 项目规则：CLAUDE.md

Claude Code 会自动读取项目里的 `CLAUDE.md`。它相当于 Codex 的 `AGENTS.md`，适合放：

- 构建、测试、lint 命令；
- 代码风格和目录结构；
- 不能改的接口或文件；
- 完成标准和验证方式。

可以在项目里运行：

```text
/init
```

生成初始 `CLAUDE.md`，然后手动精简。不要把长文档全文塞进去，只保留“不知道就一定会做错”的规则。

## 8. 常用命令

```bash
claude                     # 交互式会话
claude "解释这个仓库"       # 带初始任务
claude -p "总结这次改动"    # 非交互，输出后退出
claude -c                  # 继续当前目录最近会话
claude -r                  # 选择历史会话恢复
claude --model <model>     # 指定模型
claude doctor              # 检查安装和配置
```

会话内常用：

```text
/status      查看 base URL、认证、模型
/model       切换模型
/init        生成 CLAUDE.md
/clear       清空上下文
```

## 9. 常见问题

| 现象 | 可能原因 | 处理 |
| --- | --- | --- |
| `401` / 认证失败 | 用了错误的变量或密钥分组不对 | Bearer 用 `ANTHROPIC_AUTH_TOKEN`，x-api-key 用 `ANTHROPIC_API_KEY`；确认密钥有 Claude 权限 |
| `404` | base URL 多了 `/v1` 或路径不对 | 改成 `https://miaoapi.site`，让 Claude Code 自己拼 `/v1/messages` |
| 模型不存在 | 模型 ID 不是 MiaoAPI 的 ID | 在控制台查看实际模型名，更新 `ANTHROPIC_MODEL` |
| 仍然走官方账号 | 环境变量没进入当前进程 | 在同一个终端启动 `claude`，或用 `~/.claude/settings.json` |
| 后台任务报模型错误 | 没有设置 Haiku/小模型 | 设置 `ANTHROPIC_DEFAULT_HAIKU_MODEL` |
| MCP 工具不出现 | 第三方网关默认关闭 tool search | 确认网关支持后设置 `ENABLE_TOOL_SEARCH=true` |

## 下一步

配置完成后，可以继续阅读 [Claude Code 功能全景](../claude-code/overview.md)，了解计划模式、检查点、子代理、Agent Teams、Skills 和场景实战。

## 官方参考

- [Claude Code 安装](https://code.claude.com/docs/en/quickstart)
- [Claude Code 设置](https://code.claude.com/docs/en/settings)
- [连接 LLM 网关](https://code.claude.com/docs/en/llm-gateway-connect)
- [环境变量](https://code.claude.com/docs/en/env-vars)
- [模型配置](https://code.claude.com/docs/en/model-config)
- [CLAUDE.md](https://code.claude.com/docs/en/memory)
