---
name: multica
description: Multica CLI 完整参考 — Agent 任务调度、Issue 管理、Daemon 运维
---

# Multica CLI 速查

Multica 是 Agent 任务调度平台。CLI 通过 `multica` 命令操作，配置文件 `~/.multica/config.json`。

## 前置

```bash
# 查看当前配置（workspace、server）
multica config show

# 切换 workspace
multica config set workspace_id <uuid>

# Daemon 状态
multica daemon status
```

**关键概念：**
- `issue list` 默认走云端 API，能看到所有 issue
- `issue get <id>` 需要完整 UUID（短 ID 会 404）
- `issue search <query>` 搜标题/描述，走云端
- `issue list --status all` 可能为空，直接 `multica issue list` 即可

## Agent 管理

```bash
# 列出所有 agent
multica agent list --output json

# 查看 agent 详情
multica agent get <id>

# 创建 agent
multica agent create \
  --name "Mr. X" \
  --runtime-id <runtime-id> \
  --description "描述" \
  --instructions "系统提示词" \
  --visibility private|workspace

# 更新 agent
multica agent update <id> \
  --name "新名字" \
  --instructions "新提示词" \
  --status idle|working \
  --runtime-id <runtime-id>

# 归档/恢复
multica agent archive <id>
multica agent restore <id>

# Agent 技能管理
multica agent skills list <agent-id>
multica agent skills set <agent-id> --skill-ids id1,id2,id3

# 查看 agent 任务
multica agent tasks <id>
```

## Issue 管理

```bash
# 列出 issue（走云端 API）
multica issue list
multica issue list --status todo --output json
multica issue list --assignee "Mr. 77-0"
multica issue list --project <project-id>

# 搜索 issue
multica issue search "bridge"
multica issue search "product" --include-closed --limit 50

# 获取详情（⚠️ 必须用完整 UUID）
multica issue get <full-uuid> --output json

# 创建 issue
multica issue create \
  --title "标题" \
  --description "描述" \
  --assignee "Mr. 77-0" \
  --priority none|low|medium|high|urgent \
  --status todo|in_progress|done|cancelled \
  --project <project-id> \
  --due-date 2026-04-20T00:00:00+08:00

# 更新 issue
multica issue update <id> \
  --title "新标题" \
  --assignee "Mr. 77-0" \
  --status in_progress \
  --priority high

# 改状态（快捷方式）
multica issue status <id> in_progress

# 指派/取消指派
multica issue assign <id> --to "Mr. 77-0"
multica issue assign <id> --unassign
```

## Issue 评论

```bash
# 添加评论（⚠️ 必须用完整 UUID）
multica issue comment add <full-uuid> --content "@Mr. 77-0 请继续执行"
multica issue comment add <id> --content-stdin < message.txt

# 回复评论
multica issue comment add <id> --content "回复" --parent <comment-id>

# 附件
multica issue comment add <id> --content "看图" --attachment /path/to/file.png

# 列出评论
multica issue comment list <full-uuid>
multica issue comment list <id> --limit 10 --since 2026-04-16T00:00:00Z

# 删除评论
multica issue comment delete <comment-id>
```

## Issue 执行记录

```bash
# 查看执行历史
multica issue runs <issue-id> --output json

# 查看某次执行的消息
multica issue run-messages <task-id> --output json
multica issue run-messages <task-id> --since 5  # 从第5条消息开始
```

## Project 管理

```bash
multica project list
multica project get <id>
multica project create --title "项目名" --description "描述" --icon 🚀
multica project update <id> --title "新名字" --lead "Mr. 77-0"
multica project status <id> active|archived
multica project delete <id>
```

## Skill 管理

```bash
# 列出/查看 skill
multica skill list
multica skill get <id>

# 创建 skill
multica skill create --name "skill-name" --description "描述" --content "内容"

# 更新 skill
multica skill update <id> --name "新名字" --content "新内容"

# Skill 文件管理
multica skill files list <skill-id>
multica skill files upsert <skill-id> --path "refs/api.md" --content "内容"
multica skill files delete <skill-id> <file-id>

# 导入 skill（从 clawhub.ai 或 skills.sh）
multica skill import --url https://clawhub.ai/skills/xxx

# 删除 skill
multica skill delete <id> --yes
```

## Runtime 管理

```bash
# 列出 runtime
multica runtime list

# Ping 测试连通性
multica runtime ping <runtime-id>
multica runtime ping <runtime-id> --wait

# Token 用量
multica runtime usage <runtime-id> --days 30

# 活动统计
multica runtime activity <runtime-id>

# 更新 runtime CLI 版本
multica runtime update <runtime-id> --target-version 0.2.1 --wait
```

## Workspace 管理

```bash
multica workspace list
multica workspace get [workspace-id]
multica workspace members [workspace-id]
```

## Daemon 运维

```bash
# 启动（默认后台）
multica daemon start
multica daemon start --foreground  # 前台运行（调试用）

# 高级启动参数
multica daemon start \
  --max-concurrent-tasks 4 \
  --poll-interval 10s \
  --agent-timeout 30m \
  --runtime-name "MacBook Pro"

# 状态
multica daemon status

# 日志
multica daemon logs
multica daemon logs -f          # 跟踪日志
multica daemon logs -n 200      # 最近200行

# 停止
multica daemon stop
```

## 附件与认证

```bash
# 下载附件
multica attachment download <attachment-id> -o /tmp/

# 认证状态
multica auth status
multica auth logout

# 登录
multica login
multica login --token
```

## 全局参数

| 参数 | 说明 |
|------|------|
| `--workspace-id <uuid>` | 覆盖默认 workspace |
| `--server-url <url>` | 覆盖默认 server URL |
| `--profile <name>` | 使用独立配置 profile |
| `--output json\|table` | 输出格式（大部分命令支持） |

## 常见问题

1. **`issue get` 返回 404** → 需要完整 UUID，短 ID 只在 `list` 的表格输出中可见
2. **`issue list` 为空但云端有数据** → 不要加 `--status all`，直接 `multica issue list`
3. **ClassLoader 错误** → `composer dump-autoload -o` 清理缓存
4. **Daemon 无法执行任务** → `multica daemon logs` 查看日志，确认 runtime 状态 `multica runtime ping <id>`
5. **`agent create` 没有 `--instructions-stdin`** → 只能用 `--instructions "内容"` 或 `--instructions "$(cat file)"`。别与 `issue comment add --content-stdin` 混淆

## 创建受限 Agent（远程 Claude Code）

在共享 runtime 上创建只做特定工作的 Agent 时，instructions 末尾必须加 Restrictions 段：

```markdown
## Restrictions

You are a Claude Code agent running on a remote server. Your ONLY job is [...].
You are strictly forbidden from:
- Reading, writing, or modifying any files on the local filesystem
- Running shell commands (ls, cat, rm, mkdir, etc.)
- Accessing databases, APIs, or services directly

Your only allowed actions are:
- **Talking** — conversation
- **Multica issue operations** — create, update, assign, comment
```

禁止什么、允许什么，必须逐一列出，否则 Agent 可能越权操作服务器。
