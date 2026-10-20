# 场景实战：把 OpenCode 用在真实任务里

下面是可以直接改一改就用的任务模板。每个场景包含：什么时候用、推荐入口、提示词、验证方式。

通用结构：

```text
目标 + 上下文 + 约束 + 完成标准 + 验证方式
```

## 场景选择表

| 场景 | 推荐入口 | 关键能力 |
| --- | --- | --- |
| 接手陌生代码库 | TUI / IDE | Plan、Explore 子代理 |
| 修可复现 bug | TUI | 复现、测试、undo |
| 开发新功能 | TUI | Plan → Build |
| 重构 / 迁移 | TUI + git worktree | 分阶段计划、子代理 |
| 补测试 | IDE / TUI | 选区上下文、测试命令 |
| UI / 前端 | TUI + 浏览器 | 截图、`opencode web` |
| 代码审查 | TUI / GitHub | diff 审查、`/oc` |
| 并行调研 | TUI / Web | 子代理、多 session |
| CI / 自动化 | `opencode run` / GitHub | 非交互、JSON 输出 |

## 1. 接手陌生代码库

**入口**：TUI 或 IDE，先 Plan，只读。

```text
我要理解这个仓库，先不要改代码。

请完成：
1. 从入口到数据库的主要调用链
2. 每个模块的职责
3. 鉴权、错误处理、配置加载分别在哪里
4. 三个最容易改错的边界

约束：
- 只读
- 结论带文件路径和关键函数名
- 不确定的地方标注“不确定”
```

大仓库可以：

```text
@explore 搜索所有处理 OAuth 回调的文件和测试，只返回关键文件与职责。
```

验证：

- 自己点开关键文件核对；
- 追问“加一个 API endpoint 要改哪些文件？”
- 把长期有用的架构说明写进 `AGENTS.md`。

## 2. 修一个可复现的 bug

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

- 要求展示失败测试输出；
- 检查 diff 是否只改相关文件；
- 不满意用 `/undo` 回滚后重试。

## 3. 开发一个新功能

先用 Plan：

```text
我要给团队加一个“批量归档项目”的功能。先不要写代码。
阅读现有项目、权限和审计逻辑，给出：
- 数据模型和接口
- 权限与角色
- 批量上限
- 失败回滚
- UI 交互
- 测试计划
```

确认后切回 Build：

```text
按计划实现。先做最小垂直切片：
1. API 和权限
2. 单条归档
3. 批量入口
4. UI
5. 审计日志

每个阶段跑对应测试，偏差写回计划。
```

验证：

- 每阶段可独立运行；
- 测试覆盖 happy path 和失败路径；
- 权限和审计单独检查；
- 提交前审查 diff。

## 4. 重构或迁移

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

长任务建议手动开一个 Git worktree：

```bash
git worktree add ../auth-refactor -b auth-refactor
cd ../auth-refactor
opencode
```

OpenCode 没有内置 worktree 管理，但 Git worktree 本身就能隔离两个 session。

验证：

- 每阶段独立可测；
- 公共契约测试通过；
- 没有顺手改的无关文件；
- `/undo` 只用于小步回滚，大回滚用 Git。

## 5. 补测试

```text
为 src/transform.ts 里的 invertList 写测试。

要求：
- 先读现有测试，遵循相同框架和命名
- 覆盖正常路径、空数组、重复元素、非法输入
- 不要修改被测函数
- 如果发现函数行为和文档不一致，先报告

完成标准：
- 新测试能独立运行
- 覆盖上述边界
- 给出运行命令和结果
```

验证：

- 测试是否真的断言行为；
- 解释每个测试覆盖的风险；
- 如果测试暴露 bug，单独开修复任务。

## 6. UI / 前端

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

可以用 `opencode web` 在浏览器里看同一会话，或在本地跑 dev server 后让 agent 对照截图。

验证：

- 桌面和移动端都看；
- 真实点一遍交互；
- 每轮只改一个视觉目标，避免 diff 失控。

