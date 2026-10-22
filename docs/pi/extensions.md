# 扩展与生态：安装、编写与推荐

Pi 的扩展性来自四个层次：

| 层次 | 做什么 | 是否需要写代码 |
| --- | --- | --- |
| **Prompt Template** | 可复用 `/命令` | 否，Markdown |
| **Skill** | 按需加载的流程/知识/脚本 | 少量 Markdown |
| **Extension** | 注册工具、命令、provider、事件、TUI | TypeScript |
| **Package** | 打包分发以上资源 | 取决于内容 |

Pi 的核心很小，很多“别的 Agent 自带”的能力，在 Pi 里是扩展。这就是它灵活的原因，也是它需要你审代码的原因。

!!! warning "扩展以你的权限运行"
    Pi 扩展是 TypeScript 模块，加载进 Pi 进程，能读提示词、工具调用、文件、凭据和会话历史。**只安装你信任的扩展，安装前看源码。**

## 安装扩展/包

### 从 npm

```bash
pi install npm:pi-web-access
```

### 从 git

```bash
pi install git:github.com/owner/repo@v1
```

### 从本地目录

```bash
pi install ./my-package
```

### 临时试用，不永久安装

```bash
pi -e npm:pi-web-access
```

可以同时加载多个：

```bash
pi -e npm:pi-web-access -e npm:pi-subagents
```

### 安装到项目而不是全局

```bash
pi install -l npm:pi-web-access
```

`--local` / `-l` 写入项目 `.pi/settings.json`，项目 trust 后才加载。

### 管理

```bash
pi list                    # 已配置的包
pi config                  # 启用/禁用资源
pi remove npm:pi-web-access
pi update --extensions     # 更新所有包
pi update npm:pi-web-access
```

`pi config` 可以按资源粒度启用/禁用，不必卸载整个包。

## 怎么选来源

优先：

- 官方文档/官方目录列出的包；
- 有公开源码仓库和 README；
- 作者可识别、版本有持续更新；
- 安装命令和包名一致；
- 代码量可审，依赖不过度。

安装前检查：

1. 包名是否与 `pi.dev/packages` 一致；
2. 是否有源码链接；
3. 是否有 postinstall 脚本/二进制下载；
4. 是否读取凭据、网络、文件系统；
5. 是否声明最低 Pi 版本；
6. 最近是否维护。

!!! tip "先临时加载"
    用 `pi -e npm:...` 试一次，确认行为和权限符合预期，再 `pi install` 永久安装。

## 写一个最小扩展

Extension 的默认导出是一个工厂函数，接收 `ExtensionAPI`。

创建 `~/.pi/agent/extensions/hello.ts`：

```ts
import type { ExtensionAPI } from "@earendil-works/pi-coding-agent";

export default function (pi: ExtensionAPI) {
  pi.registerCommand("hello", {
    description: "Show a greeting",
    handler: async (name, ctx) => {
      ctx.ui.notify(`Hello, ${name || "world"}!`, "info");
    },
  });
}
```

加载：

```bash
pi --extension ./hello.ts
```

或放到：

```text
~/.pi/agent/extensions/    # 用户级
<project>/.pi/extensions/  # 项目级，需要 trust
```

启动 Pi 后运行：

```text
/hello
```

本地 TypeScript 由 jiti 直接加载，不需要单独编译。

### 能注册什么

- 工具：`pi.registerTool()`；
- 命令：`pi.registerCommand()`；
- 快捷键/CLI flag：`registerShortcut()`、`registerFlag()`；
- provider：`registerProvider()`；
- MCP 服务器：`registerMcpServer()`；
- 生命周期事件：`pi.on()`；
- TUI 渲染器。

