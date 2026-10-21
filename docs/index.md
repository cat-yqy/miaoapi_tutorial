# MiaoAPI 教程

MiaoAPI 提供的 API 可在国内网络环境直接使用 Codex CLI、Codex App、OpenCode、OpenClaw 等工具，也可以在 Python 代码中调用。

[开始阅读](guide/install.md){ .md-button .md-button--primary }
[访问 MiaoAPI 官网](https://miaoapi.site){ .md-button }

<div class="grid cards" markdown>

-   :material-download:{ .lg .middle } __安装 Codex__

    ---

    安装 Node.js 和 Codex CLI，或直接下载 Codex App。

    [:octicons-arrow-right-24: 开始安装](guide/install.md)

-   :material-key-variant:{ .lg .middle } __配置喵API__

    ---

    推荐直接修改配置文件，也可以用 Cockpit Tools 或 CC Switch 管理。

    [:octicons-arrow-right-24: 查看配置](guide/configure.md)

-   :material-image-outline:{ .lg .middle } __GPT-Image-2 生图__

    ---

    用 Python 调用图片生成接口，并保存生成结果。

    [:octicons-arrow-right-24: 查看示例](guide/image-generation.md)

-   :material-console:{ .lg .middle } __OpenCode 配置__

    ---

    配置 provider、模型列表和推理强度。

    [:octicons-arrow-right-24: 配置 OpenCode](guide/opencode.md)

-   :material-bug-outline:{ .lg .middle } __报错排查__

    ---

    处理 403、503、WebSocket 重连等常见问题。

    [:octicons-arrow-right-24: 查看排查方法](guide/troubleshooting.md)

-   :material-open-in-new:{ .lg .middle } __MiaoAPI 官网__

    ---

    注册账号、管理 API Key、查看套餐和最新公告。

    [:octicons-arrow-right-24: 打开官网](https://miaoapi.site)

</div>

## 深入 Codex

配置好之后，Codex 还有计划模式、子代理、Skills、MCP、自动化、Code Review 和云端任务等能力。

[打开 Codex 指南](codex/overview.md){ .md-button .md-button--primary }
[场景实战](codex/playbooks.md){ .md-button }
[最佳实践](codex/best-practices.md){ .md-button }

## 深入 Claude Code

从功能全景、权限与检查点，到子代理、Agent Teams、Skills、MCP 和场景实战。

[打开 Claude Code 指南](claude-code/overview.md){ .md-button .md-button--primary }
[场景实战](claude-code/playbooks.md){ .md-button }
[最佳实践](claude-code/best-practices.md){ .md-button }

## 深入 OpenCode

从 TUI、Plan/Build、子代理，到 Skills、Plugins、MCP、Server/SDK 和场景实战。

[打开 OpenCode 指南](opencode/overview.md){ .md-button .md-button--primary }
[场景实战](opencode/playbooks.md){ .md-button }
[最佳实践](opencode/best-practices.md){ .md-button }

## 更多工具

除了 Codex，也可以把 MiaoAPI 接到其他终端代理。下面的配置都按“安装 → 端点 → 密钥 → 模型 → 验证”组织。

| 工具 | 特点 | 配置 |
| --- | --- | --- |
| **Claude Code** | Anthropic 官方终端代理，只接 Claude 模型 | [接入 Claude Code](guide/claude-code.md) |
| **OpenCode** | 开源、多 provider，支持 Responses / Chat Completions | [配置 OpenCode](guide/opencode.md) |
| **Grok Build** | xAI 终端代理，自定义模型走 `config.toml` | [接入 Grok Build](guide/grok-build.md) |

## 网络与代理

Clash Verge Rev 是桌面代理客户端，和 MiaoAPI 的模型接入是两套独立配置。这里讲安装、订阅、系统代理、TUN、规则和排错。

[打开 Clash Verge 指南](clash-verge/overview.md){ .md-button .md-button--primary }
[详细配置](clash-verge/configuration.md){ .md-button }
[常见问题](clash-verge/troubleshooting.md){ .md-button }

## 推荐阅读顺序

1. 先完成 [Codex 安装](guide/install.md)。
2. 登录 MiaoAPI，创建一个 `codex` 分组的 API Key。
3. 从[三种配置方式](guide/configure.md)中选择一种接入。
4. 遇到报错时，查看[常见报错与解决方法](guide/troubleshooting.md)。

!!! warning "保护 API Key"
    教程中的 `your-api-key` 都是占位符。请勿将真实密钥提交到公开仓库，也不要在截图、Issue 或聊天记录中泄露。
