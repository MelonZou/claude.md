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

1. `~/.claude/CLAUDE.md` 全局规则 —— 每次会话自动加载
2. 规则强制约束输出风格：删冠词、删客套、删铺垫、删总结、保留技术术语与代码块
3. 触发词 / `/caveman` 斜杠命令切换强度（lite / full / ultra）
4. 配套 Codex 协作流，把代码落地这种「必须详细」的工作交给另一个 Agent，Claude 只做高密度判断输出

## 文件结构

| 文件 | 用途 |
|------|------|
| `CLAUDE.md` | 全局规则主体，放进 `~/.claude/` |
| `codex-commands.md` | Codex 命令速查 |
| `code-review-checklist.md` | 审查清单 |
| `codex-run` | Codex 异步启动脚本（核心配套工具） |

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
