# Skill Reference Map

> Maps skill numbers to their purpose and which subagents depend on them.
> Subagents load skills dynamically at runtime from `.claude/skills/`.

---

## Installed Skills Path

Skills are installed at: `.claude/skills/{NN}-{skill-name}/SKILL.md`

Alternative nested path: `.claude/skills/{NN}-{skill-name}/skills/{skill-name}/SKILL.md`

---

## Skill Catalog

| # | Skill | Purpose | Subagent Dependencies |
|---|-------|---------|----------------------|
| 01 | Naming Conventions | Types, functions, variables, protocols naming | scaffold-feature, swift-review, gen-tests |
| 02 | Observable Architecture | @Observable, @State, @Bindable patterns | swift-review |
| 03 | Swift Concurrency | async/await, actors, Sendable, @MainActor | swift-review, gen-tests |
| 04 | Project Structure | Feature folders, file organization | scaffold-feature, swift-review |
| 05 | Protocol-Oriented Programming | Protocol design, composition, generics | scaffold-feature, swift-review |
| 06 | Error Handling | Domain errors, LocalizedError, typed throws | scaffold-feature, swift-review, gen-tests |
| 07 | Dependency Injection | Constructor injection, protocols, @Environment | scaffold-feature, swift-review, gen-tests |
| 08 | SwiftUI Performance | Lazy containers, stable IDs, body optimization | swift-review |
| 09 | Access Control | private(set), access levels, MARK sections | scaffold-feature, swift-review, gen-tests |
| 10 | Swift Testing | @Suite, @Test, #expect, #require, parameterized | scaffold-feature, swift-review, gen-tests |
| 11 | Accessibility | VoiceOver labels, Dynamic Type, touch targets | swift-review |
| 12 | Previews | Preview states, preview data, accessibility previews | swift-review |
| 13 | Codable & Data Modeling | Structs, Identifiable, Codable, Sendable | scaffold-feature, swift-review |
| 14 | SwiftData | @Model, @Query, #Predicate, relationships | swift-review |
| 15 | MVVM + Service Architecture | Core architecture, ViewModel, Service, Loadable | scaffold-feature, swift-review, gen-tests |
| 16 | Swift Security | Keychain, CryptoKit, biometrics, Secure Enclave | swift-review |

## Future Skills (when built)

| # | Skill | Planned Subagent Dependencies |
|---|-------|-------------------------------|
| 17 | Navigation | scaffold-feature |
| 18 | Networking | build-network-layer |
| 22 | Figma Integration | figma-to-feature |

---

## How Subagents Use Skills

Each subagent has a **Step 0** that reads specific SKILL.md files before doing work:

- **scaffold-feature**: Reads 9 skills (01, 04, 05, 06, 07, 09, 10, 13, 15)
- **swift-review**: Reads ALL skills (01-16+) — globs for any new ones
- **gen-tests**: Reads 3 required + 4 optional skills (10, 15, 07 + 06, 03, 01, 09)

This means updating a skill in `swift-agent-skills` automatically updates what subagents enforce — no manual sync needed.
