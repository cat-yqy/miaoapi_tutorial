# 教程文档

使用 **MkDocs Material** 构建、由 **GitHub Actions** 自动发布到 **GitHub Pages** 的教程站点。

- 在线地址：<https://cat-yqy.github.io/tutorial/>
- 源码仓库：<https://github.com/cat-yqy/tutorial>

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
├── docs/                    # 教程正文
│   ├── index.md             # 首页
│   ├── getting-started.md   # 开始使用
│   ├── writing.md           # 写作与排版
│   └── stylesheets/         # 自定义样式
├── mkdocs.yml               # 站点与导航配置
├── requirements.txt         # Python 依赖
└── .github/workflows/       # GitHub Pages 自动部署
```

## 新增教程页面

1. 在 `docs/` 下新建 `.md` 文件，例如 `docs/chapter-01.md`。
2. 在 `mkdocs.yml` 的 `nav` 中登记页面标题和路径。
3. 本地运行 `mkdocs serve` 预览。
4. 提交并推送到 `main` 分支。

## 发布方式

推送到 `main` 后，GitHub Actions 会执行 `mkdocs build --strict`，并把构建产物发布到 GitHub Pages。

首次发布前，需要到仓库的 **Settings → Pages → Build and deployment**，把 **Source** 设置为 **GitHub Actions**。

Git 远程地址使用专用别名：

```bash
git remote add origin git@github-cat-yqy:cat-yqy/tutorial.git
```
