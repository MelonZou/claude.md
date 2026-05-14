# Caveman CLAUDE.md

> 一套让 Claude **输出 token 大幅缩减** 的配置方案。同等技术准确度，回复体积砍到原来的 1/3。

## 中心思想

**Claude 默认很啰嗦。**

- 开头铺垫：`Sure! I'd be happy to help you with that...`
- 中间解释：`The reason this works is because...`
- 结尾总结：`To summarize what we just did...`
- 充斥填充词：`just / really / basically / actually / simply`

这些字 **不传递任何技术信息**，但每个字都在烧你的 output token —— 而 output token 通常比 input 贵 **3~5 倍**。

本仓库通过一份精心设计的 `CLAUDE.md` 全局规则，**强制 Claude 用 Caveman 风格输出**：

> `[对象] [动作] [原因]. [下一步].`

技术内容一字不少，废话一字不留。

## 省了多少

| 场景 | 原始输出 | Caveman 输出 | 缩减 |
|------|---------|------------|------|
| 解释一个 bug | ~180 字 | ~45 字 | **75%** |
| 给出修复方案 | ~250 字 | ~80 字 | **68%** |
| 多步骤指南 | ~600 字 | ~200 字 | **66%** |

按月度对话量估算，长期使用 Claude API / Claude Code 的开发者，**输出账单可降 50%~70%**。

## 对比示例

**问：React 组件为什么重复渲染？**

普通输出（≈ 110 token）：
> Sure! Great question. The reason your React component is re-rendering on every parent update is likely because you're passing an inline object as a prop. Each render creates a new object reference, which React's reconciliation treats as a changed prop, triggering a re-render of the child. To fix this, you can wrap the object in `useMemo`...

Caveman 输出（≈ 25 token）：
> 内联对象 prop 每渲染生成新引用 = 子组件重渲染。`useMemo` 包住即可。

**信息量完全相同，token 砍掉 77%。**

## 工作原理

1. `~/.claude/CLAUDE.md` 全局规则 —— 每次会话自动加载，作为「主索引」
2. 规则强制约束输出风格：删冠词、删客套、删铺垫、删总结、保留技术术语与代码块
3. 触发词 / `/caveman` 斜杠命令切换强度（lite / full / ultra）
4. 配套 Codex 协作流，把代码落地这种「必须详细」的工作交给另一个 Agent，Claude 只做高密度判断输出
5. 主 CLAUDE.md 拆出两份独立规则文件，按场景按需加载，避免每次会话都把所有规则塞进上下文

## 全局规则结构

主文件 `~/.claude/CLAUDE.md` 现已模块化，章节如下：

| 章节 | 内容 | 加载方式 |
|------|------|---------|
| 顶部 caveman 风格规则 | 输出精简、禁止冗余 | 会话启动自动加载 |
| §0 角色分工与协作规范 | Claude 不写代码、全交 Codex、--watch 询问、Monitor 监听等 | **按需启用**：每轮新任务前 Claude 问用户是否启用，启用才 Read `role-collaboration-rules.md` |
| §1 不明确必须提问 | 任何歧义都先问用户 | 永远生效 |
| §2 CLAUDE.md 工作流 | 项目识别、读取项目 CLAUDE.md、更新规范 | **每轮对话必读** `claude-md-workflow.md`（除非用户明确豁免本轮） |
| §3 语言规则 | 简体中文 | 永远生效 |
| §4 Maven 路径 | 本地仓库改到 `/Users/zoujunyong/repository` | 永远生效 |
| §5 数据库连接 | 优先 Python 驱动，不用 CLI | 永远生效 |
| §6 代码审查规则 | 任何代码改动后必须读 `code-review-checklist.md` 逐项执行 | **全局强制**：不论 Claude 还是 Codex 写的代码，都由 Claude 按清单审查 |

## 文件结构

| 文件 | 用途 | 加载时机 |
|------|------|---------|
| `CLAUDE.md` | 全局规则主索引 | 会话启动自动加载 |
| `role-collaboration-rules.md` | 「角色分工与协作规范」完整内容（Claude/Codex 分工、--watch、Monitor、审查清单等） | 每轮新任务前 Claude 问用户是否启用，启用才 Read |
| `claude-md-workflow.md` | 「CLAUDE.md 工作流」完整内容（项目识别、读取、更新流程） | 每轮对话必读 |
| `codex-commands.md` | Codex 命令速查 | 调用 Codex 前 Read |
| `code-review-checklist.md` | 代码审查清单 | 任何代码改动后 Read 并逐项执行 |
| `codex-run` | Codex 异步启动脚本（核心配套工具） | 调用 Codex 时执行 |
| `hooks/inject-project-claude-md.sh` | UserPromptSubmit hook 脚本，自动把当前 cwd 的项目 CLAUDE.md 注入到上下文。**当前未在 settings.json 注册**（启用时把 hook 节加回 `~/.claude/settings.json` 即可） | 注册后每轮自动触发 |

### 模块化的好处

- **主 CLAUDE.md 更短**：会话启动只加载主索引（约 80 行），不会一次性塞进所有规则
- **按需加载**：角色分工只在需要时启用，不强制每次都走重流程
- **独立维护**：修改某条规则只动一个文件，不会污染主 CLAUDE.md
- **冲突隔离**：每份独立文件职责单一，互不耦合

## codex-run 脚本

省 token 的另一半秘密：**把代码落地丢给 Codex，Claude 只做判断**。

`codex-run` 是配套的异步启动脚本，让 Claude 一行命令把任务扔给 Codex，自己继续做高密度输出。

### 用法

```bash
codex-run <project_dir> <task> [--resume] [--watch]
```

| 参数 | 说明 |
|------|------|
| `<project_dir>` | Codex 工作目录 |
| `<task>` | 任务描述（自然语言） |
| `--resume` | 接着上一轮 Codex 上下文继续干 |
| `--watch` | 自动弹出新终端实时附着 `tmux` 会话查看进度 |

### 工作机制

- 在 `tmux` 会话 `codex-work` 内异步执行 `codex exec`
- 日志写入 `/tmp/codex-output-<ctx-id>.log`
- 任务完成 → 创建标记文件 `/tmp/codex-done-<ctx-id>`
- Claude 通过轮询标记文件判断完成，无需阻塞对话

### 终端弹出策略

仅当 **同时满足** 以下三条才弹窗：

1. 显式传入 `--watch`
2. 当前不存在 `codex-work` 会话
3. 本次不是 `--resume`

否则一律静默后台运行，避免重复弹窗骚扰。

### 为什么这套设计省 token

- Claude 写代码 = 大量 output token（注释、解释、完整代码块）
- Codex 写代码 = 在它自己的进程里跑，**完全不消耗 Claude 的 output token**
- Claude 只负责：分析需求 → 拆任务 → 审查结果（全部高密度短输出）

**Claude 当指挥，Codex 当工人。** 指挥说话短，工人闷头干。账单自然薄。

## 使用方法

1. 克隆本仓库
2. `cp CLAUDE.md ~/.claude/CLAUDE.md`
3. 重启 Claude Code 会话
4. 体感：回复变短，信息密度变高，账单变薄

## 何时不用

- 教学解释（需要展开）
- 对外正式文档
- 与非技术用户沟通

技术对话场景闭眼用。

## License

MIT
