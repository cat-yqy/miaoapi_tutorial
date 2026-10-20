# 安装 Codex CLI / App

MiaoAPI 提供的 API 可在国内网络环境直接使用 Codex CLI、Codex App、OpenCode、OpenClaw 等工具，也可以在 Python 代码中调用。

## 安装 Codex App

如果您有特殊的网络工具，强烈建议上官网直接安装 Codex App，根据您的系统下载即可。

<https://openai.com/zh-Hans-CN/codex/>

## 安装 Codex CLI

对于 Codex CLI，您需要先安装 [Node.js](http://nodejs.org/zh-cn?utm_source=chatgpt.com)。

安装完成 Node.js 后，打开终端，输入：

```bash
node -v
npm -v
```

查看是否正确安装。接着输入以下命令安装 Codex CLI（网络不稳定时建议先开启代理）：

```bash
npm install -g @openai/codex
```

安装完成后，输入：

```bash
codex
```

即可打开 Codex CLI。此时尚未配置 API，打开后可按 `Ctrl+C` 先退出，按照[配置喵API](configure.md)继续操作。

如果安装命令卡住，建议使用阿里云镜像：

```bash
npm install -g @openai/codex --registry=https://registry.npmmirror.com
```

!!! tip "安装完成后"
    配置好 API 后，可以继续阅读 [Codex 功能全景](../codex/overview.md)，了解计划模式、子代理、Skills、MCP 和场景实战。
