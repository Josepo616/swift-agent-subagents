# Swift Agent Subagents — Project Specification

> **Purpose:** This document is the single source of context for building the `swift-agent-subagents` repo. Load this at the start of any Claude conversation to avoid losing context.

> **Last updated:** 2026-04-09

---

## 1. What We're Building

A **separate repository** (`swift-agent-subagents`) containing Claude Code subagents that perform real work on Swift/SwiftUI codebases. These subagents are **connected to** the `swift-agent-skills` marketplace — they internalize the rules defined in the skills and apply them as composed, automated workflows.

### The Two-Layer Ecosystem

```
┌──────────────────────────────────────────────────────────┐
│  swift-agent-subagents                                    │
│  ACTION LAYER — performs work on codebases                │
│  Encodes the Ravn way as its opinionated default          │
│  Lives in: .claude/commands/*.md (user-invocable)         │
├──────────────────────────────────────────────────────────┤
│          ▼ references / composes                          │
├──────────────────────────────────────────────────────────┤
│  swift-agent-skills                                       │
│  KNOWLEDGE LAYER — teaches Claude best practices          │
│  16+ skills as SKILL.md files with plugin.json metadata   │
│  Lives in: skills/NN-skill-name/skills/name/SKILL.md      │
└──────────────────────────────────────────────────────────┘
```

**Key distinction:**
- **Skills** = passive knowledge ("here's how to do X correctly")
- **Subagents** = active executors ("do X for me, correctly, the Ravn way")

---

## 2. The Ravn Way — Core Opinions

Every subagent enforces these as non-negotiable defaults. These are the architectural decisions that define Ravn's iOS team practices:

| Decision | The Ravn Way | NOT This |
|---|---|---|
| Architecture | MVVM + Service-Oriented Architecture | TCA, VIPER, MV, Clean Architecture |
| ViewModel rule | 1 View : 1 ViewModel (strict) | Shared ViewModels between screens |
| Service design | Protocol contract + concrete implementation | Concrete types, singletons |
| Observation | `@Observable` + `@MainActor` (iOS 17+) | `ObservableObject` + `@Published` |
| Reactive layer | Combine where async streams needed | RxSwift, third-party reactive |
| Testing framework | Swift Testing (`import Testing`) | XCTest |
| Project config | XcodeGen (`project.yml`) | Manual .xcodeproj management |
| DI approach | Constructor injection via protocols | Service locator, @Environment in VMs |
| Feature structure | Model / Service / ViewModel / View folders | Flat files, type-based folders |
| State modeling | `Loadable<T>` enum (idle/loading/loaded/failed) | Separate Bool flags |

### Source of Truth for Each Rule

These map directly to existing skills in `swift-agent-skills`:

| Rule | Skill Reference |
|---|---|
| Naming | Skill 01 — Naming Conventions |
| @Observable patterns | Skill 02 — Observable Architecture |
| async/await, actors, Sendable | Skill 03 — Swift Concurrency |
| Feature folder layout | Skill 04 — Project Structure |
| Protocol-oriented design | Skill 05 — Protocol-Oriented Programming |
| Domain errors, typed throws | Skill 06 — Error Handling |
| Constructor injection | Skill 07 — Dependency Injection |
| SwiftUI body optimization | Skill 08 — SwiftUI Performance |
| Access levels | Skill 09 — Access Control |
| Swift Testing framework | Skill 10 — Swift Testing |
| VoiceOver, Dynamic Type | Skill 11 — Accessibility |
| Preview states | Skill 12 — Previews |
| JSON, Codable patterns | Skill 13 — Codable & Data Modeling |
| @Model, @Query, #Predicate | Skill 14 — SwiftData |
| MVVM + SOA (core architecture) | Skill 15 — MVVM + Service Architecture |
| Keychain, CryptoKit, biometrics | Skill 16 — Swift Security |

Future skills (17-25) will feed into subagents as they're implemented.

