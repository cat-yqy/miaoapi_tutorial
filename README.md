# MiaoAPI 教程

MiaoAPI 使用教程，覆盖 Codex CLI / App、Claude Code、OpenCode、Grok Build、GPT-Image-2 图片生成和常见报错处理，并附带完整的 Claude Code 与 Codex 功能指南。

- 在线地址：<https://cat-yqy.github.io/miaoapi_tutorial/>
- 源码仓库：<https://github.com/cat-yqy/miaoapi_tutorial>
- MiaoAPI 官网：<https://miaoapi.site>

## 内容结构

- 安装 Codex CLI / App
- 在 Codex 中配置喵API（推荐手动配置，另有 Cockpit Tools、CC Switch）
- Claude Code 接入
- OpenCode 接入
- Grok Build 接入
- 使用 GPT-Image-2 生成图片
- 常见报错与解决方法
- Claude Code 指南：功能全景、核心机制、日常使用、进阶能力、场景实战、最佳实践
- Codex 指南：功能全景、核心机制、日常使用、进阶能力、场景实战、最佳实践

## 本地预览

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
mkdocs serve
```

浏览器打开 <http://127.0.0.1:8000/>。

## 目录结构

```text
.
├── docs/
│   ├── index.md                 # 首页
│   ├── guide/                   # 教程正文
│   │   ├── install.md
│   │   ├── configure.md
│   │   ├── image-generation.md
│   │   ├── claude-code.md
│   │   ├── opencode.md
│   │   ├── grok-build.md
│   │   └── troubleshooting.md
│   ├── claude-code/             # Claude Code 功能与实战指南
│   │   ├── overview.md
│   │   ├── core-concepts.md
│   │   ├── daily-workflow.md
│   │   ├── advanced.md
│   │   ├── playbooks.md
│   │   └── best-practices.md
│   ├── codex/                   # Codex 功能与实战指南
│   │   ├── overview.md
│   │   ├── core-concepts.md
│   │   ├── daily-workflow.md
│   │   ├── advanced.md
│   │   ├── playbooks.md
│   │   └── best-practices.md
│   ├── assets/miaoapi/          # 教程截图（WebP）
│   └── stylesheets/             # 自定义样式
├── mkdocs.yml                   # 站点与导航配置
├── requirements.txt             # Python 依赖
└── .github/workflows/           # GitHub Pages 自动部署
```

## 更新教程

1. 在 `docs/guide/` 下新建或编辑 Markdown 文件。
2. 在 `mkdocs.yml` 的 `nav` 中登记页面标题和路径。
3. 图片放到 `docs/assets/miaoapi/`，建议压缩为 WebP 后再引用。
4. 本地运行 `mkdocs serve` 预览。
5. 提交并推送到 `main` 分支。

## 发布方式

推送到 `main` 后，GitHub Actions 会执行 `mkdocs build --strict`，并把构建产物发布到 GitHub Pages。

首次发布前，需要到仓库的 **Settings → Pages → Build and deployment**，把 **Source** 设置为 **GitHub Actions**。

Git 远程地址使用专用别名：

```bash
git remote add origin git@github-cat-yqy:cat-yqy/miaoapi_tutorial.git
```
