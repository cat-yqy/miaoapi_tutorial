# 配置与定制：模型、AGENTS.md、Skills、MCP

Pi 的定制点很多，但可以按“影响范围”分层：全局指令、项目指令、模型、提示词模板、技能、扩展、MCP。

先记住两个目录：

```text
~/.pi/agent/      用户级配置
<project>/.pi/    项目级配置（需要 trust）
```

## 目录结构

### 用户级

```text
~/.pi/agent/
├── settings.json
├── auth.json
├── models.json
├── mcp.json
├── AGENTS.md
├── SYSTEM.md
├── APPEND_SYSTEM.md
├── extensions/
├── skills/
├── prompts/
└── themes/
```

### 项目级

```text
<project>/.pi/
├── settings.json
├── mcp.json
├── SYSTEM.md
├── APPEND_SYSTEM.md
├── extensions/
├── skills/
├── prompts/
└── themes/
```

项目级资源只有在 project trust 后才加载；`AGENTS.md` 是例外，它不需要 trust。

## AGENTS.md：长期指令

Pi 会从当前目录和父目录加载：

- `AGENTS.md`；
- `AGENTS.override.md`（同目录替换）；
- `CLAUDE.md` / `CLAUDE.MD`（兼容）；
- `~/.pi/agent/AGENTS.md`（全局）。

建议内容：

```markdown
# Project Rules

## Commands
- Test: `pnpm test`
- Lint: `pnpm lint`
- Typecheck: `pnpm typecheck`

## Conventions
- Use TypeScript strict mode.
- Do not change public API shapes without a migration note.

## Done when
- Tests, lint, and typecheck pass.
```

原则：

- 只写 Pi 猜不到、而且每次都要遵守的内容；
- 短而准，不要写百科；
- 详细资料放 Skill 或文档，按需加载；
- 重复犯错两次后再补规则。

### SYSTEM.md 与 APPEND_SYSTEM.md

- `SYSTEM.md` **替换** Pi 的默认系统提示；
- `APPEND_SYSTEM.md` 在默认系统提示后追加。

除非你明确知道自己在做什么，优先用 `AGENTS.md` 或 `APPEND_SYSTEM.md`。替换整个系统提示可能破坏工具使用和 Agent 行为。

## 设置

交互式：

```text
/settings
```

或直接编辑 `~/.pi/agent/settings.json`。

常见设置：

- 默认模型 / thinking level；
- 默认工具；
- shell 路径；
- 主题和快捷键；
- 资源路径；
- 自动压缩；
- 会话存储位置。

改完手动编辑的文件后，在会话里：

```text
/reload
```

## 模型

### 内置 provider

```text
/login
/model
```

或启动时：

```bash
pi --model <model>
```

API Key 可以通过 `/login` 保存，也可以用 provider 的环境变量。

### 本地模型

Pi 支持通过 llama.cpp router 使用本地 GGUF 模型；官方有 CPU 和 NVIDIA RTX 的配置指南。适合隐私敏感或离线场景，但需要自己管理模型、显存和性能。

### 兼容端点

在 `~/.pi/agent/models.json` 注册 OpenAI/Anthropic/Google 兼容端点。最小示例：

```json
{
  "$schema": "https://pi.dev/schemas/models.schema.json",
  "providers": {
    "myprovider": {
      "baseUrl": "https://api.example.com/v1",
      "api": "openai-completions",
      "apiKey": "$MY_API_KEY",
      "models": [
        { "id": "my-model" }
      ]
    }
  }
}
```

`api` 可选：

- `openai-completions`；
- `openai-responses`；
- `anthropic-messages`；
- Google 兼容协议。

`apiKey` 支持 `$NAME` / `${NAME}` 环境变量插值，也支持 `!command` 在请求时取密钥。

改完打开 `/model`，Pi 会重新加载。

### 可选：接入 MiaoAPI

如果 MiaoAPI 的 `/v1` 端点兼容 Responses API：

```json
{
  "$schema": "https://pi.dev/schemas/models.schema.json",
  "providers": {
    "miaoapi": {
      "baseUrl": "https://miaoapi.site/v1",
      "api": "openai-responses",
      "apiKey": "$MIAOAPI_API_KEY",
      "models": [
        { "id": "gpt-6.1-sol", "name": "GPT-6.1 Sol" }
      ]
    }
  }
}
```

