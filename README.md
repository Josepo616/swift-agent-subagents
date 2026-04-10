# Swift Agent Subagents

Claude Code subagents that perform real work on Swift/SwiftUI codebases following **Ravn's architecture**.

These subagents are the **Action Layer** — they dynamically load rules from [swift-agent-skills](https://github.com/Josepo616/swift-agent-skills) at runtime and apply them as composed, automated workflows.

## Architecture

```
swift-agent-subagents (ACTION LAYER)
  Performs work on codebases. Loads skills at runtime for latest rules.
  ▼ reads at runtime
swift-agent-skills (KNOWLEDGE LAYER)
  Teaches Claude best practices. 16+ skills as SKILL.md files.
  Installed at: .claude/skills/{NN}-{name}/SKILL.md
```

**Skills** = passive knowledge ("here's how to do X correctly")
**Subagents** = active executors ("do X for me, correctly, the Ravn way")

### Dynamic Skill Loading

Subagents do **not** embed static copies of skill rules. Instead, each subagent's first step is to **read the SKILL.md files** from the project's installed `swift-agent-skills`. This means:

- Updating a skill in `swift-agent-skills` **automatically updates** what subagents enforce
- No manual sync between repos needed
- Subagents always use the **latest version** of the rules
- If skills aren't installed, subagents fall back to core constraints and warn the user

## Prerequisites

Install `swift-agent-skills` in your project first:

```bash
/install-plugin Josepo616/swift-agent-skills
```

## Available Subagents

### Tier 1: High Impact

| Command | Purpose | Skills Loaded |
|---------|---------|---------------|
| `/scaffold-feature <name>` | Generate a complete feature module (Model, Service, ViewModel, View, Tests) | 01, 04, 05, 06, 07, 09, 10, 13, 15 |
| `/swift-review <path>` | Review Swift code against all Ravn conventions | ALL (01-16+) |
| `/gen-tests <path>` | Generate Swift Testing tests for existing code | 10, 15, 07 + optional 01, 03, 06, 09 |

### Tier 2: Strong Additions (Coming Soon)

| Command | Purpose |
|---------|---------|
| `/audit-security [path]` | Scan for security anti-patterns |
| `/audit-performance [path]` | Find SwiftUI performance issues |
| `/audit-accessibility [path]` | Check accessibility compliance |
| `/build-network-layer` | Scaffold a complete networking layer |

## The Ravn Way

Every subagent enforces these non-negotiable defaults:

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

## Installation

```bash
# Clone the repository
git clone https://github.com/Josepo616/swift-agent-subagents

# Copy commands into your project
cp -r swift-agent-subagents/.claude/commands/ your-project/.claude/commands/
```

## Project Structure

```
swift-agent-subagents/
├── .claude/
│   ├── commands/                  # User-invocable subagents
│   │   ├── scaffold-feature.md    # /scaffold-feature <name>
│   │   ├── swift-review.md        # /swift-review <path>
│   │   └── gen-tests.md           # /gen-tests <path>
│   └── subagents-project-spec.md  # Project specification
├── knowledge/
│   └── skill-reference-map.md     # Maps skills to subagent dependencies
├── README.md
└── LICENSE
```

## License

MIT
