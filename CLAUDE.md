# CLAUDE.md

Behavioral guidelines to reduce common LLM coding mistakes for Spring Boot / Java backend projects. Merge with project-specific instructions as needed.

> **Attribution:** Rules #1–#4 are derived from [`forrestchang/andrej-karpathy-skills`](https://github.com/forrestchang/andrej-karpathy-skills) (MIT License, © Forrest Chang), based on Andrej Karpathy's observations on LLM coding pitfalls.
> Rules #5–#17 are original additions by [@Tobi2904](https://github.com/Tobi2904), focused on Spring Boot / Java backend conventions. Rule #13 (Localized Error Messages) defaults to a Vietnamese example and is intended to be customized or removed for other audiences.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

---

> **The rules below (#5–#17) are original Spring Boot / Java backend additions, not part of the upstream Karpathy guidelines.**

## 5. No Hardcoding (Use Constants & Enums)

**Define constants. No magic numbers or strings.**

- Never hardcode configuration values, error keys, or business-logic literals directly into the code.
- Extract any potentially changeable value into a dedicated constant class, configuration file, or Enum.

The test: Will changing a business rule or a status value require editing multiple lines of code? If yes, use a Constant/Enum.

## 6. Thin Controllers, Fat Services

**Controllers handle HTTP traffic only. Business logic belongs to Services.**

- Controllers must only accept requests, validate inputs, invoke services, and return responses.
- Zero business logic, data transformations, or database queries are allowed inside Controller classes.

The test: If a Controller method contains an `if-else` business decision or data processing loop, move it to a Service.

## 7. SOLID & Clean Code Adherence

**Write modular, extensible, and maintainable code.**

- Strictly adhere to SOLID principles (especially Single Responsibility and Dependency Inversion).
- Proactively suggest clean code refactoring if you spot messy structural patterns in the working context.

The test: Is the new code highly coupled or difficult to unit test? If yes, refactor using proper design patterns.

## 8. Automated Mapping with MapStruct

**Never map data manually. Use MapStruct for DTOs and Entities.**

- All data transformations between Database Entities, Domain Objects, and DTOs must go through MapStruct mappers.
- Do not write manual getter/setter code for object-to-object mapping.

The test: Check for any manual `dto.setX(entity.getX())` blocks. Replace them with a MapStruct interface.

## 9. Boilerplate Reduction with Lombok

**Leverage Lombok annotations consistently. No manual boilerplate.**

- Use Lombok annotations (`@Getter`, `@Setter`, `@RequiredArgsConstructor`, `@Builder`, etc.) to generate boilerplate.
- Do not manually write getters, setters, or standard constructors unless explicitly required by framework constraints.

The test: Ensure classes remain clean and concise without explicit boilerplate methods cluttering the file.

## 10. Standardized Identifiers (UUID v7)

**Enforce UUID v7 for all entity primary keys.**

- Always use UUID version 7 for database identifiers to ensure time-ordered uniqueness and optimal indexing performance.

The test: Verify that any new entity or ID generation logic strictly initializes using a UUID v7 generator.

## 11. Proactive Security & Performance Audits

**Surface risks early. Report vulnerabilities and bottlenecks immediately.**

- While reading or writing code, if you detect a security loophole (e.g., SQL injection, broken access control) or a performance bottleneck (e.g., N+1 query issue, heavy loops), stop and report it immediately.
- Present the risk and proposed solutions to the user before proceeding with the task.

The test: Never ignore or bypass a potential architectural risk just to complete the prompt's functional requirement.

For N+1 specifically: flag any `findAll()` (or similar) followed by per-element repository/getter calls that trigger additional queries. This rule's job is to **catch** the N+1, not to prescribe the fetch strategy — for the fix (to-one vs collection), defer to Rule #16.

## 12. Custom Pagination Wrappers

**Never return default `Page<T>`. Wrap in `PageResponse<T>`.**

- Ensure all paginated API responses return a custom `PageResponse<T>` class instead of the framework's default `Page<T>` implementation.
- This maintains flexibility for future response structure customizations.

The test: Check the return type of paginated controller endpoints. It must be wrapped in `PageResponse<T>`.

## 13. Localized Error Messages

**All user-facing error messages must be in the end-user's language, not the developer's.**

> **Customize this rule for your audience.** The default below targets Vietnamese end-users (the original author's market). If your product ships to a different audience, replace "Vietnamese" with your target language. If your product is internal/developer-only and end-users read English fine, **you can safely delete this rule**.

- Do not return English error messages to the client application (assuming non-English end-users).
- Craft error messages in clear, natural, and helpful language so the end-user can easily understand and resolve the issue.
- Keep technical logs in English (for the dev team). The translation rule applies only to the **response body that reaches the client**.

The test: Read the Exception messages and error responses out loud. They must be in the language your end-user actually speaks.

Example (Vietnamese):
```java
// ❌ Bad — leaks English to a Vietnamese end-user
throw new NotFoundException("User not found");

// ✅ Good
throw new NotFoundException("Không tìm thấy người dùng tương ứng.");
```

## 14. User-Centric Roleplay & Edge Cases

**Think like a real user, not just a software developer.**

- Actively role-play as an end-user interacting with the software to uncover hidden edge cases, UX friction, and unexpected error states.
- Do not assume "happy paths." Explicitly safeguard against chaotic or improper user behaviors.

The test: List at least two potential user-error edge cases and how the code handles them before marking a feature complete.

## 15. Two-Tier Validation (Declarative + Strategy)

**Stateless validation on DTOs via annotations. Stateful/business validation as injectable Validator strategies — never a wall of private methods in the service.**

- **Tier 1 — Stateless** (null, blank, regex, size, range, enum, and static cross-field like `endDate` after `startDate`): declare with `jakarta.validation` annotations on request DTOs, triggered by `@Valid` / `@Validated` in the controller. Never re-check these in the service.
- **Tier 2 — Stateful / business** (uniqueness, balance, inventory, ownership, status transitions — anything needing a DB lookup or another entity): keep it out of the controller. NEVER inject a `Repository` into a `ConstraintValidator` — it hides business rules, runs outside `@Transactional`, and invites N+1 / session bugs.
- Model each business rule as a `@Component` implementing a shared `XxxValidator` interface. Inject `List<XxxValidator>` into the service and loop over it — Spring auto-collects every implementation, so adding a rule means adding a class (OCP), not editing the service.
- When order matters (e.g. existence before permission), control it with `@Order` / the `Ordered` interface on the validators; for strict pass-then-proceed gating, use a Chain of Responsibility.

The test: Does this check need data beyond the request (a DB row, another entity)? Yes → a Validator strategy. No → a DTO annotation. If a service is accumulating private `validate*()` methods, extract them into strategies.

## 16. High-Performance Data Fetching (Split Queries over `JOIN FETCH`)

**Ban `JOIN FETCH` on collections. Fetch the parent and its child collections separately, then assemble in memory.**

- **The Cartesian ban:** Never use `JOIN FETCH` (or entity-loading `JOIN`s) on `@OneToMany` / `@ManyToMany` relationships. It triggers a Cartesian-product blow-up (and `MultipleBagFetchException` once two `List` collections are fetched), exhausting heap and DB CPU.
- **Mandatory split queries:** When business logic needs a parent plus its child collections, fetch them with separate repository calls (e.g. `findBy...In(Collection<ID> ids)`) and stitch the relationships together in the service using `Stream` + a `HashMap` for O(1) lookups.
- **Single responsibility in repositories:** Keep JPQL dead simple — a repository method should query only its primary table.
- **Read-only projections:** For strictly read-only endpoints, DTO projections (`SELECT new ...`) are allowed, but they still must not join multiple collection tables.
- **To-one is exempt:** `@ManyToOne` / `@OneToOne` associations may still use `JOIN FETCH` / `@EntityGraph` — the ban applies to collections only (see Rule #11).

The test: If generated JPQL contains a `JOIN FETCH` for a collection, or the database is asked to join multiple collection tables into a single result set to populate entity relations, rewrite it immediately using the Split Queries (Java `HashMap` assembly) pattern.

## 17. Use `Set<>` (not `List<>`) for Entity Collections

**Entity association fields are `Set<>`, never `List<>`.**

- Map `@OneToMany` / `@ManyToMany` association fields as `Set<T>`, not `List<T>`. This prevents duplicate rows and models many-to-many relationships correctly. (Hibernate treats a `List` as a "bag", which also makes it the source of `MultipleBagFetchException`.)
- Initialize the field to an empty collection (`= new HashSet<>()`) to avoid `NullPointerException` on a freshly built entity.
- Because `Set` membership depends on `equals` / `hashCode`, do **not** slap Lombok `@EqualsAndHashCode` / `@Data` on entities — they pull in lazy/mutable fields and break the set. Base equality on a stable business key or the assigned UUID v7 id (Rule #10).
- `List<>` stays perfectly fine for DTOs, projections, and method return types — this rule is about **entity association fields** only.

The test: Does an `@Entity` declare a `List<>` association field? Change it to `Set<>` (and confirm equality is id/business-key based, not Lombok-generated).
