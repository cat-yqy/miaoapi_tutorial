# 场景实战：把 Codex 用在真实任务里

这一页给的是“可以直接改一改就用”的任务模板。每个场景都包含：适合什么情况、用哪个入口、提示词怎么写、怎么验收。

先记住一个通用结构：

```text
目标 + 上下文 + 约束 + 完成标准
```

下面所有模板都围绕这四件事展开。

## 场景选择表

| 场景 | 推荐入口 | 关键能力 |
| --- | --- | --- |
| 接手陌生代码库 | IDE / CLI | 代码理解、只读探索、计划 |
| 修可复现 bug | CLI | 复现、调试、回归测试 |
| 开发小功能 | CLI / IDE | 计划、实现、测试 |
| 重构或迁移 | CLI + Goal / Cloud | 多阶段计划、worktree、并行 |
| 补测试 | IDE / CLI | 选中代码、边界分析 |
| 还原 UI | app / IDE | 图片输入、浏览器预览、实时迭代 |
| 代码审查 / PR | CLI / GitHub | `/review`、PR 评论 |
| CI 与定时任务 | `exec` / Action / Automations | 非交互、JSON、调度 |

## 1. 接手陌生代码库

**适合**：新入职、接手遗留系统、需要快速理解请求链路。

**入口**：IDE 扩展最快；CLI 更适合留下完整记录。

```text
我要理解这个仓库的请求链路，但先不要改任何代码。

请完成：
1. 画出从入口到数据库的主要调用链
2. 说明每个模块的职责
3. 列出 3 个最容易改错的边界
4. 指出哪些目录是历史遗留、哪些是当前主路径

约束：
- 只读，不要修改文件
- 结论必须带文件路径和关键函数名
- 不确定的地方明确标注“不确定”，不要猜

完成标准：
- 我能根据你的说明找到对应代码
- 每个结论都有文件依据
```

验收：

- 让 Codex 用“编号步骤 + 文件列表”复述一遍；
- 自己点开两三个关键文件核对；
- 追问“如果我要改 X，应该先看哪里？”

!!! tip "先只读，后动手"
    接手代码库时先保持 Read-only。理解阶段最忌讳 agent 一边解释一边改文件。

## 2. 修一个可复现的 bug

**适合**：有稳定复现步骤、能跑测试或启动服务。

**入口**：CLI，适合“复现 → 修 → 验证”的 tight loop。

```text
Bug：设置页打开“Enable alerts”并保存后，刷新页面开关又恢复原状。

复现：
1. npm run dev
2. 打开 /settings
3. 打开 Enable alerts
4. 点击 Save
5. 刷新页面，开关复位

约束：
- 不改变 API 形状
- 保持现有 UI 风格
- 先复现，再提出最小修复
- 加一个回归测试

完成标准：
- 复现步骤不再出现
- 新增测试能在修复前失败、修复后通过
- lint 和相关测试通过
- 最后列出修改文件和验证命令
```

验收：

- 要求 Codex 展示修复前后的测试结果；
- 用 `/diff` 检查是否只改了相关文件；
- 让它解释根因，而不只是“改了这里就好了”。

!!! warning "不要只要补丁"
    如果 Codex 没有真正复现，只根据猜测改代码，bug 很可能换个路径又出现。复现步骤比补丁更重要。

## 3. 开发一个小功能

**适合**：需求相对明确，能拆成一个可交付的垂直切片。

**入口**：CLI 或 IDE。IDE 适合边看现有实现边写。

```text
目标：给 `list` 命令增加 `--json` 参数，输出机器可读的 JSON。

上下文：
- 入口在 src/cli/list.ts
- 现有格式化逻辑在 src/formatters/list.ts
- 测试参考 test/cli/list.test.ts

约束：
- 不改变默认的人类可读输出
- 不新增运行时依赖
- 保持现有退出码语义
- 遵循仓库现有错误处理方式

完成标准：
- `list --json` 输出合法 JSON
- 覆盖空列表、正常列表、错误情况
- lint、类型检查、相关测试通过

先读代码并给出简短计划，确认后再实现。
```

验收：

- 先看计划，确认没有理解偏差；
- 要求先做最小垂直切片，不要一次改十个文件；
- 跑真实命令，不只看单元测试；
- `/review` 一次，再提交。

## 4. 重构或迁移

**适合**：跨模块、多阶段、不能改变外部行为。

**入口**：CLI + `/goal`，长任务可交给 Cloud 或 worktree。

```text
/goal 把 auth 子系统拆分为 token 解析、session 加载、权限判断三层。
要求外部行为不变，公共 API 保持兼容，测试全部通过。

先输出分阶段计划：
- 每个阶段改哪些文件
- 如何保证行为不变
- 每阶段如何验证
- 失败时如何回滚

在我确认计划前不要改代码。
```

计划确认后，一次只推进一个里程碑：

```text
实现里程碑 1。只改计划中列出的文件。
完成后运行相关测试，并总结与计划的偏差。
```

验收：

- 每个阶段都能独立运行测试；
- 公共 API 的契约测试通过；
- diff 中没有顺手改的无关文件；
- 用 worktree 或 Cloud 隔离长任务，避免影响主工作区。

!!! tip "重构最怕“顺手优化”"
    在提示词里明确写“不要改与目标无关的代码”。否则 agent 很容易在重构时顺手重命名、换库或调整格式，扩大 review 范围。

