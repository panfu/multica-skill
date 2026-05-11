---
name: multica
description: Multica CLI — agent task orchestration, issue management, daemon operations
---

# Multica CLI

Operate Multica (agent task orchestration platform) via `multica` CLI. Config: `~/.multica/config.json`.

## Self-Hosted Instance

- **URL:** https://desk.artextilestore.com.cn
- **Workspace:** 85539409
- CLI 默认连此实例（`~/.multica/config.json` 中的 `server_url`）

> 开发相关（仓库路径、架构、Gateway）见 `multica-dev` skill。

## Gotchas

- `issue list` hits cloud API — no `--status all` flag needed (it may return empty)
- `issue get <id>` accepts **issue key** (e.g. `ART-47`) or **full UUID**; list 命令默认输出短 key，加 `--full-id` 输出完整 UUID
- `issue search <query>` also hits cloud API
- **multica.ai 云实例 vs 自部署实例是两套独立系统**：不同的 token、不同的 workspace_id。CLI 默认连 `server_url`（自部署），查云实例需直接 curl `api.multica.ai`。
- **multica.ai 的 API 端点是 `api.multica.ai`，不是 `multica.ai`**（后者是前端 CDN，返回 404 页面）。URL 路径 `/artextile/issues/UUID` 中的 `artextile` 是 `workspace_slug`，不是 workspace_id。
- **跨实例查 issue 的 curl 写法**：
  ```bash
  # 云实例（multica.ai）需要用 workspace_slug，不能用 desk 的 workspace_id
  curl -s -H "Authorization: Bearer $TOKEN" \
    "https://api.multica.ai/api/issues/<uuid>?workspace_slug=artextile"
  # 自部署实例（desk）用 multica CLI 或 curl desk URL
  ```
- **Long content via `--content` flag may timeout** when passed directly as shell argument (special chars like `&`, `<`, `>`, backticks, quotes break escaping). Fix: write to a temp file, then use shell process substitution `<()`:
  ```bash
  # Write content to temp file
  content_file="/tmp/skill-content-$$.md"
  cat > "$content_file" << 'SKILLEOF'
  ---
  name: my-skill
  ...
  SKILLEOF
  
  # Create/update skill using process substitution (no base64 needed)
  multica skill create --name "my-skill" --description "..." --content "$(cat "$content_file")"
  
  # Or for updates
  multica skill update <id> --content "$(cat "$content_file")"
  
  rm "$content_file"
  ```
  
  Alternative (base64 pipe, works when process substitution fails):
  ```python
  import base64, subprocess
  encoded = base64.b64encode(content.encode()).decode()
  subprocess.run(f"echo '{encoded}' | base64 -d | xargs -0 multica skill update {id} --content", shell=True)
  ```
  Works for `multica agent update --instructions` and `multica skill update --content` alike.
- `skill update` does NOT support `--content-stdin` (only `issue comment add` does)
- **Skill content delivery**: The `--content` flag accepts the full skill body. For reliable delivery without shell escaping issues:
  1. **Process substitution** (preferred): `multica skill create --name "..." --content "$(cat /tmp/skill.md)"`
  2. **Base64 pipe** (fallback): `echo '<base64>' | base64 -d | xargs -0 multica skill create --content`
  3. **Direct inline** (only for short content < 1KB with no special chars)
- **Daemon can't find agent CLIs** — daemon starts with a minimal PATH (doesn't source .zshrc/.bashrc). If agents are in `/opt/homebrew/bin` or `~/.local/bin`, daemon won't find them. Fix: start daemon with explicit PATH:
  ```bash
  PATH=/opt/homebrew/bin:/usr/local/bin:$HOME/bin:$HOME/.local/bin:$HOME/.opencode/bin:$PATH multica daemon start
  ```
- **NAS SCP fails** — scp to NAS (port 7110) silently fails with "No such file or directory". Use pipe instead:
  ```bash
  cat local_file | ssh nas "cat > /home/panfu/remote_file"
  ```
- **sudo over non-interactive SSH** — `ssh host "sudo cmd"` fails without TTY. Fix:
  ```bash
  ssh host 'echo "password" | sudo -S sh -c "command"'
  ```
