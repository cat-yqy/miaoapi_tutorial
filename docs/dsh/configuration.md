# 配置：模型、代理、指令与插件

DSH 的配置分三层：

1. **用户级**：`$DSH_HOME`（默认 `~/.dsh`）；
2. **profile 级**：`$DSH_HOME/profiles/<profile>/`；
3. **项目级**：工作区里的 `AGENTS.md`、`.agents/skills`、`.dsh/skills` 等。

Desktop、Web、TUI 共享部分数据，但 **profile 和插件激活是分开的**。这一点最容易让人误以为“Desktop 配好了，CLI 也应该好了”。

## 目录与优先级

### `$DSH_HOME`

默认：

```text
~/.dsh
```

可用环境变量覆盖：

```bash
export DSH_HOME=/path/to/dsh-home
```

优先级：

```text
显式配置 > $DSH_HOME > ~/.dsh
```

### 关键文件

| 路径 | 内容 |
| --- | --- |
| `$DSH_HOME/.env` | 代理、API Key 等环境变量 |
| `$DSH_HOME/.credentials.yaml` | 设置页保存的凭据 |
| `$DSH_HOME/profiles/<name>/cordis.patch.yml` | 该 profile 的用户配置覆盖 |
| `$DSH_HOME/profiles/<name>/package.json` | profile 的插件依赖 |
| `$DSH_HOME/sessions/` | 会话数据 |
| `$DSH_HOME/skills/` | 用户 skills（DSH 根） |
| `~/.agents/AGENTS.md` | 共享的用户级指令 |
| `~/.agents/skills/` | 共享的用户级 skills |
| `~/.dsh-tui/settings.json` | 社区 TUI 自己的设置 |

### `$DSH_HOME/.env`

适合放不想每次 export 的变量：

```dotenv
HTTPS_PROXY=http://127.0.0.1:7890
HTTP_PROXY=http://127.0.0.1:7890
MIAOAPI_API_KEY=sk-...
```

已导出的环境变量优先于 `.env`。

### profile

```text
$DSH_HOME/profiles/web/
$DSH_HOME/profiles/desktop/
$DSH_HOME/profiles/dsh-tui/
$DSH_HOME/profiles/headless/
```

Desktop 的 `desktop` profile 由 Electron 持有，npm CLI 不能启动或管理它。Desktop 自带命令可以管理。

**模型/provider 配置通常按 profile 写入**。在 Desktop 设置里配的 provider 不一定自动出现在 `dsh web`；需要在对应 profile 再配，或复制 `cordis.patch.yml` 中相关段落。

## 模型

### 推荐：用设置页

Desktop/Web：

1. **设置 → 模型**；
2. DeepSeek 卡片填 API Key；
3. 或“添加模型提供商”；
4. 或“自定义模型 API”。

保存后下一次请求生效，不需要重启。

### DeepSeek

