# 写作与排版

本页汇总常用的 Markdown 写法。新增章节时，直接复制需要的片段即可。

## 新建页面

在 `docs/` 下创建文件，例如 `docs/chapter-01.md`：

```markdown
# 第一章：准备工作

在这里写正文。
```

然后在 `mkdocs.yml` 的 `nav` 中登记：

```yaml
nav:
  - 首页: index.md
  - 开始使用: getting-started.md
  - 第一章: chapter-01.md
```

## 标题与段落

使用 `#` 表示一级标题，`##` 表示二级标题。每页只保留一个一级标题，方便生成目录。

```markdown
## 二级标题

正文段落。段落之间空一行。
```

## 提示块

!!! tip "提示"
    适合放补充说明、经验或注意事项。

!!! warning "注意"
    适合放容易出错的操作或前置条件。

!!! danger "危险"
    适合放不可逆、可能造成数据丢失的操作。

写法：

```markdown
!!! tip "提示"
    适合放补充说明、经验或注意事项。
```

## 代码块

```python
def greet(name: str) -> str:
    return f"Hello, {name}!"


print(greet("MkDocs"))
```

写法：

````markdown
```python
def greet(name: str) -> str:
    return f"Hello, {name}!"
```
````

## 表格

| 参数 | 说明 | 默认值 |
| --- | --- | --- |
| `site_name` | 站点名称 | 必填 |
| `site_url` | 站点完整地址 | 无 |
| `theme` | 主题配置 | `material` |

## 任务列表

- [x] 创建站点骨架
- [x] 配置自动部署
- [ ] 补充第一章正文
- [ ] 校对并发布

## 链接与图片

```markdown
[站内页面](getting-started.md)
[外部链接](https://github.com/cat-yqy)

![图片说明](assets/example.png)
```

图片放在 `docs/assets/` 目录，使用相对路径引用。

## 分栏与标签页

=== "方法一"

    直接修改 `docs/index.md`。

=== "方法二"

    新建一个页面，再在导航中登记。

写法：

```markdown
=== "方法一"

    内容一。

=== "方法二"

    内容二。
```