- **Label commands may return 404 on self-hosted** — `label`, `issue label`, `issue subscriber` commands exist in CLI v0.2.29+ but require backend support. If you get `404 page not found`, the instance doesn't have these features enabled.
- **Agent avatar requires `--file` flag** — `multica agent avatar <id> --file <path>` uploads an image. No other flags supported.
- **Daemon `disk-usage` is new in v0.2.29** — shows per-workspace storage. Use `--by-workspace` for breakdown, `--top N` for largest items.

## Setup

```bash
multica setup cloud                    # 配置 cloud 实例
multica setup self-host                # 配置自部署实例
```

## Login

```bash
multica login [--token]                # 交互式登录
multica login --token -                # 从 stdin 读取 token（非交互式）
multica auth status
multica auth logout
```

## Config

```bash
multica config show
multica config set <key> <value>       # keys: server_url, app_url, workspace_id
```

## Update

```bash
multica update                         # 升级 CLI 到最新版
multica update --download-timeout 300  # 自定义下载超时（秒）
multica version                        # 查看当前版本
```

## Workspace

```bash
multica workspace list
multica workspace get [workspace-id]
multica workspace members [workspace-id]
multica workspace update <id> [--name] [--description] [--context] [--issue-prefix]  # 长字段支持 --description-stdin / --context-stdin
```

## Agent

```bash
multica agent list --output json
multica agent get <id>
multica agent create --name <name> --runtime-id <id> --description <desc> --instructions <prompt> [--visibility private|workspace] [--model <model>] [--custom-args <args>] [--custom-env <env>]
multica agent update <id> [--name] [--instructions] [--status idle|working] [--runtime-id] [--max-concurrent-tasks N] [--model] [--custom-args] [--custom-env]
multica agent archive <id>
multica agent restore <id>
multica agent skills list <agent-id>
multica agent skills set <agent-id> --skill-ids id1,id2,id3
multica agent tasks <id>               # 需要 full UUID，不支持 slug
multica agent avatar <id> --file <path>
```

## Issue

```bash
multica issue list [--status todo|in_progress|done|cancelled] [--assignee "name"] [--project <id>] [--output json] [--full-id]
multica issue search <query> [--include-closed] [--limit N]
multica issue get <id> --output json   # 支持 issue key（ART-47）或 full UUID
multica issue create --title <title> --description <desc> --assignee "name" --priority none|low|medium|high|urgent --status todo [--project <id>] [--due-date RFC3339] [--attachment path1,path2]
multica issue update <id> [--title] [--assignee] [--status] [--priority] [--description] [--due-date] [--project] [--parent]
multica issue status <id> --set <status>
multica issue assign <id> --agent <agent-slug>   # 立即触发 task
multica issue assign <id> --unassign
multica issue rerun <id>               # 重新执行 issue
```

## Issue Comment

```bash
multica issue comment add <id> --content <text> [--parent <comment-id>] [--attachment path1,path2] [--content-stdin]
multica issue comment list <id> [--limit N] [--since RFC3339]
multica issue comment delete <comment-id>
```

## Issue Label

```bash
multica issue label list <issue-id>
multica issue label add <issue-id> --label <label-name>
multica issue label remove <issue-id> --label <label-name>
```

> ⚠️ 需要后端支持。self-host 实例可能返回 404。

## Issue Subscriber

```bash
multica issue subscriber list <issue-id>
multica issue subscriber add <issue-id> --user <user-id>
multica issue subscriber remove <issue-id> --user <user-id>
```

> ⚠️ 需要后端支持。self-host 实例可能返回 404。

## Issue Execution

```bash
multica issue runs <issue-id> --output json
multica issue run-messages <task-id> --output json [--since N]
```

## Label

```bash
multica label list
multica label get <id>
multica label create --name <name> [--description] [--color <hex>]
multica label update <id> [--name] [--description] [--color]
multica label delete <id>
```

> ⚠️ 需要后端支持。self-host 实例可能返回 404。

## Project

```bash
multica project list
multica project get <id>
multica project create --title <title> [--description] [--icon emoji] [--lead "name"] [--repo <github-url>]
multica project update <id> [--title] [--description] [--icon] [--lead] [--status]
multica project status <id> <status>
multica project delete <id>
```

## Project Resource

```bash
multica project resource list <project-id>
multica project resource add <project-id> --type <type> --url <url>
multica project resource remove <project-id> <resource-id>
```

## Skill