---

## 3. Subagent Catalog — Planned

### Tier 1: High Impact (Build First)

#### `scaffold-feature`
- **Purpose:** Generate a complete feature module from a name
- **Composes:** Skills 01, 04, 06, 07, 15
- **Input:** Feature name (e.g., "Orders", "UserProfile")
- **Output:** Full folder structure with:
  - Model structs (Codable, Sendable, Identifiable)
  - Service protocol + implementation
  - Mock service for testing
  - ViewModel with Loadable state, task cancellation, error handling
  - View with .task, .refreshable, error alert
  - Swift Testing test file for ViewModel
- **Example:** `/scaffold-feature Orders` generates `Features/Orders/Model/`, `Service/`, `ViewModel/`, `View/`, and `Tests/`

#### `swift-review`
- **Purpose:** Review Swift code against all Ravn conventions
- **Composes:** Skills 01-16 (all available)
- **Input:** File path, directory, or PR diff
- **Output:** Structured review with:
  - Violations found (grouped by skill)
  - Severity (error / warning / suggestion)
  - Fix recommendation with code
  - What's done well (positive reinforcement)
- **Key behaviors:**
  - Does NOT auto-fix — only reports
  - References specific skill rules
  - Focuses on Ravn-specific patterns, not generic Swift lint

#### `gen-tests`
- **Purpose:** Generate Swift Testing tests for existing code
- **Composes:** Skills 10, 15, 07
- **Input:** File or type to test (ViewModel, Service)
- **Output:**
  - Mock services (protocol conformances)
  - Test suite with @Suite, @Test
  - Tests for success, failure, edge cases
  - @MainActor for ViewModel tests
  - Arrange-Act-Assert structure
- **Key behaviors:**
  - Detects if target is ViewModel or Service and adjusts strategy
  - Uses Swift Testing (never XCTest)
  - Creates mock based on existing protocol

### Tier 2: Strong Additions

#### `audit-security`
- **Purpose:** Scan codebase for security anti-patterns
- **Composes:** Skills 16, 09
- **Input:** Directory or file path
- **Output:** Security report with:
  - Hardcoded secrets or API keys
  - Sensitive data in UserDefaults (should be Keychain)
  - Missing access control on sensitive types
  - Insecure network calls
  - OWASP Mobile Top 10 alignment
- **Official references:** Apple Security framework docs, OWASP MASVS

#### `audit-performance`
- **Purpose:** Find SwiftUI performance anti-patterns
- **Composes:** Skills 08, 03
- **Input:** SwiftUI views directory
- **Output:** Performance report with:
  - Heavy body computations
  - Missing lazy containers
  - Unstable ForEach identity
  - Main actor blocking
  - Unnecessary state invalidation

#### `audit-accessibility`
- **Purpose:** Check accessibility compliance
- **Composes:** Skills 11, 12
- **Input:** SwiftUI views directory
- **Output:** Accessibility report with:
  - Missing accessibility labels
  - Images without descriptions
  - Insufficient touch targets
  - Dynamic Type support gaps
  - Color contrast issues

#### `build-network-layer`
- **Purpose:** Scaffold a complete networking layer
- **Composes:** Skills 18 (when built), 15, 06, 13
- **Input:** API base URL, optional endpoint list
- **Output:**
  - APIClientProtocol + APIClient implementation
  - Request/Response types
  - Endpoint enum pattern
  - Error mapping (HTTP status → domain errors)
  - Token management skeleton
  - Mock API client for testing

### Tier 3: Specialized

#### `figma-to-feature`
- **Purpose:** Take Figma design specs and generate a full feature
- **Composes:** Skills 22 (when built), 15, 01, 04
- **Input:** Figma component description or screenshot
- **Output:** Full feature module with views matching the design

#### `migrate-observable`
- **Purpose:** Migrate ObservableObject code to @Observable
- **Composes:** Skills 02, 03
- **Input:** File or directory with ObservableObject classes
- **Output:** Migrated code using @Observable, removed @Published, updated views

