# multica-skill

Hermes skill for [Multica](https://multica.ai) CLI — agent task orchestration, issue management, and daemon operations.

## What is Multica?

Multica is an agent task orchestration platform. You create AI agents, assign them tasks (issues), and let them run autonomously. The `multica` CLI is the primary interface for managing agents, issues, projects, skills, and runtimes.

## What this skill covers

This skill provides Hermes with comprehensive knowledge of the `multica` CLI, including:

- **Agent management** — create, update, archive, assign skills
- **Issue lifecycle** — create, search, assign, comment, label, subscribe, rerun
- **Project & resource management** — projects, GitHub repo linking
- **Skill management** — create, update, import from clawhub/skills.sh
- **Autopilot** — scheduled agent triggers with cron
- **Runtime & daemon ops** — start/stop daemon, disk usage, migration
- **Cross-instance migration** — migrate data between cloud and self-hosted instances
- **Gotchas & pitfalls** — real-world lessons from production use

## Target CLI version

Tested against **multica CLI v0.2.29** (May 2026).

## Self-hosted instance reference

This skill was developed against a self-hosted Multica instance:

- **URL:** https://desk.artextilestore.com.cn
- **Workspace:** 85539409

> The gotchas section documents self-host specific behaviors (e.g., label commands returning 404 when backend features are disabled).

## Structure

```
SKILL.md          # Full CLI reference, gotchas, and workflows
README.md         # This file
```

## Key sections in SKILL.md

| Section | What's inside |
|---------|--------------|
| Gotchas | Shell escaping, UUID vs issue key, PATH issues, daemon pitfalls |
| Agent | Full CRUD + skills assignment + avatar + custom model args |
| Issue | Lifecycle + comments + labels + subscribers + rerun |
| Autopilot | Scheduled triggers, cron syntax, timezone handling |
| Daemon | Start/stop/restart, disk-usage, migration procedures |
| Cross-Instance Migration | Python scripts to migrate agents/issues/projects between instances |

## Contributing

This skill is maintained by [panfu](https://github.com/panfu). Updates are tested against a live self-hosted instance before committing.

To update after a new multica CLI release:

1. Run `multica version` to confirm target version
2. Test every command section in SKILL.md
3. Update flags, add new commands, revise gotchas
4. Commit with version number in message

## Related

- [multica-ai/multica](https://github.com/multica-ai/multica) — Official Multica repository
- [multica.ai/docs/cli](https://multica.ai/docs/cli) — Official CLI documentation
