### Codex 常用命令（持续积累，避免重复查 help）

```bash
# --watch 时 Codex 进程在后台运行，Terminal.app 窗口只用于显示日志（tail -f），关闭窗口不会中断 Codex；静默模式完全在后台运行

# 推荐方式：通过 codex-run 脚本调用（--watch 时自动弹出系统原生 Terminal.app 窗口实时查看）
# --ctx <uuid> 由 Claude 在对话开始时生成 UUID 并全程传递，用户无需关心；作用：① 决定日志文件名（/tmp/codex-output-<uuid>.log）② 标记对应的 Terminal 窗口 tab（标题 Codex-<uuid>），同一 ctx 的多次 --watch 调用复用同一窗口
codex-run <项目目录> "任务描述" --ctx <uuid>              # 新建会话，静默
codex-run <项目目录> "任务描述" --ctx <uuid> --watch      # 新建会话，自动弹出终端实时查看
codex-run <项目目录> "任务描述" --ctx <uuid> --resume     # 复用会话，静默
codex-run <项目目录> "任务描述" --ctx <uuid> --resume --watch  # 复用 Codex 会话，有已有 Codex-<uuid> 标记窗口时在该窗口新建 tab，否则新开窗口

# 底层命令（codex-run 内部使用，一般不直接调用）
# 当前对话第一次调用（新建会话）
codex exec --full-auto -C /path/to/project "任务描述" < /dev/null

# 当前对话后续再调用（接续上次会话，保留项目上下文，更快）
codex exec resume --last --full-auto -C /path/to/project "任务描述" < /dev/null

# 指定模型
codex exec --full-auto --model gpt-5.4 -C /path/to/project "任务描述" < /dev/null

# 实时输出 JSONL 事件流
codex exec --full-auto --json -C /path/to/project "任务描述" < /dev/null

# 临时强制赋予读写权限（信任配置未加时的应急方案，优先用信任配置）
codex exec --full-auto -c 'sandbox_permissions=["disk-full-read-access","disk-write-access"]' -C /path/to/project "任务描述" < /dev/null
```

**关键注意：**
- `< /dev/null` 必须加，否则 Codex 会在非交互环境下等待 stdin 阻塞
- `-C /path/to/project` 指定工作目录，让 Codex 在正确目录下读写文件
- `--full-auto` 等价于 `-a on-request --sandbox workspace-write`，免去每步确认
- **完成信号文件为 `/tmp/codex-done-<ctx-id>`（有 `--ctx` 时）或 `/tmp/codex-done`（无 `--ctx` 时）。** Monitor 监听路径必须严格匹配这条规则，不能一律写成 `/tmp/codex-done`；`--ctx` 同时影响日志文件名（`/tmp/codex-output-<ctx-id>.log`）和 Terminal 窗口标记（tab 标题 `Codex-<uuid>`）

---

### --ctx 与 --resume 强制规则（无例外）

**在同一对话内多次调用 Codex 时：**

1. **--ctx 必须全程沿用同一个值**：对话开始时生成一个 UUID，之后每次调用都传同一个 --ctx，不得每次生成新值。同一 --ctx 的 --watch 调用会复用同一 Terminal 窗口，生成新 --ctx 会开新窗口。

2. **第二次及之后的调用必须加 --resume**：复用上次 Codex 会话上下文，更快且不丢失项目理解。

3. **判断是否首次调用**：当前对话上下文中有无 codex exec 调用记录——有则 resume，无则新建。

❌ 严禁每次调用都生成新的时间戳作为 --ctx
❌ 严禁后续调用忘记加 --resume

### Codex 会话复用规则（强制）

| 场景 | 使用命令 |
|------|---------|
| 当前对话第一次调用 Codex | `codex exec`（新建） |
| 当前对话后续再次调用 Codex | `codex exec resume --last`（接续） |
| 新对话 / 上下文已被清空压缩 | `codex exec`（新建） |

**判断依据**：当前对话上下文中是否已有 `codex exec` 的调用记录。有则接续，无则新建。

> 新发现的有用命令，随时追加到这里，不要重复查 `codex --help`。

### Codex 中转文件清理规则（强制）

当 Codex 因沙盒限制无法直接写入目标路径（如 `~/.local/bin/`），需要先写到信任目录中转时：

1. 我用 Bash 工具把文件复制到目标路径
2. **复制完成后立即删除信任目录里的中转文件**，不得遗留
3. 确认目标路径文件存在后，任务才算完成

---

### Codex 信任配置（必须检查，调用前强制执行）

**每次分配任务给 Codex 之前**，必须先检查 `~/.codex/config.toml`，确认当前工作目录已在信任列表中：

```toml
[projects."/path/to/当前项目"]
trust_level = "trusted"
```

- 若已存在 → 直接继续
- 若不存在 → **立即添加，再调用 Codex**，否则 Codex 会以沙盒只读模式运行，无法写入任何文件

> 信任配置文件路径：`~/.codex/config.toml`  
> 当前已信任目录：`/Users/zoujunyong/Desktop/浦东ai值班`、`/Users/zoujunyong/Desktop/虹桥ai值班`

