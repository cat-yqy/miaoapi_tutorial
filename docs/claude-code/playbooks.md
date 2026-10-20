# 场景实战：把 Claude Code 用在真实任务里

下面是可以直接改一改就用的任务模板。每个场景包含：什么时候用、推荐入口、提示词、验证方式。

通用结构仍然是：

```text
目标 + 上下文 + 约束 + 完成标准 + 验证方式
```

## 场景选择表

| 场景 | 推荐入口 | 关键能力 |
| --- | --- | --- |
| 接手陌生代码库 | CLI / IDE | Plan mode、Explore 子代理 |
| 修可复现 bug | CLI | 复现、失败测试、回归验证 |
| 开发新功能 | CLI / Desktop | 需求访谈、spec、分阶段实现 |
| 重构 / 迁移 | CLI + worktree | 计划、子代理、多阶段验证 |
| 补测试 | IDE / CLI | 选区上下文、边界分析 |
| UI / 浏览器 | Desktop / Chrome | 截图、浏览器操作、`/verify` |
| 代码审查 | CLI / GitHub | `/code-review`、Ultrareview、PR |
| 并行任务 | Agent View / Teams | 后台会话、worktree、协作 |
| CI / 定时任务 | `claude -p` / Actions / `/schedule` | 非交互、权限、自动触发 |

## 1. 接手陌生代码库

**入口**：CLI 或 IDE。先只读，不要一上来就改。

```text
我要理解这个仓库，先不要改任何代码。

请完成：
1. 从入口到数据库的主要调用链
2. 每个模块的职责
3. 鉴权、错误处理、配置加载分别在哪里
4. 三个最容易改错的边界
5. 哪些目录是历史遗留，哪些是当前主路径

约束：
- 只读
- 结论带文件路径和关键函数名
- 不确定的地方标注“不确定”，不要猜
```

建议：

- 按 `Shift+Tab` 进入 Plan mode；
- 大仓库用 `Use subagents to investigate ...` 让 Explore 子代理去读；
- 把得到的架构说明整理进 `CLAUDE.md`，但只保留长期有用的部分。

验证：

- 自己点开两三个关键文件核对；
- 追问“如果我要加一个 API endpoint，应该改哪些文件？”
- 让 Claude 用编号步骤复述，确认没有跳步。

## 2. 修一个可复现的 bug

**入口**：CLI，适合“复现 → 失败测试 → 修复 → 回归”的闭环。

```text
Bug：设置页打开 Enable alerts 并保存后，刷新页面开关又复位。

复现：
1. npm run dev
2. 打开 /settings
3. 打开 Enable alerts
4. 点击 Save
5. 刷新页面，开关复位

约束：
- 不改变 API 形状
- 保持现有 UI 风格
- 先写一个能复现问题的失败测试，再修
- 修复范围尽量小

完成标准：
- 复现步骤不再出现
- 新测试修复前失败、修复后通过
- lint、类型检查和相关测试通过
- 给出根因说明和验证命令
```

验证：

- 要求展示失败测试的输出；
- 用 `/diff` 看是否只改了相关文件；
- 让它解释根因，而不只是“改了这里”。

!!! warning "不要只要补丁"
    如果 Claude 没有复现，只根据猜测改代码，bug 很可能换个路径又出现。复现步骤和失败测试比补丁更重要。

## 3. 开发一个新功能

**入口**：CLI 或 Desktop。需求模糊时，先让 Claude 访谈你。

### 先写 spec

```text
我想给团队加一个“批量归档项目”的功能。先不要写代码。
用 AskUserQuestion 详细访谈我，覆盖：
- 权限和角色
- 批量上限
- 失败回滚
- UI 交互
- 审计日志
- 测试策略

访谈结束后，把完整 spec 写到 SPEC.md，包含范围、接口、边界和验收标准。
```

然后开一个**新会话**实现 spec：

```text
按 SPEC.md 实现。先做最小垂直切片：
1. API 和权限
2. 单条归档
3. 批量入口
4. UI
5. 审计日志

每个阶段跑对应测试，偏差写回 SPEC.md。
```

为什么新会话：实现阶段上下文更干净，不会被需求访谈的细节淹没。

验证：

- 先看计划/垂直切片；
- 每阶段跑测试；
- 用 `/code-review` 审查；
- 检查权限和失败回滚，而不只是 happy path。

## 4. 重构或迁移

**入口**：CLI + worktree；长任务可以用 Agent View 或 Cloud（若账号支持）。

```text
目标：把 auth 子系统拆成 token 解析、session 加载、权限判断三层。
外部行为不变，公共 API 保持兼容，测试全部通过。

先输出分阶段计划：
- 每阶段改哪些文件
- 如何保证行为不变
- 每阶段验证命令
- 失败时如何回滚

确认计划前不要改代码。
```

隔离环境：

```bash
claude --worktree auth-refactor
```

或让 Claude 在 worktree 中工作。

推进方式：

```text
实现里程碑 1。只改计划中列出的文件。
完成后运行相关测试，并总结与计划的偏差。
```

验证：

- 每个阶段独立可测；
- 公共契约测试通过；
- diff 没有顺手改的无关文件；
- 用 `/rewind` 回滚失败尝试；
- 大迁移用子代理并行分析不同模块，但写操作分阶段。

## 5. 补测试

**入口**：IDE 选区最快；CLI 用文件路径和函数名。

```text
为 src/transform.ts 里的 invertList 写测试。

要求：
- 先读现有测试，遵循相同框架和命名
- 覆盖正常路径、空数组、重复元素、非法输入
- 不要修改被测函数
- 如果发现函数行为和文档不一致，先报告，不要直接改实现
- 运行测试并给出结果
```

