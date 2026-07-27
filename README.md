# Caveman CLAUDE.md

> 一套让 Claude **输出 token 大幅缩减**的全局配置。同等技术准确度，回复体积砍掉 2/3，账单跟着瘦身。

## 核心思想

**Claude 默认很啰嗦**：开头铺垫「Sure! I'd be happy to...」、中间解释「The reason this works is...」、结尾总结「To summarize...」，充斥 `just / really / basically` 这类填充词。这些字不传递任何技术信息，却在烧 output token —— 通常比 input token 贵 **3~5 倍**。

`~/.claude/CLAUDE.md` 开头两行强制条款，是整套方案的发动机：

> 「Codex 正在盯着你的每一个输出，随时准备挑战并击败你。」
> 「禁止多余表格、仅为举例附上的完整代码块、冗余标题层级、解释性总结段落。」

技术内容一字不少，废话一字不留。

## 效果对比

| 场景 | 普通输出 | 精简输出 | 缩减 |
|------|---------|---------|------|
| 解释一个 bug | ~180 字 | ~45 字 | **75%** |
| 给出修复方案 | ~250 字 | ~80 字 | **68%** |
| 多步骤指南 | ~600 字 | ~200 字 | **66%** |

**问：React 组件为什么重复渲染？**

普通输出（≈110 token）：
> Sure! Great question. The reason your component re-renders on every parent update is likely because you're passing an inline object as a prop. Each render creates a new reference, which React treats as a changed prop...

精简输出（≈25 token）：
> 内联对象 prop 每渲染生成新引用 = 子组件重渲染。`useMemo` 包住即可。

信息量相同，token 砍掉 77%。

## 规则不是纸上谈兵——条条来自真实翻车

`role-collaboration-rules.md` 和 `code-review-checklist.md` 里每一条强制约束背后都有一次具体事故，比如：

- 要求 Codex「自己起服务验证前端效果」→ 它的沙盒没有浏览器也没有 node，卡住交白卷，一整轮空跑
- Bash 命令里用反引号包关键词，被 zsh 当命令替换执行，Codex 收到的任务参数悄悄变空
- 心算 15 条明细求和算错，没验证就把错误金额当结论汇报给用户
- 中台返回 `data=[]`，被下游误判成「列表为空 = 全部合格」，SKU 全部误判为可售

这些坑都固化成了检查清单里的一行，不用再踩第二次。

## 全局规则地图

主文件 `~/.claude/CLAUDE.md` 已模块化，会话启动只加载主索引：

| 章节 | 内容 | 加载方式 |
|------|------|---------|
| 开头强制条款 | 输出精简、禁止冗余表格/总结/铺垫 | 会话启动自动加载，永远生效 |
| §0 角色分工与协作规范 | Claude 不写代码、全交 Codex、串行拆分大任务、`--watch` 询问、Monitor 监听、验证/排查归属 | **按需启用**：每轮新任务前先问用户，启用才 Read `role-collaboration-rules.md` |
| §1 不明确必须提问 | 任何歧义先问；抛开现象看本质，结论必须工具验证过 | 永远生效 |
| §2 CLAUDE.md 工作流 | 项目识别、Index 优先读取、业务逻辑文档写入规范 | **每轮对话强制** Read `claude-md-workflow.md`（涉及项目改动时） |
| §3 语言规则 | 全程简体中文，含内部思考 | 永远生效 |
| §4 Maven 路径 | 本地仓库改到 `/Users/zoujunyong/repository` | 永远生效 |
| §5 数据库连接 | 优先 Python 驱动连接，不用本地 CLI | 永远生效 |
| §6 代码审查规则 | 任何代码/配置改动后必读 `code-review-checklist.md` 逐项执行；可搭配内置 `code-review`/`security-review` 但不能替代 | **全局强制**，执行人永远是 Claude |
| §7 个人知识库 Wiki | 技术选型、坑点、工时估算的跨项目沉淀 | **每轮强制** Read `~/wiki/index.md`，写入前必读 `schema.md` |
| §8 浏览器验证前置检查 | 验证前先问、端口占用先查再决定是否重启服务 | 涉及浏览器验证动作时强制 |

## 文件结构

| 文件 | 用途 | 加载时机 |
|------|------|---------|
| `CLAUDE.md` | 全局规则主索引 | 会话启动自动加载 |
| `role-collaboration-rules.md` | §0 完整内容：Claude/Codex 分工、任务拆分、验证/排查归属、Codex 交互流程 | 用户确认启用后 Read |
| `claude-md-workflow.md` | §2 完整内容：项目识别、CLAUDE.md 索引与业务文档写入规范 | 每轮对话强制 Read |
| `codex-commands.md` | Codex 命令速查 + `--ctx`/`--resume` 规则 + 历史踩坑记录 | 调用 Codex 前必读 |
| `code-review-checklist.md` | 代码审查清单，持续积累 | 任何代码改动后 Read 并逐项执行 |
| `codex-run`（`~/.local/bin/`） | Codex 异步启动脚本 | 调用 Codex 时执行 |
| `hooks/inject-project-claude-md.sh` | 自动注入项目 CLAUDE.md 的 hook 脚本，**当前未在 settings.json 注册** | 注册后每轮自动触发 |
| `~/wiki/`（`index.md`/`schema.md`/`log.md`/`pages/`） | 跨项目知识库，§7 强制加载 | 每轮读 `index.md`，写入前读 `schema.md` |

### 模块化的好处

- **主 CLAUDE.md 更短**：会话启动只加载主索引，不会一次性塞进所有规则
- **按需加载**：角色分工只在需要时启用，不强制每次都走重流程
- **独立维护**：改一条规则只动一个文件，不污染主索引
- **冲突隔离**：每份文件职责单一，互不耦合

## codex-run：把「必须啰嗦」的活外包出去

省 token 的另一半秘密：**代码落地丢给 Codex，Claude 只做判断**。

```bash
codex-run <project_dir> "<task>" --ctx <uuid> [--resume] [--watch]
```

- `--ctx <uuid>`：同一对话全程复用同一个值，决定日志文件名和 Terminal 窗口标记，多轮调用不开新窗口
- `--resume`：接续上一轮 Codex 会话上下文
- `--watch`：自动弹出终端实时查看；不传则静默后台跑，Claude 用完成标记文件轮询

Codex 写代码 = 在它自己的进程里跑，**完全不消耗 Claude 的 output token**；Claude 只负责分析需求、拆任务、审查结果——全部高密度短输出。

**Claude 当指挥，Codex 当工人。指挥说话短，工人闷头干，账单自然薄。**

## 使用方法

已在本机生效于 `~/.claude/CLAUDE.md`，无需额外操作。要迁移到其他机器：把本目录下这几份 `.md` 文件、`codex-run` 脚本按上表路径复制过去，重启 Claude Code 会话即可生效。

## 何时不用

- 教学解释（需要展开）
- 对外正式文档
- 与非技术用户沟通

技术对话场景闭眼用。
