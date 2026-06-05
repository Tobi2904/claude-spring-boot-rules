---
name: spring-boot-guidelines
description: Behavioral guidelines that make Claude write production-grade Spring Boot / Java backend code. Use whenever editing or generating Java files (especially `*.java`, `pom.xml`, or Spring Boot config), or whenever the user mentions controllers, services, entities, DTOs, mappers, pagination, validation, or error handling in a Spring/Java context. Enforces: no hardcoding (use Constants/Enums), thin controllers and fat services, MapStruct for all DTO/Entity mapping, Lombok for boilerplate, UUID v7 for primary keys, custom `PageResponse<T>` wrappers, Vietnamese user-facing error messages, proactive security and performance audits (SQLi, N+1), two-tier validation (stateless DTO annotations + business `Validator` strategies), and roleplay-based edge case coverage.
---

# Spring Boot / Java Backend Guidelines

This skill packages 15 behavioral rules for writing production-grade Spring Boot backend code. See [`../../CLAUDE.md`](../../CLAUDE.md) for the full rules and [`../../EXAMPLES.md`](../../EXAMPLES.md) for before/after code.

## Quick Reference

| # | Rule | One-line test |
|---|------|---------------|
| 1 | Think before coding | Did I state assumptions? |
| 2 | Simplicity first | Could a senior call this overcomplicated? |
| 3 | Surgical changes | Does every diff line trace to the user's request? |
| 4 | Goal-driven execution | Do I have a verifiable success criterion? |
| 5 | No hardcoding | Will a value change require editing >1 line? |
| 6 | Thin controllers | Is there business logic in the controller? |
| 7 | SOLID | Is this hard to unit test? |
| 8 | MapStruct mapping | Any manual `dto.setX(entity.getX())`? |
| 9 | Lombok | Any hand-written getters/setters/constructors? |
| 10 | UUID v7 | Are new IDs time-ordered? |
| 11 | Security & perf audits | Did I miss SQLi or N+1? |
| 12 | `PageResponse<T>` | Is the endpoint returning raw `Page<T>`? |
| 13 | Localized errors | Are user-facing messages in the end-user's language? |
| 14 | Edge cases | Did I list 2+ failure modes? |
| 15 | Two-tier validation | A DB-dependent check inside a `ConstraintValidator`, or a wall of private `validate*()`? |

## When to invoke

Auto-invoke this skill whenever:

- A file in `src/main/java/**` is being created or edited
- The user mentions Spring Boot, REST endpoints, JPA entities, DTOs, or mappers
- The user asks about pagination, authentication, validation, or error handling in a Java/Spring context
- `pom.xml` or `build.gradle` is being modified

## How to apply

1. Before coding, scan the user's prompt against rules 1–4 (planning rules)
2. While coding, enforce rules 5–13 and 15 (implementation rules)
3. Before declaring done, run rule 14 (edge case audit)

If any rule cannot be satisfied, stop and explain why before continuing.
