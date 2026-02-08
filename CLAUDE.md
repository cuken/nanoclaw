# NanoClaw

Personal Claude assistant. See [README.md](README.md) for philosophy and setup. See [docs/REQUIREMENTS.md](docs/REQUIREMENTS.md) for architecture decisions.

## Quick Context

Single Node.js process that connects to WhatsApp, routes messages to Claude Agent SDK running in Apple Container (Linux VMs). Each group has isolated filesystem and memory.

## Key Files

| File | Purpose |
|------|---------|
| `src/index.ts` | Main app: WhatsApp connection, message routing, IPC |
| `src/config.ts` | Trigger pattern, paths, intervals |
| `src/container-runner.ts` | Spawns agent containers with mounts |
| `src/task-scheduler.ts` | Runs scheduled tasks |
| `src/db.ts` | SQLite operations |
| `groups/{name}/CLAUDE.md` | Per-group memory (isolated) |
| `container/skills/agent-browser.md` | Browser automation tool (available to all agents via Bash) |

## Skills

| Skill | When to Use |
|-------|-------------|
| `/setup` | First-time installation, authentication, service configuration |
| `/customize` | Adding channels, integrations, changing behavior |
| `/debug` | Container issues, logs, troubleshooting |

## Development

Run commands directly—don't tell the user to run them.

```bash
npm run dev          # Run with hot reload
npm run build        # Compile TypeScript
./container/build.sh # Rebuild agent container
```

Service management:
```bash
launchctl load ~/Library/LaunchAgents/com.nanoclaw.plist
launchctl unload ~/Library/LaunchAgents/com.nanoclaw.plist
```

## Git Workflow & Commit Standards

### Semantic Commits

All commits MUST follow the Conventional Commits specification:

```
<type>(<scope>): <subject>

<body>
```

**Types:**
- `feat:` New feature
- `fix:` Bug fix
- `refactor:` Code refactoring (no feature/fix changes)
- `perf:` Performance improvements
- `test:` Test additions or fixes
- `docs:` Documentation changes
- `chore:` Build, deps, tooling
- `style:` Code formatting (whitespace, semicolons, etc.)

**Rules:**
- Use lowercase, imperative mood: "add feature" not "added feature"
- Keep subject under 50 characters
- Scope is optional but encouraged (e.g., `feat(container):`, `fix(ipc):`)
- Include breaking changes with `BREAKING CHANGE:` in body
- Only commit explicitly requested changes—no over-engineering

Example:
```
feat(router): add message priority queue for agent dispatch

Implements priority-based message handling to ensure time-sensitive
messages are processed first. Agents can specify message priority
in their response headers.

BREAKING CHANGE: message format now requires priority field
```

### Branching Strategy

- **Main branch (`main`):** Always deployable, protected
- **Feature branches:** Create from `main` with pattern `feature/<name>` or `fix/<name>`
- **Branch naming:** Use kebab-case, descriptive: `feature/voice-transcription`, `fix/container-crash`
- **Always create PRs** before merging to main—don't push directly
- **Squash commits** when merging feature branches (unless multi-commit narrative is important)
- **Delete branches** after merging