验证：

- 测试是否真的断言行为，而不是只跑通；
- 让 Claude 解释每个测试覆盖的风险；
- 如果测试暴露 bug，单独开修复任务。

## 6. UI / 浏览器任务

**入口**：Desktop app 的内置浏览器、Claude in Chrome，或 CLI + 截图。

```text
根据 attached screenshot 实现设置页。

约束：
- 使用现有 React + Tailwind
- 复用现有组件，不引入新 UI 库
- 390px 移动端不横向溢出
- 交互状态完整：hover、disabled、loading、error

完成标准：
- 本地能启动
- 主要断点布局正常
- 截图对比后列出差异并修正
```

验证：

- 用 Desktop 的浏览器预览，或 `/verify` 启动并检查应用；
- 让 Claude 截图对比设计稿；
- 真实点一遍交互，不只看静态页面；
- 移动端单独检查。

!!! tip "一次只改一个视觉目标"
    “大改版”很容易让 diff 失控。每轮只改头部、间距、颜色或某个组件，确认后再继续。

## 7. 代码审查与 PR

**入口**：CLI `/code-review`；GitHub Code Review；Ultrareview（如可用）。

### 本地审查

```text
/code-review high
```

重点：

- 正确性和边界条件；
- 权限和错误处理；
- 回归风险；
- 测试是否覆盖新增行为。

应用修复：

```text
/code-review --fix
```

安全专项：

```text
/security-review
```

### PR 审查

```text
/code-review 1234
```

或把发现发到 PR：

```text
/code-review --comment
```

GitHub 上也可以让 Codex/Claude 的 PR 审查机器人参与，具体取决于仓库接入方式。

### 独立审查

用一个全新的子代理审查 diff，避免“自己改、自己判”：

```text
Use a subagent to review the current diff against SPEC.md.
Only report gaps that affect correctness or the stated requirements.
Include file and line references.
```

验证：

- 逐条判断 finding 是否成立；
- 机械检查交给 CI；
- 高优先级问题要求具体触发条件；
- 不要为了“清零 findings”过度防御。

## 8. 并行任务与 Agent Teams

### 两个独立任务：worktree

```bash
# 终端 1
claude --worktree feature-auth

# 终端 2
claude --worktree fix-billing
```

每个会话有独立工作目录，不会互相覆盖。

### 多个后台任务：Agent View

```bash
claude agents
```

派发：

- 修一个 bug；
- 审查一个 PR；
- 调查 flaky test。

然后用 `claude attach <id>` 或界面附加到某个会话。Agent View 是研究预览，注意额度消耗。

### 需要协作：Agent Teams

仅在确实需要多个 agent 讨论、互相验证时启用：

```text
Spawn three teammates to investigate this performance regression:
one on database queries, one on frontend rendering, one on caching.
Have them share findings and challenge each other's conclusions.
```

适合：

- 竞争假设调试；
- 多角度研究；
- 跨层改动，各自负责独立模块。

不适合：

- 同文件并行编辑；
- 强顺序重构；
- 小任务。

## 9. CI、脚本与定时任务

### 非交互

```bash
claude -p "检查最近的提交，列出风险点"
claude -p "列出所有 API 端点" --output-format json
```

批处理时限制工具：

```bash
claude -p "迁移这个文件" \
  --allowedTools "Edit,Bash(git commit *)" \
  --permission-mode dontAsk
```

### GitHub Actions

在 Issue/PR 中：

```text
@claude implement this feature based on the issue description
```

工作流使用：

```yaml
- uses: anthropics/claude-code-action@v1
  with:
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

### 定时任务

- 当前会话轮询：`/loop 5m check if the deploy finished`
- 云端定时任务：`/schedule 每天 9 点审查昨天合并的 PR`
- 桌面定时任务：需要本地文件和工具时使用

云端定时任务依赖 claude.ai 和 GitHub；MiaoAPI 网关模式下通常不可用。

## 10. 文档、发布说明与 Artifacts

```text
根据最近 20 个提交和合并的 PR，写一份面向用户的 release notes。

要求：
- 按 feat / fix / perf / docs 分组
- 面向用户描述影响，不复制 commit message
- 单独列出破坏性变更和升级步骤
- 每条结论可追溯到 commit 或 PR
- 输出 Markdown，并列出需要人工确认的条目
```

需要可视化时，可以要求生成 Artifact：

```text
把这个 PR 的改动做成一个带注释的 diff 页面 artifact。
```

Artifacts 依赖 claude.ai 登录；不可用时让 Claude 生成本地 HTML 或 Markdown。

## 通用验收清单

- [ ] 改动范围和任务一致
- [ ] 没有密钥、日志、调试代码
- [ ] 测试/构建/lint 真的跑过
- [ ] 边界和失败路径有覆盖
- [ ] diff 有人类可读的解释
- [ ] 文档、类型、配置同步更新
- [ ] 需要人工确认的点已列出

## 官方参考

- [Common workflows](https://code.claude.com/docs/en/common-workflows)
- [Best practices](https://code.claude.com/docs/en/best-practices)
- [Agent teams](https://code.claude.com/docs/en/agent-teams)
- [Local code review](https://code.claude.com/docs/en/code-review-local)
- [Ultrareview](https://code.claude.com/docs/en/ultrareview)
- [GitHub Actions](https://code.claude.com/docs/en/github-actions)
- [Agent view](https://code.claude.com/docs/en/agent-view)
- [Worktrees](https://code.claude.com/docs/en/worktrees)