---

## 4. Subagent File Structure

### Repository Layout

```
swift-agent-subagents/
├── .claude/
│   └── commands/                    # User-invocable subagents
│       ├── scaffold-feature.md      # /scaffold-feature <name>
│       ├── swift-review.md          # /swift-review <path>
│       ├── gen-tests.md             # /gen-tests <path>
│       ├── audit-security.md        # /audit-security [path]
│       ├── audit-performance.md     # /audit-performance [path]
│       ├── audit-accessibility.md   # /audit-accessibility [path]
│       ├── build-network-layer.md   # /build-network-layer
│       ├── figma-to-feature.md      # /figma-to-feature <description>
│       └── migrate-observable.md    # /migrate-observable <path>
├── .claude-plugin/
│   └── marketplace.json             # Registry of all subagents
├── knowledge/                       # Shared prompt fragments
│   ├── ravn-architecture.md         # The Ravn way — core rules (extracted from skills)
│   ├── ravn-conventions.md          # Naming, access control, file structure rules
│   ├── swift-testing-patterns.md    # Testing rules, mock patterns
│   └── apple-references.md         # Links to official Apple documentation
├── README.md
└── LICENSE
```

### Subagent File Format (.claude/commands/*.md)

Each subagent follows this template:

```markdown
---
description: One-line description of what this subagent does
user_invocable: true
---

# Role and Context

You are a [specific role] for Swift/SwiftUI projects following Ravn's architecture.

## Knowledge Sources (from swift-agent-skills)

[Embedded rules extracted from relevant skills — not links, but the actual rules
the agent needs to follow. This makes the subagent self-contained.]

## Official References

[Apple documentation URLs for the relevant frameworks]

## Task

[What the subagent does, step by step]

## Input

$ARGUMENTS — [what the user provides]

## Output Format

[Exact structure of what gets generated]

## Rules

[Non-negotiable constraints — the Ravn way]

## Examples

[One concrete input → output example]
```

### Key Design Decision: Dynamic Skill Loading at Runtime

Subagents **read SKILL.md files at runtime** from the project's installed `swift-agent-skills` (`.claude/skills/`). Each subagent's "Step 0" reads the specific skills it needs before doing any work. This means:

- **Always up-to-date** — updating a skill automatically updates what subagents enforce
- **No manual sync** — no need to update subagents when skills change
- **Curated per subagent** — each subagent lists which skills it reads (not all 16)
- **Graceful fallback** — if skills aren't installed, subagents warn the user and fall back to hardcoded core constraints
- **Skills repo is a prerequisite** — users should install `swift-agent-skills` first for full functionality
- The `knowledge/` directory holds the skill reference map (which skills exist and which subagents depend on them)

---

## 5. How the Connection Works in Practice

### For Users (Installing Both Repos)

```bash
# Install skills (knowledge layer)
/plugin marketplace add Josepo616/swift-agent-skills

# Install subagents (action layer)
# Option 1: Clone and symlink commands
git clone https://github.com/Josepo616/swift-agent-subagents
# Copy or symlink .claude/commands/ into their project

# Option 2: If marketplace supports it
/plugin marketplace add Josepo616/swift-agent-subagents
```

### For Developers (Maintaining Both Repos)

When a skill is updated:
1. Identify which subagents reference that skill (see mapping in Section 3)
2. Update the embedded rules in those subagents
3. Update the `knowledge/` shared fragments if the change is cross-cutting

When a new skill is added:
1. Determine which existing subagents could benefit
2. Optionally create a new subagent if the skill enables a new workflow
3. Update this spec document

---

## 6. Build Order — Implementation Priority

