# CURSOR.md

Spring Boot / Java backend rules for Cursor IDE. Same content as `CLAUDE.md`, optimized for Cursor's rule-attachment system.

For auto-attached behavior, the rule file lives at `.cursor/rules/spring-boot.mdc`. This file is the human-readable reference.

---

## Rule 1 — Think Before Coding
State assumptions explicitly. If multiple interpretations exist, present them. If a simpler approach exists, push back. If unclear, ask before coding.

## Rule 2 — Simplicity First
Minimum code that solves the problem. No speculative features, abstractions, configurability, or error handling for impossible cases. If 200 lines could be 50, rewrite.

## Rule 3 — Surgical Changes
Touch only what the user asked. Don't refactor adjacent code or improve formatting. Match existing style. Remove only orphans your changes created.

## Rule 4 — Goal-Driven Execution
Every task gets verifiable success criteria. "Add validation" → "Tests for invalid inputs pass." "Fix bug" → "Failing test reproduces it, then passes."

## Rule 5 — No Hardcoding
Magic numbers, error keys, business literals → Constants class, config file, or Enum. If changing a value requires editing multiple lines, it's wrong.

## Rule 6 — Thin Controllers, Fat Services
Controllers: receive request, validate, call service, return response. Zero business logic, transformations, or DB queries in controllers.

## Rule 7 — SOLID & Clean Code
Especially Single Responsibility and Dependency Inversion. If new code is hard to unit test, refactor.

## Rule 8 — MapStruct for Mapping
No manual `dto.setX(entity.getX())`. Every Entity ↔ DTO ↔ Domain transformation goes through a MapStruct interface.

## Rule 9 — Lombok for Boilerplate
`@Getter`, `@Setter`, `@RequiredArgsConstructor`, `@Builder`. No hand-written getters/setters/constructors unless framework constraints force it.

## Rule 10 — UUID v7 for IDs
All entity primary keys use UUID v7. Time-ordered, index-friendly.

## Rule 11 — Proactive Security & Performance Audits
Spot SQLi, broken access control, N+1 queries, heavy loops → stop, report, propose fix before continuing.

## Rule 12 — Custom `PageResponse<T>`
Paginated endpoints return `PageResponse<T>`, never raw `Page<T>` from Spring Data.

## Rule 13 — Localized Error Messages
All user-facing error messages in the end-user's language. Default: Vietnamese (the original author's market). Customize for your audience, or delete this rule if your end-users read English natively.

## Rule 14 — Roleplay & Edge Cases
Before declaring "done," list at least 2 user-error scenarios and how the code handles them. No "happy path only."

## Rule 15 — Two-Tier Validation (Declarative + Strategy)

**Tier 1 (stateless):** null, blank, regex, size, range, enum, static cross-field → `jakarta.validation` annotations on request DTOs, triggered by `@Valid`/`@Validated` in the controller. Never re-check in the service. 

**Tier 2 (stateful/business):** uniqueness, balance, ownership, status transitions → keep out of the controller and NEVER inject a `Repository` into a `ConstraintValidator`. Model each business rule as a `@Component` implementing a shared `XxxValidator` interface, inject `List<XxxValidator>` into the service, and loop — adding a rule means adding a class (OCP), no service edit. Use `@Order` / `Ordered` when sequence matters, or a Chain of Responsibility for strict gating.

---

**Tip:** This file pairs with `.cursor/rules/spring-boot.mdc` which Cursor auto-attaches when editing `*.java`, `pom.xml`, or `build.gradle` files.
