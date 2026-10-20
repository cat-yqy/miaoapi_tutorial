# OpenCode 接入 MiaoAPI

OpenCode 是开源的终端编码代理，支持 75+ 模型提供商，也可以用自定义 OpenAI 兼容端点接入 MiaoAPI。它的配置逻辑和 Codex 类似：**安装 → 配 provider → 配密钥 → 选模型 → 验证 → 写项目规则**。

!!! note "版本说明"
    官方目前同时维护 OpenCode 1 和 OpenCode 2。本文按 `opencode.ai/docs` 的 1.x `opencode.json` 写法整理；如果你使用 V2，字段和迁移方式请参考官方 `/v2/docs`。

## 1. 安装

macOS / Linux：

```bash
curl -fsSL https://opencode.ai/install | bash
opencode --version
```

也可以用 npm：

```bash
npm install -g opencode-ai
```

macOS / Linux Homebrew：

```bash
brew install anomalyco/tap/opencode
```

Windows 建议使用 WSL；原生 Windows 可以用 Chocolatey、Scoop 或 npm：

```powershell
choco install opencode
# 或
npm install -g opencode-ai
```

安装后在项目目录运行：

```bash
opencode
```

## 2. 创建 MiaoAPI 密钥

在 MiaoAPI 控制台新建密钥，选择通用或 Codex 分组（以你实际要用的模型权限为准），复制 `sk-...`。

不要把真实密钥写进要提交到 Git 的 `opencode.json`。推荐用环境变量或 `/connect`。

## 3. 配置文件位置

OpenCode 的配置会**合并**，不是互相替换：

| 层级 | 位置 | 用途 |
| --- | --- | --- |
| 全局 | `~/.config/opencode/opencode.json` | 个人默认 provider、模型、权限 |
| 项目 | 项目根目录 `opencode.json` | 团队共享配置，可提交 |
| 自定义 | `OPENCODE_CONFIG` 指向的文件 | 临时或特殊环境 |
| 托管 | 系统级 managed config | 组织强制策略，优先级最高 |

全局配置适合放 MiaoAPI provider；项目配置适合放模型选择和项目规则。

## 4. 推荐配置：走 Responses API

MiaoAPI 提供 `/v1/responses` 时，用 `@ai-sdk/openai`。写入 `~/.config/opencode/opencode.json`：

```json
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "miaoapi": {
      "npm": "@ai-sdk/openai",
      "name": "MiaoAPI",
      "options": {
        "baseURL": "https://miaoapi.site/v1",
        "apiKey": "{env:MIAOAPI_API_KEY}"
      },
      "models": {
        "gpt-6.1-sol": {
          "name": "GPT-6.1 Sol",
          "limit": {
            "context": 200000,
            "output": 65536
          }
        }
      }
    }
  },
  "model": "miaoapi/gpt-6.1-sol"
}
```

然后在启动 OpenCode 的同一个终端里设置：

```bash
export MIAOAPI_API_KEY="sk-你的密钥"
opencode
```

要点：

- `options.baseURL` 要带 `/v1`，OpenCode 会在后面拼 `/responses`；
- `npm` 选 `@ai-sdk/openai` 表示用 Responses API；
- `model` 的格式是 `provider-id/model-id`；
- `limit` 是 OpenCode 用来估算剩余上下文的，按模型实际值调整；
- `{env:MIAOAPI_API_KEY}` 是 OpenCode 的环境变量语法，不要把密钥直接写进 JSON。

## 5. 兼容配置：走 Chat Completions

如果 MiaoAPI 或目标模型不支持 `/v1/responses`，把配置里的 `npm` 字段改成 `@ai-sdk/openai-compatible`，其他字段不变。OpenCode 会走 `/v1/chat/completions`。

!!! warning "不要同时混用两套协议"
    一个 provider 只能选一种 `npm`。如果同一网关同时提供两种协议，建议建两个 provider，例如 `miaoapi` 和 `miaoapi-chat`，避免模型和端点错配。

## 6. 用 `/connect` 管理密钥

如果不想把密钥放进环境变量，可以在 OpenCode 里运行：

```text
/connect
```

选择 **Other**，输入 provider id（要和配置里的 `miaoapi` 一致），粘贴密钥。OpenCode 会把凭据存到：

```text
~/.local/share/opencode/auth.json
```

然后从配置里删掉 `apiKey` 字段，保留 `baseURL` 和模型定义。验证凭据：

```bash
opencode auth list
```

## 7. 选择模型并验证

进入 OpenCode 后：

```text
/models
```

应该能看到 `MiaoAPI` 下的模型。选择后发一条简单任务：

```text
解释这个仓库的目录结构，不要改文件
```

如果模型列表为空，先检查：

- provider id 是否和配置一致；
- `npm` 是否选对（Responses 用 `@ai-sdk/openai`，Chat 用 `@ai-sdk/openai-compatible`）；
- `baseURL` 是否包含 `/v1`；
- 环境变量是否在启动 OpenCode 的同一个终端里。

查看最终合并后的配置：

```bash
opencode debug config
```

## 8. 项目规则：AGENTS.md

在项目里运行：

```text
/init
```

OpenCode 会分析项目并生成 `AGENTS.md`。建议提交到 Git，让团队共享：

- 构建、测试、lint 命令；
- 目录结构和架构约定；
- 不能改的接口；
- 完成标准和验证方式。

也可以手动维护 `AGENTS.md`，不用完全依赖生成结果。

## 9. 日常使用

```bash
opencode                 # 交互式 TUI
opencode run "解释这个函数"  # 非交互执行
```

常用操作：

- `Tab` 切换 Plan / Build 模式；
- `@` 模糊搜索文件；
- `/models` 切换模型；
- `/undo` 撤销上一轮改动；
- `/redo` 重做；
- `/init` 生成 `AGENTS.md`。

权限、MCP、Skills、Agents 和插件都在配置文件的对应区块中管理。先用最小权限跑通，再按需放开。

## 10. 常见问题

| 现象 | 可能原因 | 处理 |
| --- | --- | --- |
| 401 / 认证失败 | 环境变量没生效，或 `/connect` 的 provider id 不一致 | 在同一终端 export；运行 `opencode auth list` |
| 404 | `baseURL` 少了 `/v1`，或协议选错 | Responses 用 `@ai-sdk/openai`，Chat 用 `@ai-sdk/openai-compatible` |
| 模型列表为空 | `models` 没定义，或 provider 未加载 | 检查 `provider.miaoapi.models` 和 `opencode debug config` |
| 上下文显示不对 | `limit.context` 和真实模型不一致 | 按模型实际上下文修改 |
| 改完配置不生效 | 项目配置覆盖了全局配置 | 检查项目根目录的 `opencode.json`，配置是合并的 |

## 官方参考

- [OpenCode 安装与快速开始](https://opencode.ai/docs/)
- [OpenCode 配置](https://opencode.ai/docs/config/)
- [OpenCode Providers](https://opencode.ai/docs/providers/)
- [OpenCode 自定义 provider](https://opencode.ai/docs/providers/#custom-provider)