```bash
multica skill list
multica skill get <id>
multica skill create --name <name> --description <desc> --content <body>
multica skill update <id> [--name] [--description] [--content]
multica skill files list <skill-id>
multica skill files upsert <skill-id> --path <path> --content <body>
multica skill files delete <skill-id> <file-id>
multica skill import --url <clawhub|skills.sh url>
multica skill delete <id> --yes
```

## Autopilot

```bash
multica autopilot list
multica autopilot get <id>
multica autopilot create --title <title> --agent <name-or-id> --mode create_issue --description <prompt> [--priority none|low|medium|high|urgent] [--project <id>] [--issue-title-template <tpl>]
multica autopilot update <id> [--title] [--description] [--agent] [--priority] [--mode]
multica autopilot delete <id>
multica autopilot trigger <id>                    # 手动触发一次
multica autopilot runs <id>                       # 查看执行历史

# 定时触发器
multica autopilot trigger-add <id> --cron <expr> [--timezone Asia/Shanghai] [--label <name>]
multica autopilot trigger-update <trigger-id> [--cron] [--timezone] [--label]
multica autopilot trigger-delete <trigger-id>
```

**Note:** `--mode` currently only supports `create_issue`. `--timezone` defaults to UTC — always set explicitly for non-UTC schedules.

## Runtime

```bash
multica runtime list
multica runtime ping <runtime-id> [--wait]
multica runtime usage <runtime-id> [--days N]
multica runtime activity <runtime-id>
multica runtime update <runtime-id> --target-version <ver> [--wait]
```

## Daemon

```bash
multica daemon start [--foreground] [--max-concurrent-tasks N] [--poll-interval dur] [--agent-timeout dur] [--runtime-name <name>]
multica daemon status
multica daemon logs [-f] [-n N]
multica daemon stop
multica daemon restart
multica daemon disk-usage [--by-workspace] [--top N]
```

Daemon requires at least one agent CLI (claude/codex/opencode/openclaw/hermes/gemini) on PATH. Without any, daemon refuses to start with `Error: no agent CLI found`.

### Daemon Migration (switch server instance)

When migrating runtimes to a new Multica instance, every server needs:

1. `multica config set server_url <new-url>`
2. `multica config set workspace_id <new-workspace-id>`
3. Re-authenticate with new instance token: tokens are per-instance, not portable
4. `multica daemon stop && multica daemon start`

Runtimes auto-register on daemon start. No manual runtime creation needed.

### Cross-Instance Data Migration

Migrate agents, issues, projects from one Multica instance to another (e.g. api.multica.ai → desk.artextilestore.com.cn).

**Prerequisites:** source API token (Bearer), target instance already configured in `~/.multica/config.json`.

**Workflow (Python/execute_code):**

```python
import json, subprocess, base64, os

SRC_TOKEN = "mul_xxx"
SRC_API = "https://api.multica.ai"
SRC_WS = "workspace-uuid"

# 1. Fetch agents from source
r = subprocess.run(["curl", "-s", "-H", f"Authorization: Bearer {SRC_TOKEN}",
     f"{SRC_API}/api/agents?workspace_id={SRC_WS}"], capture_output=True, text=True)
agents = json.loads(r.stdout)  # Returns array directly

# 2. Build runtime ID mapping (old runtime → new runtime)
#    Use `multica runtime list` on target to get new runtime IDs
#    Map by machine/provider (hostname determines runtime name)

# 3. Create agents on target
for a in agents:
    encoded = base64.b64encode(a['instructions'].encode()).decode()
    cmd = f"echo '{encoded}' | base64 -d | xargs -0 multica agent create " \
          f"--name '{a['name']}' --runtime-id <NEW_RT_ID> " \
          f"--description '{a.get('description','')}' " \
          f"--max-concurrent-tasks {a.get('max_concurrent_tasks', 6)}"
    subprocess.run(cmd, shell=True, capture_output=True, text=True)

# 4. Fetch issues from source
r = subprocess.run(["curl", "-s", "-H", f"Authorization: Bearer {SRC_TOKEN}",
     f"{SRC_API}/api/issues?workspace_id={SRC_WS}&limit=100"], capture_output=True, text=True)
data = json.loads(r.stdout)  # Returns {"data": [...]} or direct array
issues = data if isinstance(data, list) else data.get('data', data.get('issues', []))

# 5. Create issues on target (without assignee)
for issue in issues:
    encoded = base64.b64encode((issue.get('description','') or '').encode()).decode()
    cmd = f"echo '{encoded}' | base64 -d | xargs -0 multica issue create " \
          f"--title '{issue['title']}' --status {issue.get('status','todo')}"
    subprocess.run(cmd, shell=True, capture_output=True, text=True)

# 6. Fetch projects from source
r = subprocess.run(["curl", "-s", "-H", f"Authorization: Bearer {SRC_TOKEN}",
     f"{SRC_API}/api/projects?workspace_id={SRC_WS}"], capture_output=True, text=True)
data = json.loads(r.stdout)  # Returns {"projects": [...], "total": N}

# 7. Create projects via target API (CLI project create may not persist)
with open(os.path.expanduser('~/.multica/config.json')) as f:
    tgt = json.load(f)
for p in data['projects']:
    subprocess.run(["curl", "-s", "-X", "POST",
        "-H", f"Authorization: Bearer {tgt['token']}",
        "-H", "Content-Type: application/json",
        "-d", json.dumps({"title": p['title'], "description": p.get('description'),
                          "icon": p.get('icon'), "status": p.get('status', 'planned')}),
        f"{tgt['server_url']}/api/projects?workspace_id={tgt['workspace_id']}"],
        capture_output=True, text=True)
```

