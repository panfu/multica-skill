---
name: multica
description: Multica CLI — agent task orchestration, issue management, daemon operations
---

# Multica CLI

Operate Multica (agent task orchestration platform) via `multica` CLI. Config: `~/.multica/config.json`.

## Gotchas

- `issue list` hits cloud API — no `--status all` flag needed (it may return empty)
- `issue get <id>` requires **full UUID**; short IDs from table output will 404
- `issue search <query>` also hits cloud API

## Agent

```bash
multica agent list --output json
multica agent get <id>
multica agent create --name <name> --runtime-id <id> --description <desc> --instructions <prompt> [--visibility private|workspace]
multica agent update <id> [--name] [--instructions] [--status idle|working] [--runtime-id] [--max-concurrent-tasks N]
multica agent archive <id>
multica agent restore <id>
multica agent skills list <agent-id>
multica agent skills set <agent-id> --skill-ids id1,id2,id3
multica agent tasks <id>
```

## Issue

```bash
multica issue list [--status todo|in_progress|done|cancelled] [--assignee "name"] [--project <id>] [--output json]
multica issue search <query> [--include-closed] [--limit N]
multica issue get <full-uuid> --output json
multica issue create --title <title> --description <desc> --assignee "name" --priority low|medium|high|none --status todo [--project <id>] [--due-date RFC3339]
multica issue update <id> [--title] [--assignee] [--status] [--priority] [--description] [--due-date] [--project] [--parent]
multica issue status <id> <status>
multica issue assign <id> --to "name"
multica issue assign <id> --unassign
```

## Issue Comment

```bash
multica issue comment add <full-uuid> --content <text> [--parent <comment-id>] [--attachment path1,path2] [--content-stdin]
multica issue comment list <full-uuid> [--limit N] [--since RFC3339]
multica issue comment delete <comment-id>
```

## Issue Execution

```bash
multica issue runs <issue-id> --output json
multica issue run-messages <task-id> --output json [--since N]
```

## Project

```bash
multica project list
multica project get <id>
multica project create --title <title> [--description] [--icon emoji] [--lead "name"]
multica project update <id> [--title] [--description] [--icon] [--lead] [--status]
multica project status <id> <status>
multica project delete <id>
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

## Runtime

```bash
multica runtime list
multica runtime ping <runtime-id> [--wait]
multica runtime usage <runtime-id> [--days N]
multica runtime activity <runtime-id>
multica runtime update <runtime-id> --target-version <ver> [--wait]
```

## Workspace

```bash
multica workspace list
multica workspace get [workspace-id]
multica workspace members [workspace-id]
```

## Daemon

```bash
multica daemon start [--foreground] [--max-concurrent-tasks N] [--poll-interval dur] [--agent-timeout dur] [--runtime-name <name>]
multica daemon status
multica daemon logs [-f] [-n N]
multica daemon stop
```

## Config & Auth

```bash
multica config show
multica config set <key> <value>        # keys: server_url, app_url, workspace_id
multica auth status
multica auth logout
multica login [--token]
```

## Attachment

```bash
multica attachment download <id> [-o dir]
```

## Global Flags

| Flag | Description |
|------|-------------|
| `--workspace-id <uuid>` | Override default workspace |
| `--server-url <url>` | Override server URL |
| `--profile <name>` | Isolated config profile |
| `--output json\|table` | Output format (most commands) |