具体 API 以[官方扩展文档](https://pi.dev/docs/latest/extensions)为准。

### 一个权限门禁示例

```ts
import type { ExtensionAPI } from "@earendil-works/pi-coding-agent";

export default function (pi: ExtensionAPI) {
  pi.on("tool_call", async (event, ctx) => {
    if (event.toolName === "bash" && String(event.input.command).includes("rm -rf")) {
      const ok = await ctx.ui.confirm("Allow destructive command?", String(event.input.command));
      if (!ok) return { block: true, reason: "blocked by user" };
    }
  });
}
```

这比“在提示词里叫模型不要删库”可靠得多。

## 发布成 Package

一个 Pi 包可以包含：

- `extensions/`；
- `skills/`；
- `prompts/`；
- `themes/`；
- 自己的依赖。

包可以是普通 npm 包、git 仓库或本地目录。用 `package.json` 的 `pi` 字段声明资源路径，或使用约定目录。

发布前：

- 写清 README、安装命令、权限需求；
- 声明最低 Pi 版本；
- 固定依赖版本；
- 不把密钥写进包；
- 提供可撤销/禁用方式。

## 推荐扩展清单

以下是目前社区里较常用、职责清晰的扩展。**不是“全装清单”**，按需选 2–3 个开始。

| 扩展 | 用途 | 安装 |
| --- | --- | --- |
| **pi-web-access** | Web 搜索、网页提取、视频理解；支持多种搜索/抓取后端 | `pi install npm:pi-web-access` |
| **pi-subagents** | 子代理：审查、侦察、并行实现、保存工作流 | `pi install npm:pi-subagents` |
| **pi-mcp-adapter** | 懒加载 MCP，减少工具定义占用；服务器按需启动 | `pi install npm:pi-mcp-adapter` |
| **@narumitw/pi-plan-mode** | 类似 `/plan` 的只读计划模式，计划确认后再实现 | `pi install npm:@narumitw/pi-plan-mode` |
| **@gotgenes/pi-permission-system** | 对 tool/bash/MCP/skill/path 做 allow/ask/deny 门禁 | `pi install npm:@gotgenes/pi-permission-system` |
| **pi-memory** | Markdown 长期记忆、日志、scratchpad；可选语义搜索 | `pi install npm:pi-memory` |
| **pi-background-tasks** | 后台 shell 任务、委托子代理、多模型 Fusion | `pi install npm:pi-background-tasks` |
| **pi-simplify** | 审查改动代码的清晰度/一致性，并运行测试 | `pi install npm:pi-simplify` |
| **pi-powerline-footer** | Powerline 状态栏、欢迎页、队列、编辑器暂存 | `pi install npm:pi-powerline-footer` |
| **@langfuse/pi-observability-plugin** | 把会话、模型调用、工具调用送到 Langfuse | `pi install npm:@langfuse/pi-observability-plugin` |
| **pi-claude-bridge** | 通过 Agent SDK 在 Pi 里使用 Claude Code/Claude 模型 | `pi install npm:pi-claude-bridge` |
| **@plannotator/pi-extension** | 浏览器里可视化审阅/标注计划 | `pi install npm:@plannotator/pi-extension` |

### 怎么选

**写代码为主：**

- `@narumitw/pi-plan-mode`：先计划；
- `@gotgenes/pi-permission-system`：加权限门禁；
- `pi-subagents`：审查/并行；
- `pi-simplify`：提交前清理；
- `pi-memory`：记住项目偏好。

**研究/资料为主：**

- `pi-web-access`；
- `pi-subagents`（并行调研）；
- `pi-memory`。

**长任务：**

- `pi-background-tasks`；
- `pi-subagents` 的 background children。

**MCP 很多：**

- `pi-mcp-adapter`；
- 如果只有一两个服务器，先用 Pi 内置 MCP。

**团队：**

- `@langfuse/pi-observability-plugin` 或 Braintrust；
- `@gotgenes/pi-permission-system`。

**UI：**

- `pi-powerline-footer`。

### 不建议一上来全装

- 每个扩展都增加上下文、权限面和维护成本；
- 多个扩展可能注册同名命令/工具；
- 扩展更新可能改变行为；
- 出问题时很难定位是哪个扩展。

先装 2–3 个，确认稳定后再加。

## 版本与更新

- 固定版本：`pi install npm:pi-web-access@0.44.0`；
- 更新：`pi update --extensions`；
- 更新后回归测试关键流程；
- 注意扩展要求的最低 Pi 版本；
- 生产环境避免 `latest` 自动漂移。

## 安全清单

- [ ] 扩展来自可信作者/仓库
- [ ] 看过 README 和权限需求
- [ ] 没有可疑 postinstall/二进制下载
- [ ] 先用 `pi -e` 试过
- [ ] 不在不信任的仓库自动加载项目扩展
- [ ] 密钥不进入扩展配置/会话
- [ ] 知道如何禁用/卸载
- [ ] 重要任务在容器/隔离环境运行

## 官方/社区参考

- [Pi Packages 目录](https://pi.dev/packages)
- [Build Extensions](https://pi.dev/docs/latest/extensions)
- [Pi Packages](https://pi.dev/docs/latest/packages)
- [Skills](https://pi.dev/docs/latest/skills)
- [pi-extensions 社区合集](https://github.com/Villoh/pi-extensions)
