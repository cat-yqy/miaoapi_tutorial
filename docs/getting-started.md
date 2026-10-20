# 开始使用

本站使用 [MkDocs Material](https://squidfunk.github.io/mkdocs-material/) 构建。所有教程正文都是 `docs/` 目录下的 Markdown 文件。

## 本地预览

首次使用时，在项目根目录创建虚拟环境并安装依赖：

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

启动本地预览服务：

```bash
mkdocs serve
```

浏览器打开 <http://127.0.0.1:8000/>。修改 `docs/` 中的文件后，页面会自动刷新。

## 目录结构

```text
.
├── docs/                    # 教程正文
│   ├── index.md             # 首页
│   ├── getting-started.md   # 本页
│   ├── writing.md           # 写作与排版
│   └── stylesheets/         # 自定义样式
├── mkdocs.yml               # 站点与导航配置
├── requirements.txt         # Python 依赖
└── .github/workflows/       # 自动部署配置
```

## 发布流程

每次推送到 `main` 分支，GitHub Actions 会自动完成：

1. 安装 `requirements.txt` 中的依赖。
2. 执行 `mkdocs build --strict` 构建静态站点。
3. 将 `site/` 发布到 GitHub Pages。

也可以在仓库的 **Actions** 页面手动触发 `Deploy docs` 工作流。

## 常见问题

!!! question "本地端口被占用怎么办？"
    可以指定其他端口，例如 `mkdocs serve -a 127.0.0.1:8001`。

!!! question "构建时报错怎么办？"
    先执行 `mkdocs build --strict` 查看具体错误。常见原因是导航里登记了不存在的文件，或页面内链接写错。
