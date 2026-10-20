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

## 推荐阅读顺序

1. 先完成 [Codex 安装](guide/install.md)。
2. 登录 MiaoAPI，创建一个 `codex` 分组的 API Key。
3. 从[三种配置方式](guide/configure.md)中选择一种接入。
4. 遇到报错时，查看[常见报错与解决方法](guide/troubleshooting.md)。

!!! warning "保护 API Key"
    教程中的 `your-api-key` 都是占位符。请勿将真实密钥提交到公开仓库，也不要在截图、Issue 或聊天记录中泄露。