- 在 [DeepSeek 开放平台](https://platform.deepseek.com/api_keys) 创建 Key；
- 填进 DeepSeek 卡片；
- 保存后只显示脱敏描述符；
- 明文存储在 `$DSH_HOME/.credentials.yaml`，不会回显到页面。

默认端点下，DeepSeek 模型列表跟随官方发布；也可以自定义模型目录。

### 第三方 provider

DSH 自带一些 provider，例如：

- `anthropic`；
- `openai`；
- `moonshotai`（Kimi）；
- `zai`（GLM）。

在“第三方模型提供商”里选 provider，填 Key。端点、协议和模型列表由已安装目录提供。

### 自定义模型 API

适合中转站、公司网关、自建服务：

| 字段 | 说明 |
| --- | --- |
| **Provider ID** | 永久标识，小写；会话和凭据会引用它 |
| **基础 URL** | API 根地址，例如 `https://api.example.com/v1` |
| **API 协议** | OpenAI Chat Completions / OpenAI Responses / Anthropic Messages |
| **API Key** | 可保存，也可留空用环境变量 |
| **模型** | 至少一个模型 ID；可补上下文、最大输出、输入类型 |

一个 provider 只使用一种协议；网关同时支持两种时，建两个 provider。

### 接入 MiaoAPI（可选）

如果 MiaoAPI 的 `/v1` 兼容 OpenAI Chat Completions：

1. 设置 → 模型 → 添加模型提供商；
2. 切到“自定义模型 API”；
3. 填：

```text
Provider ID: miaoapi
基础 URL:    https://miaoapi.site/v1
API 协议:    OpenAI Chat Completions
API Key:     sk-你的密钥
模型:        gpt-6.1-sol
```

如果端点支持 Responses，也可以选 OpenAI Responses。模型 ID 以 MiaoAPI 控制台为准。

### 高级：`cordis.patch.yml`

设置页写入的就是 profile 的 `cordis.patch.yml`。例如：

```yaml
- id: llm-pi-ai
  config:
    providers:
      miaoapi:
        apiKeyEnv: MIAOAPI_API_KEY
        api: openai-completions
        baseURL: https://miaoapi.site/v1
        models:
          - id: gpt-6.1-sol
```

要点：

- `id` 是插件行 ID，不要随意改；
- 覆盖会替换该条目的完整配置，编辑时保留其他 provider/字段；
- `apiKeyEnv` 从环境变量读取；
- `api` 可选 `openai-completions`、`openai-responses`、`anthropic-messages`；
- 改完下一次请求生效，通常不用重启。

模型输入类型、上下文、推理等级等高级字段见[官方模型指南](https://deepseek-harness.github.io/deepseek-harness/guide/providers)。

## 网络代理

DSH **不读取操作系统代理设置**。浏览器能走系统代理，不代表 `dsh` 能走。

### 环境变量

```bash
export HTTPS_PROXY=http://127.0.0.1:7890
export HTTP_PROXY=http://127.0.0.1:7890
export NO_PROXY=internal.example.com,.corp.example.com
```

或写进 `$DSH_HOME/.env`：

```dotenv
HTTPS_PROXY=http://127.0.0.1:7890
HTTP_PROXY=http://127.0.0.1:7890
NO_PROXY=internal.example.com,.corp.example.com
```

### 注意

- **不支持 SOCKS**：`socks5://` 会被报告并跳过；请指向代理的 HTTP 端口。
- `ALL_PROXY` 也能作为两种协议的兜底，但显式写 `HTTPS_PROXY` 更清楚。
- `NO_PROXY` 支持主机名/域名后缀，**不支持 CIDR**。
- `localhost`/`127.0.0.1` 始终直连。
- 代理 URL 里的密码会进入环境，DSH 启动的子进程/工具也能读到；敏感环境用无密码入口或额外认证。
- 企业 TLS 拦截需要：
  ```bash
  export NODE_EXTRA_CA_CERTS=/path/to/corporate-ca.pem
  ```
  Node 只在启动时读取，先 export 再启动 `dsh`。

### 验证

```bash
dsh headless "fetch https://example.com and tell me the page title"
```

同时看代理软件连接日志。没有请求就检查：

```bash
env | grep -i proxy
```

## AGENTS.md：项目指令

DSH 会从项目根目录（含 `.git` 的最近祖先）到当前工作目录的链上发现：

- `AGENTS.md`；
- `CLAUDE.md`（兼容）；
- `AGENTS.local.md`；
- `CLAUDE.local.md`；
- `~/.agents/AGENTS.md`（用户级）。

建议写：

```markdown
# Project Rules

## Commands
- Test: `pnpm test`
- Lint: `pnpm lint`

## Conventions
- TypeScript strict.
- Do not change public API without a migration note.

## Done when
- Tests, lint, typecheck pass.
```

原则：

- 短、可执行；
- 写命令、边界、完成标准；
- 不写百科；
- 详细流程放 Skill；
- 重复犯错两次后再补规则。

## Skills

本地 skill 根目录与优先级：

| 优先级 | 范围 | 路径 |
| --- | --- | --- |
| 100 | 项目 DSH | `<project>/.dsh/skills` |
| 200 | 项目共享 | `<project>/.agents/skills` |
| 400 | 用户 DSH | `$DSH_HOME/skills` |
| 500 | 用户共享 | `~/.agents/skills` |

支持：

- 目录 bundle：`<name>/SKILL.md`；
- 平铺文件：`<name>.md`；
- 不扫描嵌套的 `**/SKILL.md`。

最小示例：

```markdown
---
name: release-notes
description: 根据最近合并的 PR 生成发布说明；当用户要求 release notes 时使用
---

1. 读取最近合并的 PR。
2. 按 feat/fix/perf/docs 分类。
3. 面向用户描述影响。
4. 单独列出破坏性变更。
```

`name` 必须是 kebab-case，`description` 必填。

## 权限与沙箱

DSH 有：

- 权限预设；
- 审批；
- 文件沙箱；
- 命令沙箱（bash/pwsh）；
- Windows ACL 沙箱。

但官方明确说：沙箱和审批降低风险，不保证隔离。

### Desktop/Web

- 在设置里选择权限预设；
- 需要审批的操作会弹窗；
- 不要无脑“永远允许”。

### Headless

```bash
dsh headless \
  --permission-mode workspace-write \
  --approval-policy ask \
  --add-dir /repo/.git \
  "commit the fix"
```

- `ask`：需要时审批；
- `never`：跳过审批，只适合可信任务/隔离环境；
- `--add-dir`：额外允许一个绝对目录。

### 建议

- 不信任仓库用容器/虚拟机；
- 不在有生产凭据的环境跑未知插件；
- 定期备份工作区；
- 大改动前 commit。

## 插件

DSH 的能力大量来自插件。

### Desktop/Web

- **插件 → 已安装**：启用/禁用；
- **插件 → 发现**：第三方索引（需先添加索引源）；
- **Creator 模式**：用对话创建/安装插件；
- 官方插件通常随 DSH 提供。

插件索引是第三方内容。DSH 校验格式，但不验证标题、作者、图标等声明；安装前核对确认框里的来源。

### CLI

```bash
dsh plugin --profile web add <package>
dsh plugin --profile web remove <package>
dsh plugin --profile web list
dsh plugin --profile web why <package>
```

`dsh plugin` 把参数转发给 profile 目录里的 pnpm。安装/升级会做版本兼容检查。

TUI 的 profile：

```bash
dsh plugin --profile dsh-tui add <package>
```

### 安全

插件运行在 DSH 进程里，能访问工具、文件、凭据和会话。只安装可信来源；优先固定版本；升级后回归测试。

## TUI 配置

社区 TUI 的设置通常在：

```text
~/.dsh-tui/settings.json
```

它和 `$DSH_HOME` 下的 profile 配置分开。TUI 的模型、主题、快捷键等以 TUI 自己的 `/settings` 为准。

## 备份与迁移

```bash
# 退出 Desktop/CLI 后
cp -a ~/.dsh ~/.dsh.backup
```

注意：

- `~/.dsh` 含凭据、会话、插件；
- 不要提交到 Git；
- 迁移到新机器后，路径/绝对目录可能变化；
- Desktop 和 CLI 的 profile 不要混用；
- TUI 的 `~/.dsh-tui` 单独备份。

## 配置检查清单

- [ ] `$DSH_HOME` 位置明确
- [ ] API Key 不在仓库里
- [ ] 模型/provider 在当前 profile 生效
- [ ] 代理用环境变量，不用系统代理
- [ ] `NO_PROXY` 覆盖内网
- [ ] AGENTS.md 简短可执行
- [ ] Skills 的 `description` 清楚
- [ ] 插件来源可信、版本固定
- [ ] 权限预设最小化
- [ ] 已备份 `~/.dsh`

## 官方参考

- [配置模型](https://deepseek-harness.github.io/deepseek-harness/guide/providers)
- [网络代理](https://deepseek-harness.github.io/deepseek-harness/guide/network-proxy)
- [插件索引](https://deepseek-harness.github.io/deepseek-harness/guide/plugin-market)
- [配置目录](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/config-catalog.zh.md)
- [Skills](https://github.com/deepseek-ai/deepseek-harness/blob/master/packages/skill/skill-filesystem/README.zh.md)
