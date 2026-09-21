# claude-spring-boot-rules

> A single `CLAUDE.md` file to make Claude Code (and other LLM coding assistants) write **production-grade Spring Boot / Java backend code** — instead of overcomplicated, hardcoded, locale-blind spaghetti.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Claude Code](https://img.shields.io/badge/Claude-Code-orange)](https://docs.claude.com/en/docs/claude-code)
[![Cursor](https://img.shields.io/badge/Cursor-supported-blue)](https://cursor.sh)
[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.x-brightgreen)](https://spring.io/projects/spring-boot)

🇻🇳 **[Đọc bằng tiếng Việt →](./README.vi.md)**

> **Built on prior art.** Rules #1–#4 are adapted from [`forrestchang/andrej-karpathy-skills`](https://github.com/forrestchang/andrej-karpathy-skills) (MIT, © Forrest Chang), themselves derived from Andrej Karpathy's observations on LLM coding pitfalls. Rules #5–#19 are original additions for Spring Boot / Java backend conventions. See [`NOTICE`](./NOTICE) for full attribution.

---

## Why this exists

The upstream [`andrej-karpathy-skills`](https://github.com/forrestchang/andrej-karpathy-skills) repo gives you 4 powerful, language-agnostic rules for getting better output from Claude Code. They're great.

But out of the box, LLMs writing **Spring Boot / Java backends specifically** still tend to:

- ❌ Hardcode magic strings and status values everywhere
- ❌ Cram business logic into controllers
- ❌ Manually copy `entity.getX()` → `dto.setX(x)` instead of using MapStruct
- ❌ Hand-write getters/setters/constructors instead of using Lombok
- ❌ Use random UUID v4 — terrible for B-tree indexes
- ❌ Leak Spring's internal `Page<T>` shape to API consumers
- ❌ Return error messages in the wrong language for the actual end-user
- ❌ Use the same thread-pool strategy for both blocking I/O and CPU-heavy work
- ❌ Duplicate exception statuses and messages across services

This repo **extends** the Karpathy guidelines with 15 additional rules that fix these Spring Boot–specific defaults. Opinionated for Java/Spring Boot 3.x projects.

> **Note on localization:** Rule #13 ("Localized Error Messages") ships with Vietnamese as the default example, because that's the author's market. **It's designed to be customized or removed** if your end-users speak another language. All other 18 rules are locale-agnostic.

## What's inside

| File | Purpose |
|------|---------|
| [`CLAUDE.md`](./CLAUDE.md) | The 19 rules. Drop into your project root, Claude Code reads it automatically. |
| [`CURSOR.md`](./CURSOR.md) | Same rules, formatted for Cursor IDE. |
| [`EXAMPLES.md`](./EXAMPLES.md) | Concrete before/after code for each rule. |
| [`.claude-plugin/plugin.json`](./.claude-plugin/plugin.json) | Install as a Claude Code plugin. |
| [`.cursor/rules/spring-boot.mdc`](./.cursor/rules/spring-boot.mdc) | Auto-attached Cursor rule. |
| [`skills/spring-boot-guidelines/`](./skills/spring-boot-guidelines/) | Skill format for `skills.sh` compatibility. |
| [`NOTICE`](./NOTICE) | Attribution and credits for derivative content. |

## The 19 Rules at a glance

**Upstream (from `andrej-karpathy-skills`, MIT © Forrest Chang):**

1. **Think Before Coding** — Surface assumptions, never silently pick interpretations
2. **Simplicity First** — Minimum code, nothing speculative
3. **Surgical Changes** — Touch only what the user asked for
4. **Goal-Driven Execution** — Define verifiable success criteria

**Original additions (Spring Boot / Java backend, MIT © Tobi2904):**

5. **No Hardcoding** — Constants, Enums, config files. No magic literals
6. **Thin Controllers, Fat Services** — Zero business logic in controllers
7. **SOLID & Clean Code** — Especially SRP and DIP
8. **MapStruct Mapping** — No manual DTO ↔ Entity conversion
9. **Lombok Everywhere** — No hand-written getters/setters/constructors
10. **UUID v7 for IDs** — Time-ordered, index-friendly
11. **Proactive Security & Performance Audits** — Flag SQLi, N+1, etc. before continuing
12. **Custom `PageResponse<T>`** — Never leak Spring's `Page<T>` to API consumers
13. **Localized Error Messages** — End-user-facing messages in the user's actual language *(defaults to Vietnamese as an example — customizable or removable)*
14. **User-Centric Roleplay & Edge Cases** — List 2+ failure modes before declaring "done"
15. **Two-Tier Validation** — Stateless checks on DTOs via `jakarta.validation`; business rules as injectable `Validator` strategies, not service bloat
16. **Split Queries over `JOIN FETCH`** — Never `JOIN FETCH` a collection; fetch children separately and assemble in memory with a `HashMap`
17. **`Set<>` for Entity Collections** — Use `Set<>`, not `List<>`, for entity associations — no duplicates, correct many-to-many, no `MultipleBagFetchException`
18. **Executors by Workload Type** — Virtual threads for I/O; a fixed pool sized to the background CPU budget for CPU-heavy work
19. **Module-Specific Exception Factories** — Centralize each module's exception statuses and user-facing messages behind reusable factory methods

Full details in [`CLAUDE.md`](./CLAUDE.md). Real-world examples in [`EXAMPLES.md`](./EXAMPLES.md).

---

## Quick start

### Option 1 — Claude Code (recommended)

Drop the file into your Spring Boot project root:

```bash
cd your-spring-boot-project
curl -O https://raw.githubusercontent.com/Tobi2904/claude-spring-boot-rules/main/CLAUDE.md
```

Claude Code reads `CLAUDE.md` automatically on every session in that directory. Done.

### Option 2 — Cursor IDE

```bash
mkdir -p .cursor/rules
curl -o .cursor/rules/spring-boot.mdc \
  https://raw.githubusercontent.com/Tobi2904/claude-spring-boot-rules/main/.cursor/rules/spring-boot.mdc
```

Cursor auto-attaches the rule based on file patterns.

### Option 3 — Project-wide global

Put `CLAUDE.md` in your home directory at `~/.claude/CLAUDE.md` and it applies to every project.

---

## Philosophy

These rules are **opinionated**, not universal. They reflect lessons from shipping Spring Boot APIs to non-English-speaking markets:

- **Constants > config files > hardcoded values.** Always.
- **Controllers are dumb pipes.** They take HTTP in, hand it to a service, return HTTP out.
- **Mappers and Lombok eliminate ~30% of typical Java boilerplate.** Use them.
- **End-user error messages are a UX surface**, not a debugging tool. Write them in your user's actual language.
- **"Happy path only" is how production bugs are born.** Roleplay the chaotic user.

If you disagree with a rule, delete it from your fork. That's the point of a single readable file. **Rule #13 in particular is meant to be customized for your audience** — replace Vietnamese with your target language, or delete entirely if your end-users speak English.

---

## Contributing

PRs welcome, especially:

- New examples in `EXAMPLES.md` showing a rule preventing a real bug
- Translations of the rules into other languages
- Additions / refinements based on Spring Boot 3.x patterns

Open an issue first for big changes.

---

## Credits & Attribution

This work stands on the shoulders of others. Full credit:

- **[Andrej Karpathy](https://x.com/karpathy)** — for the original [January 26, 2026 observations](https://x.com/karpathy) on LLM coding pitfalls that inspired everything downstream. He has not endorsed this work.
- **[Forrest Chang (@forrestchang)](https://github.com/forrestchang)** — for distilling those observations into the original 4-rule `CLAUDE.md` ([`forrestchang/andrej-karpathy-skills`](https://github.com/forrestchang/andrej-karpathy-skills), MIT). Rules #1–#4 in this repo are adapted from his work.
- **[multica-ai](https://github.com/multica-ai)** — for the [organization mirror](https://github.com/multica-ai/andrej-karpathy-skills) whose repo structure (`.claude-plugin/`, `.cursor/rules/`, `skills/`, `CURSOR.md`, dual-language READMEs) inspired the layout of this repo.

See [`NOTICE`](./NOTICE) for a precise file-by-file breakdown of what's upstream and what's original.

Maintained by [@Tobi2904](https://github.com/Tobi2904).

## License

MIT — see [`LICENSE`](./LICENSE).
