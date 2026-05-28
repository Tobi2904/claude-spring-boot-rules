# CLAUDE.md

Behavioral guidelines to reduce common LLM coding mistakes for Spring Boot / Java backend projects. Merge with project-specific instructions as needed.

> **Attribution:** Rules #1–#4 are derived from [`forrestchang/andrej-karpathy-skills`](https://github.com/forrestchang/andrej-karpathy-skills) (MIT License, © Forrest Chang), based on Andrej Karpathy's observations on LLM coding pitfalls.
> Rules #5–#14 are original additions by [@Tobi2904](https://github.com/Tobi2904), focused on Spring Boot / Java backend conventions. Rule #13 (Localized Error Messages) defaults to a Vietnamese example and is intended to be customized or removed for other audiences.

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

> **The rules below (#5–#14) are original Spring Boot / Java backend additions, not part of the upstream Karpathy guidelines.**

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
