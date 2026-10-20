# 在 Codex 中配置喵API

在 [MiaoAPI](https://miaoapi.site) 创建 API Key 后，从下面三种方式中任选一种。三种方式最终效果一致，推荐直接修改配置文件（方法一）。

!!! tip "先创建密钥"
    登录 MiaoAPI 后进入密钥管理，创建一个 `codex` 分组的密钥。如果已购买套餐，请选择套餐对应的专属分组。

## 方法一：修改配置文件（推荐）

### 1. 创建 API Key

登录 MiaoAPI，进入密钥管理，创建一个 `codex` 分组的密钥；如果已购买套餐，请选择套餐对应的专属分组。

![操作截图](../assets/miaoapi/img-11.webp)

点击“创建密钥”。

![操作截图](../assets/miaoapi/img-12.webp)

选择 `codex` 分组。

![操作截图](../assets/miaoapi/img-13.webp)

![操作截图](../assets/miaoapi/img-14.webp)

创建完成后，在密钥列表点击“使用密钥”。

![操作截图](../assets/miaoapi/img-38.webp)

平台弹窗会显示 `config.toml` 和 `auth.json` 两个文件的示例。`config.toml` 不要直接照抄，按下一步替换；`auth.json` 中的 API Key 保留。

### 2. 替换 config.toml

将 `~/.codex/config.toml` 的全部内容替换为下面这份推荐配置。替换前建议先备份原文件。

```toml
approval_policy = "never"
disable_response_storage = true
include_apply_patch_tool = true
include_plan_tool = true
model = "gpt-6.1-sol"
model_provider = "OpenAI"
model_reasoning_effort = "xhigh"
model_reasoning_summary = "detailed"
model_reasoning_summary_format = "experimental"
model_verbosity = "medium"
plan_mode_reasoning_effort = "xhigh"
preferred_auth_method = "apikey"
project_doc_fallback_filenames = ["CLAUDE.md", "AGENTS.md"]
project_doc_max_bytes = 65536
sandbox_mode = "danger-full-access"
suppress_unstable_features_warning = true
web_search = "live"

[analytics]
  enabled = false

[features]
  apply_patch_freeform = true
  multi_agent = true
  plan_tool = true
  remote_compaction_v2 = false
  responses_websockets = true
  shell_snapshot = true
  unified_exec = true
  view_image_tool = true

[model_providers]
  [model_providers.OpenAI]
    base_url = "https://miaoapi.site"
    name = "custom"
    requires_openai_auth = true
    supports_websockets = false
    wire_api = "responses"

[tui]
  status_line = ["model-with-reasoning", "current-dir", "context-used", "five-hour-limit", "weekly-limit"]
  status_line_use_colors = true
  [tui.model_availability_nux]
    "gpt-5.5" = 1
    "gpt-5.6-sol" = 2
```

!!! warning "注意权限设置"
    这份配置包含 `approval_policy = "never"` 和 `sandbox_mode = "danger-full-access"`。Codex 将不再逐步确认操作，并拥有当前环境的完整访问权限。请只在你信任的目录中使用；如需更保守，请自行调整这两项。

### 3. 配置 auth.json

把刚创建的 API Key 写入 `~/.codex/auth.json`（如果平台弹窗已经生成了这个文件，保留其内容即可）：

```json
{
  "OPENAI_API_KEY": "sk-你的密钥"
}
```

### 4. 找到 .codex 目录

#### macOS / Linux 配置

打开访达，进入 Home 目录（小房子图标）。

![操作截图](../assets/miaoapi/img-40.webp)

`.codex` 默认是隐藏文件夹。按 `Command + Shift + .` 显示隐藏文件，然后进入 `.codex`。

![操作截图](../assets/miaoapi/img-41.webp)

如果仍然找不到 `.codex`，先打开一次 Codex CLI 或 Codex App 让它初始化。

里面应该有 `config.toml` 和 `auth.json`；没有就手动创建同名文件。

![操作截图](../assets/miaoapi/img-42.webp)

保存两个文件后，在终端运行：

```bash
codex
```

或者直接打开 Codex App。修改配置后记得重启。

后续可以在 Codex CLI 输入 `/model`，或在 Codex App 的对话框下方切换模型。推荐配置默认使用 `gpt-6.1-sol`，实际可用模型以账号权限为准。

![操作截图](../assets/miaoapi/img-43.webp)

#### Windows 配置

按下 `Win + R`，输入 `%userprofile%\.codex` 并回车，即可打开 `.codex` 目录。

![操作截图](../assets/miaoapi/img-44.webp)

感谢您对喵API的支持！

## 方法二：CC Switch

<https://github.com/farion1231/cc-switch>

CC Switch 也可以切换不同中转站或官方订阅，保持本地长久会话不被破坏。

首先安装CC switch

![操作截图](../assets/miaoapi/img-20.webp)

找到releases，点击这里弹出来的最新版本

![操作截图](../assets/miaoapi/img-21.webp)

下滑找到相对应的版本

![操作截图](../assets/miaoapi/img-22.webp)

这里绿框是linux，蓝框是macos，红框是windows，大家根据自己的电脑系统下载安装即可。

![操作截图](../assets/miaoapi/img-23.webp)

安装完后打开一下，你就可以进入CC swtich啦～（这里dz已经配置好了）

![操作截图](../assets/miaoapi/img-24.webp)

下面讲一下如何导入我们的某个密钥配置

首先进入你的账号，然后创建API key。

![操作截图](../assets/miaoapi/img-11.webp)

这里点击“创建密钥”

![操作截图](../assets/miaoapi/img-12.webp)

这里选择codex分组即可（如您之后订阅了套餐，请新建密钥选择套餐的专属分组）

![操作截图](../assets/miaoapi/img-13.webp)

![操作截图](../assets/miaoapi/img-14.webp)

接着点击“导入到CCS”这个按钮。

![操作截图](../assets/miaoapi/img-25.webp)

允许即可！

![操作截图](../assets/miaoapi/img-26.webp)

这里点“导入”确定。

![操作截图](../assets/miaoapi/img-27.webp)

OK，我们现在有两个配置了。

![操作截图](../assets/miaoapi/img-28.webp)

可以改一下名，比如我把刚导入的改为test，然后点击启动，就可以自动切换密钥啦！（这里的查询失败是bug，不影响）

![操作截图](../assets/miaoapi/img-29.webp)

![操作截图](../assets/miaoapi/img-30.webp)

如果您还有官方订阅（或其他中转站），可以点击右上角的➕号

![操作截图](../assets/miaoapi/img-31.webp)

选择Openai Official

![操作截图](../assets/miaoapi/img-32.webp)

点击添加

![操作截图](../assets/miaoapi/img-33.webp)

![操作截图](../assets/miaoapi/img-34.webp)

再点击启用官方订阅。

![操作截图](../assets/miaoapi/img-35.webp)

随后打开中转，随便找个文件夹，输入codex logout。

![操作截图](../assets/miaoapi/img-36.webp)

出现successfully logged out之后，再输入codex，利用网页认证登录您的gpt账号即可，会话将不会消失。

![操作截图](../assets/miaoapi/img-37.webp)

如您需要切换回中转站，可以点击中转站的启用即可。

## 方法三：Cockpit Tools

<https://github.com/jlcodes99/cockpit-tools>

Cockpit Tools 可以方便地切换不同中转站或官方订阅，保持本地长久会话不被破坏。

首先安装Cockpit tools。

![操作截图](../assets/miaoapi/img-01.webp)

找到releases，这里点击弹出来的最新版本

![操作截图](../assets/miaoapi/img-02.webp)

这里红色是macos版本，黄色是linux，蓝色是windows版本。

![操作截图](../assets/miaoapi/img-03.webp)

下载完成后打开，这里以macos为例。

![操作截图](../assets/miaoapi/img-04.webp)

点击左边这个Openai标识。

![操作截图](../assets/miaoapi/img-05.webp)

点击添加账号

![操作截图](../assets/miaoapi/img-06.webp)

点击api key。

![操作截图](../assets/miaoapi/img-07.webp)

选择自定义。

![操作截图](../assets/miaoapi/img-08.webp)

下拉，红色框填API key（需要去网页密钥管理创建），绿色框填https://miaoapi.site/，黄色框随便填，比如miaoapi。

![操作截图](../assets/miaoapi/img-09.webp)

![操作截图](../assets/miaoapi/img-10.webp)

下面讲一下怎么创建API key。

首先进入你的账号，然后创建API key。

![操作截图](../assets/miaoapi/img-11.webp)

这里点击“创建密钥”

![操作截图](../assets/miaoapi/img-12.webp)

这里选择codex分组即可（如您之后订阅了套餐，请新建密钥选择套餐的专属分组）

![操作截图](../assets/miaoapi/img-13.webp)

![操作截图](../assets/miaoapi/img-14.webp)

点击红色框的复制可以复制密钥，点击蓝色框的复制可以复制api端点，也即站点。

![操作截图](../assets/miaoapi/img-15.webp)

下面讲一下怎么切换官方订阅。

还是一样的，点击添加账号。

![操作截图](../assets/miaoapi/img-16.webp)

点击红色框oauth授权，再点击从浏览器打开（或者复制之后到浏览器打开）

![操作截图](../assets/miaoapi/img-17.webp)

![操作截图](../assets/miaoapi/img-18.webp)

一直登录即可，之后授权完会出现“返回失败”界面，此时需要手动复制浏览器顶部“返回链接”（http://localhost开头的一串链接）到此框里即可，然后点我已授权，继续。

![操作截图](../assets/miaoapi/img-19.webp)