然后：

```bash
export MIAOAPI_API_KEY="sk-你的密钥"
pi
/model miaoapi/gpt-6.1-sol
```

如果 Responses 返回 404，把 `api` 改成 `openai-completions`。模型 ID 以 MiaoAPI 控制台为准。

## Prompt Templates

Prompt Template 是可复用的 `/命令`，适合固定提示词，不包含可执行逻辑。

创建 `~/.pi/agent/prompts/review.md`：

```markdown
---
description: Review staged git changes
argument-hint: "[focus]"
---
Review the staged changes. Focus on ${1:-correctness, security, and error handling}.
```

使用：

```text
/review
/review concurrency
```

支持 `$1`、`$@`、`$ARGUMENTS`、`${1:-default}` 等替换。改完 `/reload`。

## Skills

Skill 是按需加载的 `SKILL.md`，适合放流程、领域知识、参考材料和脚本。

位置：

```text
~/.pi/agent/skills/<name>/SKILL.md
<project>/.pi/skills/<name>/SKILL.md
```

示例：

```markdown
---
name: release-notes
description: 根据最近的 PR 和提交生成发布说明；当用户要求 release notes 或更新日志时使用
---

1. 读取最近合并的 PR 和提交。
2. 按 feat / fix / perf / docs 分类。
3. 面向用户描述影响，不要复制 commit message。
4. 单独列出破坏性变更。
```

触发：

- 模型按 `description` 自动加载；
- 显式 `/skill:release-notes`（如果启用了 skill 命令）。

Skill 与 Prompt Template 的区别：

- Template 只是文本替换；
- Skill 可以带脚本、参考资料，并按需加载完整内容；
- 需要执行逻辑时，用 Extension。

## MCP

Pi 内置 MCP 支持。添加本地服务器：

```bash
pi mcp add filesystem -- npx -y @modelcontextprotocol/server-filesystem .
pi mcp list
```

远程服务器：

```bash
pi mcp add docs --url https://example.com/mcp --bearer-token-env-var DOCS_TOKEN
```

配置位置：

- `~/.pi/agent/mcp.json`：全局；
- `<project>/.pi/mcp.json`：项目，需要 trust。

OAuth：

```bash
pi mcp login <server>
pi mcp logout <server>
```

Pi 支持工具暴露控制（direct / codemode / deferred / hidden）和 `tool_search`，适合服务器多、工具多的场景。MCP 工具会占上下文，不要把所有服务器都打开。

## 扩展与包

- 扩展：TypeScript，能注册工具、命令、provider、事件、TUI；
- 包：把扩展、Skills、Prompt Templates、Themes 打包分发；
- 安装：`pi install npm:...`；
- 管理：`pi list`、`pi config`、`pi remove`、`pi update --extensions`。

详见[扩展与生态](extensions.md)。

## 项目 trust

`.pi` 下的扩展、skills、prompts、MCP 等资源属于可执行/可影响行为的内容，只有在 trust 后才加载。

- 只对你信任的仓库选择 trust；
- 不要为了省事全局关闭信任；
- 项目里的 `SYSTEM.md`、扩展和 hooks 都可能改变 Agent 行为；
- 不信任的仓库先在容器/虚拟机里打开。

## 配置检查清单

- [ ] `AGENTS.md` 简短、有命令和完成标准
- [ ] 模型/端点可用
- [ ] API Key 不在仓库里
- [ ] MCP 只开需要的
- [ ] Skills 的 description 清楚
- [ ] 扩展来源可信
- [ ] 项目 trust 只给信任目录
- [ ] 改完配置执行 `/reload`

## 官方参考

- [Configuration](https://pi.dev/docs/latest/configuration)
- [Models](https://pi.dev/docs/latest/models)
- [Prompt Templates](https://pi.dev/docs/latest/prompt-templates)
- [Skills](https://pi.dev/docs/latest/skills)
- [MCP](https://pi.dev/docs/latest/mcp)
- [Settings](https://pi.dev/docs/latest/settings)
- [Security](https://pi.dev/docs/latest/security)
