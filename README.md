# Pi Memory Extension

> A persistent memory extension for Pi Coding Agent.
> Human-curated, Markdown-stored, Git-versioned long-term knowledge for Pi.
> v1 focuses on Pi only — not a replacement for Codex / Claude memory systems.

## Philosophy

1. **Human decides what becomes memory.** Only human-approved knowledge enters authoritative files.
2. **Markdown is the source of truth.** Plain text, Git-diffable, reviewable, revertible.
3. **Session history is not memory.** Pi JSONL stores raw conversations; `memory/` stores distilled knowledge.

## Installation

```bash
# From npm (recommended)
pi install npm:pi-memory-extension

# Or from GitHub
pi install git:git@github.com:appleface2050/pi-memory-extension.git
pi install git:https://github.com/appleface2050/pi-memory-extension.git

# Restart Pi or /reload
```

## Quick Start

```bash
# 1. Initialize Global Memory (personal preferences, cross-project knowledge)
/memory:init global

# 2. Initialize Workspace Memory (project-specific knowledge)
cd your-project
/memory:init workspace

# 3. Check status
/memory:status
/memory:list
```

## Commands

| Command | Description |
|---------|-------------|
| `/memory:init global` | Initialize `~/.pi/memory/` (personal layer) |
| `/memory:init workspace` | Initialize `.pi/memory/` in current project |
| `/memory:status` | Show detailed loaded memory info |
| `/memory:list` | Show file counts for Global and Workspace |
| `/memory:refresh` | Reload all memory files |
| `/memory:checkpoint` | Generate session summary to workspace inbox (requires workspace) |
| `/memory:promote <file> <targetDir> <targetFile>` | Promote inbox content to authoritative file |
| `/memory:clear-task` | Clear state/current-task.md (interrupt recovery done) |

## Storage Layout

### Global Memory (`~/.pi/memory/`)

```
~/.pi/memory/
├── user/                       # Personal preferences
│   ├── preferences.md
│   ├── coding-style.md
│   └── tools.md
├── facts/                      # Background facts
│   ├── environment.md
│   └── references.md
├── knowledge/                  # Cross-project knowledge
│   ├── decisions.md
│   ├── patterns.md
│   ├── lessons.md
│   └── bugs.md
├── state/                      # Working state
│   └── current-task.md
├── inbox/                      # Pending memory candidates
└── archive/                    # Historical archive
```

### Workspace Memory (`project/.pi/memory/`)

```
project/.pi/memory/
├── decisions.md
├── patterns.md
├── lessons.md
├── experiments.md
├── inbox/
└── archive/
```

## Memory Merge Behavior

When both Global and Workspace memory are loaded, they are merged with the following priority:

1. **Working state** (`state/current-task.md`) — always injected first, highest priority
2. **Workspace Memory** — fully preserved, takes precedence over Global
3. **Global Memory** — included if budget allows

**File override by basename**: If Workspace has a file named `decisions.md`, it overrides *any* Global file with the same basename (e.g., `knowledge/decisions.md`), regardless of subdirectory. This ensures project-specific decisions take precedence over generic ones.

If total content exceeds the character budget (default 8000), Workspace is fully preserved and Global is truncated from the tail.

## Design Document

See [docs/design.md](docs/design.md) for the full design specification.

## Configuration

The injected memory block is capped so it cannot crowd out the rest of the
context. Two settings raise those caps:

| Setting | Default | Meaning |
| --- | --- | --- |
| `maxTotalChars` | `8000` | Total characters of injected memory: the workspace, then the global layer, then the state file |
| `maxFileChars` | `4000` | Per-file cap applied before the total is considered |

Resolution order, highest first:

1. Environment: `PI_MEMORY_MAX_TOTAL_CHARS`, `PI_MEMORY_MAX_FILE_CHARS`
2. `~/.pi/agent/pi-memory-extension.json`

   ```json
   { "maxTotalChars": 25000, "maxFileChars": 14000 }
   ```

3. The defaults above

The caps are re-read at session start, so an edit applies to the next session
without restarting Pi. The startup notice reports the values in force:

```
🧠 Pi Memory: 0 global + 5 workspace = 5 files (caps 25000/14000)
```

A missing or unparseable config file falls back to the defaults rather than
failing the session.

Note that the workspace is preserved first when the total is exceeded, so a
project store can be large while the global layer absorbs the truncation. So
neither setting is a limit on what is stored on disk, only on what is injected.

## License

MIT
