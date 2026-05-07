# Caveman CLAUDE.md

> Caveman 风格的 Claude Code 配置：同等约束力，token 砍掉一大半。

## 这是什么

一套**极致省 token** 的 `CLAUDE.md` 规则集与 Codex 协作流。

砍冗余，不砍规则。每次对话少烧钱，行为不打折。

## 为什么省

普通 CLAUDE.md：长句、敬语、解释、举例堆叠 → 每轮对话都把这堆塞进上下文 → token 烧穿。

Caveman 版：

- 删冠词、填充词（just / really / basically）
- 删客套（sure / of course / happy to）
- 删铺垫与总结段
- 保留：技术术语、规则原文、代码块、错误信息
- 表达模式：`[对象] [动作] [原因]. [下一步].`

实测同等约束力下输入 token 降低约 **60%~75%**。

## 文件结构

| 文件 | 用途 |
|------|------|
| `~/.claude/CLAUDE.md` | 全局规则（所有项目生效） |
| `<项目>/CLAUDE.md` | 项目级规则（业务逻辑、接口约定、踩坑） |
| `~/.claude/codex-commands.md` | Codex 常用命令速查 |

## 核心规则摘要

### 角色分工

- **Claude**：需求分析、结构设计、任务拆分、代码审查、最终验收
- **Codex**：代码 / 脚本 / 配置 / 文档全部落地

Claude 不写代码。脚本配置文档同样算代码。仅用户明确指定时例外。

### 协作铁律

- 调 Codex 前必问：`需要 --watch 实时查看？`
- 调 Codex 后必监听完成文件 `/tmp/codex-done-<ctx-id>`
- Codex 提问 → Claude 给建议 → 用户确认 → `--resume` 继续
- 审查必读 `~/.claude/code-review-checklist.md` 逐项执行

### 文档约定

- `CLAUDE.md` 技术架构部分由 `/init` 生成，Codex 不动
- 业务逻辑部分由 Codex 在 `## 业务逻辑文档` 章节增量维护
- 一项目一文档，禁止跨项目混写

### 不明确必问

任何歧义 → 提问，不猜测、不假设。一次问完，问到清楚为止。

## 使用方法

1. 克隆本仓库
2. 把 `CLAUDE.md` 拷进 `~/.claude/`
3. 项目根目录按需放置项目级 `CLAUDE.md`
4. 配套 Caveman 模式：`/caveman full` 或在系统提示里启用

## 适用场景

- 长会话、上下文吃紧
- 多项目并行、规则集大
- API 计费敏感
- 想让 Claude 少废话直接干活

## 不适用

- 教学场景（需要详细解释）
- 团队新人 onboarding 文档
- 需要正式语气的对外材料

## License

MIT