**Pitfalls:**
- Source API response formats vary: agents→array, issues→`{"data":[...]}`, projects→`{"projects":[],"total":N}`
- `multica project create` via CLI may silently fail to persist — use direct API POST instead
- No standalone repo API on either instance. GitHub repo info lives in project description text only. Real repo linking requires Web UI GitHub App integration
- `base64 -d | xargs -0` trick is essential for long instructions/descriptions (avoids shell escaping hell)

### Hostname → Runtime Name

Runtime display name derives from machine hostname. To rename:
```bash
sudo hostname S101            # immediate
echo S101 | sudo tee /etc/hostname  # persistent
multica daemon stop && multica daemon start  # re-register with new name
```
No server reboot needed — just restart daemon.

### Manual CLI Upgrade (when in-app upgrade fails)

In-app upgrade fails with `permission denied` when binary is in `/usr/local/bin/`. Fix: stop daemon, replace binary manually, restart. Download the matching `multica-cli-<version>-<os>-<arch>.tar.gz` from GitHub releases, extract, copy binary, restart daemon.

**Apple Silicon Homebrew path:** binary lives at `/opt/homebrew/bin/multica` which symlinks to `/opt/homebrew/Cellar/multica/<version>/bin/multica`. The Cellar binary is read-only (`-r-xr-xr-x`, owner `astra:admin`). Steps:

```bash
multica daemon stop
cd /tmp && curl -L -o multica-cli.tar.gz \
  'https://github.com/multica-ai/multica/releases/download/v<VERSION>/multica-cli-<VERSION>-darwin-arm64.tar.gz'
tar xzf multica-cli.tar.gz
chmod u+w /opt/homebrew/Cellar/multica/*/bin/multica   # owner is current user
cp /tmp/multica /opt/homebrew/Cellar/multica/*/bin/multica
chmod 755 /opt/homebrew/Cellar/multica/*/bin/multica
multica version  # verify
multica daemon start
```

**Pitfalls:**
- `sudo` requires TTY — won't work in non-interactive agent terminals. Use `chmod u+w` instead (file is owned by current user).
- Don't skip `multica daemon stop` before replacing — running daemon holds the binary open on Linux (macOS allows replacement but daemon will report stale version).
- After `brew upgrade multica` later, the binary gets overwritten back — that's fine, Homebrew will have the newer version by then.

### CLI Install from GitHub

Some networks block `get.multica.ai`. Install from GitHub releases instead: download `multica-cli-<version>-<os>-<arch>.tar.gz`, extract, place binary in `~/bin/`, ensure `~/bin` is in PATH.

## Attachment

```bash
multica attachment download <id> [-o dir]
```

## Repo

```bash
multica repo checkout <url> [--ref <branch>]  # 克隆项目仓库到本地 workspace
```

## Config & Auth

```bash
multica config show
multica config set <key> <value>        # keys: server_url, app_url, workspace_id
multica auth status
multica auth logout
multica login [--token]
```

## Global Flags

| Flag | Description |
|------|-------------|
| `--workspace-id <uuid>` | Override default workspace |
| `--server-url <url>` | Override server URL |
| `--profile <name>` | Isolated config profile |
| `--full-id` | List commands: print full UUID instead of short key |
| `--output json\|table` | Output format (most commands) |

---