### Phase 1: Foundation
1. **Repository setup** — README, LICENSE, directory structure, marketplace.json
2. **`knowledge/ravn-architecture.md`** — Extract core Ravn rules from Skill 15
3. **`knowledge/ravn-conventions.md`** — Extract from Skills 01, 04, 09
4. **`scaffold-feature`** — First subagent, highest impact, proves the architecture

### Phase 2: Review & Testing
5. **`swift-review`** — Composes all skills, high daily-use value
6. **`gen-tests`** — Natural companion to scaffold-feature

### Phase 3: Audits
7. **`audit-security`** — Standalone, high value
8. **`audit-performance`** — Standalone, high value
9. **`audit-accessibility`** — Standalone, high value

### Phase 4: Advanced Workflows
10. **`build-network-layer`** — Depends on Skill 18 being built first
11. **`migrate-observable`** — Migration utility
12. **`figma-to-feature`** — Depends on Skill 22 being built first

---

## 7. Quality Criteria for Each Subagent

Before a subagent is considered complete, it must:

- [ ] Generate code that passes `swift-review` with zero errors
- [ ] Follow every Ravn convention without exception
- [ ] Include at least one worked example in the prompt
- [ ] Be self-contained (works without skills repo installed)
- [ ] Handle edge cases gracefully (empty input, invalid paths, existing files)
- [ ] Produce code that compiles (correct imports, types, syntax)
- [ ] Use Swift Testing for any generated tests (never XCTest)
- [ ] Include proper access control (private(set), internal defaults)
- [ ] Use @Observable (not ObservableObject) for any generated ViewModels
- [ ] Reference official Apple documentation where applicable

---

## 8. Open Questions

- [ ] **Marketplace format:** Will `swift-agent-subagents` use the same `.claude-plugin` marketplace format, or does Claude Code have a different mechanism for distributing commands?
- [ ] **Versioning strategy:** Should subagent versions track the skills they embed? (e.g., subagent v1.2 embeds skill 15 v1.1)
- [ ] **Mono-install option:** Should there be a meta-repo or script that installs both repos at once?
- [ ] **Skill update automation:** Can we automate detecting when a skill changes and flagging which subagents need updating?
- [ ] **Context7 integration:** Should subagents use Context7 MCP to fetch latest Apple docs at runtime, or embed static references?

---

## 9. Repository Links

| Repo | Purpose | Status |
|---|---|---|
| `swift-agent-skills` | Knowledge layer — 16 skills | Active, skills 17-25 in roadmap |
| `swift-agent-subagents` | Action layer — subagents | Setting up repo (2026-04-09) |

---

## Appendix: Skill → Subagent Dependency Map

```
Skill 01 (Naming)        → scaffold-feature, swift-review, figma-to-feature
Skill 02 (Observable)    → migrate-observable, swift-review
Skill 03 (Concurrency)   → audit-performance, migrate-observable, swift-review
Skill 04 (Structure)     → scaffold-feature, swift-review, figma-to-feature
Skill 05 (Protocols)     → scaffold-feature, swift-review
Skill 06 (Errors)        → scaffold-feature, build-network-layer, swift-review
Skill 07 (DI)            → scaffold-feature, gen-tests, swift-review
Skill 08 (Performance)   → audit-performance, swift-review
Skill 09 (Access)        → audit-security, swift-review
Skill 10 (Testing)       → gen-tests, swift-review
Skill 11 (Accessibility) → audit-accessibility, swift-review
Skill 12 (Previews)      → audit-accessibility, swift-review
Skill 13 (Codable)       → build-network-layer, swift-review
Skill 14 (SwiftData)     → swift-review
Skill 15 (MVVM+SOA)      → scaffold-feature, gen-tests, swift-review, build-network-layer
Skill 16 (Security)      → audit-security, swift-review
Skill 17 (Navigation)*   → scaffold-feature (when built)
Skill 18 (Networking)*   → build-network-layer (when built)
Skill 22 (Figma)*        → figma-to-feature (when built)
```

*Future skills — subagents will be updated when these are implemented.