## 5. 补测试

**适合**：已有函数/模块，但测试不足。

**入口**：IDE 选中代码最快；CLI 用文件路径和函数名。

```text
为 @src/transform.ts 里的 invertList 写测试。

要求：
- 先阅读现有测试，遵循同样的框架和命名
- 覆盖正常路径、空数组、重复元素、非法输入
- 不要修改被测函数
- 如果发现函数行为和文档不一致，先告诉我，不要直接改实现

完成标准：
- 新测试能独立运行
- 覆盖上述边界
- 给出运行命令和结果
```

验收：

- 看测试是否真的断言了行为，而不是只跑通；
- 让 Codex 解释每个测试覆盖的风险；
- 如果测试暴露了 bug，单独开一个修复任务，不要混在一起。

## 6. 还原 UI 或迭代前端

**适合**：有设计稿/截图，或需要边跑 dev server 边调样式。

**入口**：Codex app（内置浏览器）或 IDE；CLI 适合快速原型。

```text
根据 attached screenshot 实现设置页。

约束：
- 使用项目现有的 React + Tailwind
- 复用现有组件，不要引入新的 UI 库
- 保持响应式，移动端 390px 下不横向溢出
- 颜色、间距、字号尽量贴近截图

完成标准：
- 页面能本地启动
- 主要断点下布局正常
- 交互状态（hover、disabled、loading）完整
- 给出本地预览地址和需要人工确认的差异
```

如果使用 app：

1. 打开内置浏览器预览本地页面；
2. 直接在页面上标注要改的元素；
3. 让 Codex 只改被标注的区域；
4. 每轮只提一个视觉目标，避免“大改版”。

验收：

- 桌面和移动端都看一遍；
- 检查真实交互，不只看静态截图；
- 对不熟悉的视觉细节，要求它说明取舍。

## 7. 代码审查与 PR

**适合**：提交前自查、PR 审查、团队质量门禁。

**入口**：CLI `/review`；GitHub `@codex review`；桌面端 Code Review 面板。

### 本地审查

```text
/review 重点检查：
- 权限校验是否覆盖所有分支
- 错误处理是否会吞异常
- 是否有回归风险
- 测试是否覆盖新增行为
```

`/review` 不会修改工作区；它输出按优先级排序的发现。修完后可以再跑一次确认。

### GitHub PR 审查

```text
@codex review 重点看并发安全和错误分支
```

团队可以把审查偏好写进 `AGENTS.md` 的 `## Code Review Rules`，让审查标准稳定下来。

验收：

- 逐条判断 findings 是否成立，不要盲信；
- 对高优先级问题要求给出具体文件、行号和触发条件；
- 机械检查交给 CI，不要让 Codex 替代测试和分支保护；
- 安全敏感仓库可以再加 Security 扫描，但它也不替代人工安全评审。

## 8. CI、脚本与定时任务

**适合**：重复执行、需要机器可读结果、无人值守。

**入口**：`codex exec`、GitHub Action、Automations。

### 用 `exec` 做一次性自动化

```bash
codex exec --json "检查最近的提交，列出可能的风险点" | jq
```

需要固定字段时用 schema：

```bash
codex exec "提取项目元数据" \
  --output-schema ./schema.json \
  -o ./project-metadata.json
```

### 用 GitHub Action 做 PR 审查

```yaml
- uses: openai/codex-action@v1
  with:
    openai-api-key: ${{ secrets.OPENAI_API_KEY }}
    prompt-file: .github/codex/prompts/review.md
    output-file: codex-output.md
```

### 用 Automations 做周期任务

适合：每日提交摘要、CI 失败巡检、依赖风险扫描、发布说明草稿。

原则：

- 先在普通对话里手动跑通；
- 再用 Skill 固化步骤；
- 最后才设定时；
- 无人值守任务使用最小权限和独立 worktree。

## 9. 文档、发布说明与知识工作

**适合**：README、迁移指南、release notes、事故总结、会议纪要。

```text
根据最近 20 个提交和合并的 PR，写一份面向用户的 release notes。

要求：
- 按 feat / fix / perf / docs 分组
- 面向用户描述影响，不要复制 commit message
- 单独列出破坏性变更和升级步骤
- 所有结论必须能追溯到具体提交或 PR
- 输出 Markdown，并列出需要人工确认的条目
```

验收：

- 抽查三条结论是否真的对应提交；
- 检查有没有把内部重构写成用户可见变化；
- 发布前由人确认破坏性变更。

## 通用验收清单

无论哪个场景，提交前都问自己：

- [ ] 改动范围是否和任务一致？
- [ ] 有没有不该出现的密钥、日志、调试代码？
- [ ] 测试/构建/lint 是否真的跑过？
- [ ] 失败路径和边界条件是否覆盖？
- [ ] diff 是否有人类可读的解释？
- [ ] 是否还需要更新文档、类型或配置？

## 官方参考

- [Workflows](https://developers.openai.com/codex/workflows)
- [Best practices](https://developers.openai.com/codex/learn/best-practices)
- [Code review](https://developers.openai.com/codex/integrations/github)
- [Non-interactive mode](https://developers.openai.com/codex/noninteractive)
- [GitHub Action](https://developers.openai.com/codex/github-action)
- [Automations](https://developers.openai.com/codex/app/automations)