## 7. 代码审查与 PR

OpenCode 没有 Claude Code 那种内置 `/code-review`，可以直接让 agent 审查：

```text
审查当前工作区 diff，重点看：
- 正确性和边界条件
- 权限和错误处理
- 回归风险
- 测试是否覆盖新增行为

只报告影响正确性或需求的问题，给出文件和行号。
```

需要机器可读结果时，用 `opencode run`：

```bash
opencode run --format json "审查当前 diff，输出 JSON 数组，每项包含 file、line、severity、issue"
```

GitHub PR 里：

```text
/oc review this PR for security issues
```

验证：

- 逐条判断 finding 是否成立；
- 机械检查交给 CI；
- 高优先级问题要求触发条件；
- 不要为了清零 findings 过度防御。

## 8. 并行调研与多会话

适合：

- 同时研究多个模块；
- 一个 session 修 bug，另一个查测试；
- 用不同模型比较方案。

子代理：

```text
@explore 找出所有和 billing 相关的入口、测试和配置。
@general 独立分析这个性能问题，给出三种假设和验证方式。
```

多个独立任务可以开多个终端，或在 Web 界面管理多个 session。每个 session 有独立上下文，但都消耗额度。

如果要隔离文件修改，手动用 `git worktree`：

```bash
git worktree add ../fix-billing -b fix-billing
```

## 9. GitHub / GitLab 自动化

### GitHub

```bash
opencode github install
```

然后在 Issue/PR 评论：

```text
/oc 解释这个 issue 并给出修复方案
/opencode 实现这个功能并提 PR
```

支持：

- Issue 评论；
- PR 评论；
- PR 代码行评论；
- 定时任务；
- 手动 workflow_dispatch。

安全要点：

- 限制谁能触发；
- 保护 API Key；
- 不要让不可信 PR 文本直接驱动高权限任务；
- runner 只给必要权限。

### GitLab

通过 GitLab CI 组件运行，或在评论里 `@opencode` 使用 GitLab Duo 集成。凭据存放在 CI variables。

## 10. 固化流程：Command、Skill、Tool

### Command

固定动作，显式触发：

```markdown
---
description: 运行测试并分析失败
agent: build
---

运行完整测试套件，展示失败用例。
只分析失败原因，不要直接修改代码。
```

使用 `/test`。

### Skill

需要模型按场景自动加载的知识/流程：

```markdown
---
name: release-notes
description: 根据最近的 PR 和提交生成发布说明
metadata:
  opencode/slash: "true"
---

1. 读取最近合并记录。
2. 按 feat / fix / docs 分类。
3. 面向用户描述影响。
```

### Custom Tool

需要确定性执行逻辑时：

```ts
import { tool } from "@opencode-ai/plugin"

export default tool({
  description: "Query the project database",
  args: { query: tool.schema.string() },
  async execute({ query }) {
    return `Executed: ${query}`
  },
})
```

选择原则：

- 只是提示词：Command；
- 需要按场景自动加载：Skill；
- 需要真实执行逻辑：Custom Tool；
- 需要订阅生命周期事件：Plugin。

## 通用验收清单

- [ ] 改动范围和任务一致
- [ ] 没有密钥、日志、调试代码
- [ ] 测试/构建/lint 真的跑过
- [ ] 边界和失败路径有覆盖
- [ ] diff 有人类可读的解释
- [ ] 文档、类型、配置同步更新
- [ ] 需要人工确认的点已列出

## 官方参考

- [Intro](https://opencode.ai/docs/)
- [Agents](https://opencode.ai/docs/agents/)
- [Commands](https://opencode.ai/docs/commands/)
- [Skills](https://opencode.ai/docs/skills/)
- [Custom Tools](https://opencode.ai/docs/custom-tools/)
- [GitHub](https://opencode.ai/docs/github/)
- [GitLab](https://opencode.ai/docs/gitlab/)
