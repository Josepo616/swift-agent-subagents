# Swift Agent Subagents

Claude Code agents that perform real work on Swift/SwiftUI codebases following **Ravn's architecture**.

These agents are the **Action Layer** — they dynamically load rules from [swift-agent-skills](https://github.com/Josepo616/swift-agent-skills) at runtime and apply them as composed, automated workflows.

## Architecture

```
swift-agent-subagents (ACTION LAYER)
  Specialized agents that think and develop. Loads skills at runtime for latest rules.
  ▼ reads at runtime
swift-agent-skills (KNOWLEDGE LAYER)
  Source of truth for best practices. 16+ skills as SKILL.md files.
  Installed at: .claude/skills/{NN}-{name}/SKILL.md
```

**Skills** = source of truth ("here's how to do X correctly")
**Agents** = specialized executors ("do X for me, correctly, the Ravn way")

### Dynamic Skill Loading

Agents do **not** embed static copies of skill rules. Instead, each agent's first step is to **read the SKILL.md files** from the project's installed `swift-agent-skills`. This means:

- Updating a skill in `swift-agent-skills` **automatically updates** what agents enforce
- No manual sync between repos needed
- Agents always use the **latest version** of the rules
- If skills aren't installed, agents fall back to core constraints and warn the user

## Installation

### Option A — Add marketplaces and install (recommended for teams)

**Step 1** — Register both repos as plugin marketplaces:

```
/plugin marketplace add Josepo616/swift-agent-skills
/plugin marketplace add Josepo616/swift-agent-subagents
```

**Step 2** — Install the skills (knowledge layer):

```
/plugin install swift-agent-skills@Josepo616/swift-agent-skills
```

**Step 3** — Install the agents (action layer):

```
/plugin install swift-agent-subagents@Josepo616/swift-agent-subagents
```

### Option B — Load from local directories (for development/testing)

If you have the repos cloned locally:

```bash
claude --plugin-dir /path/to/swift-agent-skills --plugin-dir /path/to/swift-agent-subagents
```

Both repos are now available in your project via Claude Code's plugin system.

## Available Agents

### Tier 1: High Impact

| Agent | Purpose | Skills Loaded |
|-------|---------|---------------|
| `/feature-architect <name>` | Architects a complete feature module (Model, Service, ViewModel, View, Tests) | 01, 04, 05, 06, 07, 09, 10, 13, 15 |
| `/code-reviewer <path>` | Reviews Swift code against all Ravn conventions | ALL (01-16+) |
| `/test-engineer <path>` | Engineers comprehensive Swift Testing test suites | 10, 15, 07 + optional 01, 03, 06, 09 |

### Tier 2: Strong Additions (Coming Soon)

| Command | Purpose |
|---------|---------|
| `/audit-security [path]` | Scan for security anti-patterns |
| `/audit-performance [path]` | Find SwiftUI performance issues |
| `/audit-accessibility [path]` | Check accessibility compliance |
| `/build-network-layer` | Scaffold a complete networking layer |

## The Ravn Way

Every agent enforces these non-negotiable defaults:

| Decision | The Ravn Way |
|----------|-------------|
| Architecture | MVVM + Service-Oriented Architecture |
| ViewModel rule | 1 View : 1 ViewModel (strict) |
| Service design | Protocol contract + concrete implementation |
| Observation | `@Observable` + `@MainActor` (iOS 17+) |
| Testing framework | Swift Testing (`import Testing`) |
| DI approach | Constructor injection via protocols |
| Feature structure | Model / Service / ViewModel / View folders |
| State modeling | `Loadable<T>` enum (idle/loading/loaded/failed) |

## Project Structure

```
swift-agent-subagents/
├── .claude-plugin/
│   └── plugin.json               # Plugin metadata for distribution
├── agents/                        # Specialized agents
│   ├── feature-architect.md       # /feature-architect <name>
│   ├── code-reviewer.md           # /code-reviewer <path>
│   └── test-engineer.md           # /test-engineer <path>
├── .claude/
│   └── subagents-project-spec.md  # Project specification
├── knowledge/
│   └── skill-reference-map.md     # Maps skills to agent dependencies
├── README.md
└── LICENSE
```

## License

MIT
