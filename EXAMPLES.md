# EXAMPLES.md

Concrete before/after examples for each of the 15 rules in [`CLAUDE.md`](./CLAUDE.md). Real Spring Boot / Java code.

---

## Rule 1 — Think Before Coding

**Prompt:** "Add a search endpoint for users."

❌ **Bad — silently picks an interpretation:**
```java
@GetMapping("/users/search")
public List<User> search(@RequestParam String q) {
    return repo.findByNameContaining(q);
}
```

✅ **Good — surfaces ambiguity:**
> Before I implement: "search" can mean (a) name only, (b) name + email, (c) full-text including phone. Also — should this be paginated, and should it require auth? My default: name + email, paginated, requires auth. Confirm or correct?

---

## Rule 2 — Simplicity First

❌ **Bad — speculative configurability:**
```java
public class UserValidator {
    private final ValidationStrategy strategy;
    private final List<ValidationRule> rules;
    private final boolean strictMode;
    // 80 more lines of "flexibility" no one asked for
}
```

✅ **Good — what was actually needed:**
```java
public void validate(UserCreateRequest req) {
    if (req.email() == null) throw new BadRequestException("Email không được trống");
    if (req.password().length() < 8) throw new BadRequestException("Mật khẩu phải có ít nhất 8 ký tự");
}
```

---

## Rule 3 — Surgical Changes

**Prompt:** "Add a `phoneNumber` field to the User entity."

❌ **Bad — touches unrelated code:**
```diff
  public class User {
+     // Reformatted all existing fields, renamed `userName` to `username`,
+     // extracted constants for max lengths, "improved" the toString...
+     private String phoneNumber;
  }
```

✅ **Good — minimal diff:**
```diff
  public class User {
      private String email;
      private String username;
+     private String phoneNumber;
  }
```

---

## Rule 4 — Goal-Driven Execution

**Prompt:** "Fix the login bug where wrong passwords sometimes succeed."

❌ **Bad — guesses at fixes:**
> "I rewrote the `AuthService` — should be working now."

✅ **Good — define success first:**
> Plan:
> 1. Write a failing test reproducing the bug (wrong password → expect `401`) → verify: test fails on current code
> 2. Trace the issue → verify: I can explain the root cause in one sentence
> 3. Apply fix → verify: the test from step 1 passes, existing tests still pass

---

## Rule 5 — No Hardcoding

❌ **Bad:**
```java
if (user.getStatus().equals("ACTIVE")) {
    if (user.getLoginAttempts() > 5) {
        throw new RuntimeException("Tài khoản bị khóa");
    }
}
```

✅ **Good:**
```java
public enum UserStatus { ACTIVE, INACTIVE, LOCKED }

public final class AuthConstants {
    public static final int MAX_LOGIN_ATTEMPTS = 5;
    private AuthConstants() {}
}

public final class ErrorMessages {
    public static final String ACCOUNT_LOCKED = "Tài khoản đã bị khóa do nhập sai mật khẩu quá nhiều lần";
    private ErrorMessages() {}
}

if (user.getStatus() == UserStatus.ACTIVE
        && user.getLoginAttempts() > AuthConstants.MAX_LOGIN_ATTEMPTS) {
    throw new AccountLockedException(ErrorMessages.ACCOUNT_LOCKED);
}
```

---

## Rule 6 — Thin Controllers, Fat Services

❌ **Bad — business logic in controller:**
```java
@PostMapping("/orders")
public OrderResponse create(@RequestBody OrderRequest req) {
    var user = userRepo.findById(req.userId()).orElseThrow();
    if (user.getBalance().compareTo(req.amount()) < 0) {
        throw new BadRequestException("Không đủ số dư");
    }
    var order = new Order();
    order.setUser(user);
    order.setAmount(req.amount());
    order.setStatus("PENDING");
    user.setBalance(user.getBalance().subtract(req.amount()));
    orderRepo.save(order);
    userRepo.save(user);
    emailService.sendOrderConfirmation(user, order);
    return new OrderResponse(order.getId(), "PENDING");
}
```

✅ **Good — controller is a pipe:**
```java
@PostMapping("/orders")
public OrderResponse create(@RequestBody @Valid OrderRequest req) {
    return orderService.createOrder(req);
}
```
All logic — balance check, debit, save, email — lives in `OrderService.createOrder()`.

---

## Rule 7 — SOLID & Clean Code

❌ **Bad — one class does everything:**
```java
public class UserService {
    public User create(...) { /* validate + save + email + audit log */ }
}
```

✅ **Good — Single Responsibility:**
```java
@RequiredArgsConstructor
public class UserService {
    private final UserValidator validator;
    private final UserRepository repository;
    private final UserMapper mapper;
    private final EmailNotifier notifier;
    private final AuditLogger audit;

    public UserResponse create(UserCreateRequest req) {
        validator.validate(req);
        var saved = repository.save(mapper.toEntity(req));
        notifier.welcome(saved);
        audit.logCreation(saved);
        return mapper.toResponse(saved);
    }
}
```

---

## Rule 8 — MapStruct for Mapping

❌ **Bad — manual mapping:**
```java
UserResponse resp = new UserResponse();
resp.setId(user.getId());
resp.setEmail(user.getEmail());
resp.setFullName(user.getFirstName() + " " + user.getLastName());
resp.setCreatedAt(user.getCreatedAt());
// ... 20 more setters
```

✅ **Good — MapStruct interface:**
```java
@Mapper(componentModel = "spring")
public interface UserMapper {
    @Mapping(target = "fullName", expression = "java(user.getFirstName() + \" \" + user.getLastName())")
    UserResponse toResponse(User user);

    User toEntity(UserCreateRequest req);
}
```

---

## Rule 9 — Lombok for Boilerplate

❌ **Bad:**
```java
public class User {
    private UUID id;
    private String email;
    // 50 lines of getters, setters, hashCode, equals, toString, constructors
}
```

✅ **Good:**
```java
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
@ToString
public class User {
    private UUID id;
    private String email;
}
```

---

## Rule 10 — UUID v7 for IDs

❌ **Bad — random UUID v4 fragments index performance:**
```java
@Id
@GeneratedValue(strategy = GenerationType.AUTO)
private UUID id;
```

✅ **Good — UUID v7 (time-ordered):**
```java
@Id
private UUID id;

@PrePersist
public void prePersist() {
    if (this.id == null) {
        this.id = UuidCreator.getTimeOrderedEpoch(); // com.github.f4b6a3:uuid-creator
    }
}
```

---

## Rule 11 — Proactive Security & Performance Audits

**Prompt:** "Add a search by username."

❌ **Bad — ships with SQLi + N+1:**
```java
@Query(value = "SELECT * FROM users WHERE username LIKE '%" + username + "%'", nativeQuery = true)
List<User> findByUsername(String username);

// Then in service:
users.forEach(u -> u.getOrders().size()); // N+1!
```

✅ **Good — stop and flag:**
> I notice two risks before continuing:
> 1. **SQL Injection** — concatenating `username` into a native query. I'll use a parameterized JPQL query.
> 2. **Potential N+1** — if you'll access `orders` afterward, I'll add `@EntityGraph(attributePaths = "orders")` or a `JOIN FETCH`.
>
> Confirm before I proceed?

---

## Rule 12 — Custom `PageResponse<T>`

❌ **Bad — leaks Spring's internal pagination shape:**
```java
@GetMapping("/users")
public Page<UserResponse> list(Pageable pageable) {
    return userService.list(pageable);
}
```

✅ **Good:**
```java
@Getter
@Builder
public class PageResponse<T> {
    private List<T> content;
    private int page;
    private int size;
    private long totalElements;
    private int totalPages;
    private boolean hasNext;
}

@GetMapping("/users")
public PageResponse<UserResponse> list(Pageable pageable) {
    return userService.list(pageable);
}
```

---

## Rule 13 — Localized Error Messages

> Default ships with Vietnamese examples (the original author's market). Substitute your audience's language, or skip this rule entirely if your end-users read English natively.

❌ **Bad** — shipping English to non-English end-users:
```java
throw new BadRequestException("Email already exists");
throw new NotFoundException("User not found");
throw new ForbiddenException("Insufficient permissions");
```

✅ **Good** (Vietnamese example):
```java
throw new BadRequestException("Email này đã được sử dụng. Vui lòng dùng email khác.");
throw new NotFoundException("Không tìm thấy người dùng tương ứng.");
throw new ForbiddenException("Bạn không có quyền thực hiện thao tác này.");
```

✅ **Good** (Spanish example, for reference):
```java
throw new BadRequestException("Este correo ya está en uso. Por favor, utiliza otro.");
throw new NotFoundException("No se encontró el usuario.");
throw new ForbiddenException("No tienes permiso para realizar esta acción.");
```

**Important:** Keep stacktraces, log messages, and internal exception classes in English — that's for the dev team. The rule applies only to what the **client receives in the response body**.

---

## Rule 14 — User-Centric Roleplay & Edge Cases

**Prompt:** "Implement file upload for user avatars."

❌ **Bad — happy path only:**
```java
@PostMapping("/avatar")
public void upload(@RequestParam MultipartFile file) {
    Files.write(Path.of("/uploads/" + file.getOriginalFilename()), file.getBytes());
}
```

✅ **Good — roleplay first:**
> Edge cases I'll guard against:
> 1. **User uploads a 5GB file** → enforce `spring.servlet.multipart.max-file-size=5MB`, return `413` with Vietnamese message
> 2. **User uploads `../../etc/passwd` as filename** → sanitize filename, use UUID-based storage names
> 3. **User uploads a `.exe` renamed to `.jpg`** → validate magic bytes, not just extension
> 4. **User uploads simultaneously from 3 tabs** → idempotent storage, only the last write wins
> 5. **User has no avatar yet (cold path)** → endpoint returns `null`, not `404`
>
> Implementing with these guards in place:
```java
// ... safe implementation
```

---

## Rule 15 — Two-Tier Validation (Declarative + Strategy)

**Tier 1 — stateless checks live on the DTO** (null, blank, regex, size, range, enum):

❌ **Bad — format checks bloating the service:**
```java
public OrderResponse create(OrderRequest req) {
    if (req.userId() == null) throw new BadRequestException("Vui lòng chọn người dùng");
    if (req.amount() == null || req.amount().signum() <= 0)
        throw new BadRequestException("Số tiền phải lớn hơn 0");
    // ... business logic only starts here
}
```

✅ **Good — annotations on the request DTO, triggered in the controller:**
```java
public record OrderRequest(
    @NotNull(message = "Vui lòng chọn người dùng")
    UUID userId,

    @NotNull @Positive(message = "Số tiền phải lớn hơn 0")
    BigDecimal amount
) {}

@PostMapping("/orders")
public OrderResponse create(@RequestBody @Valid OrderRequest req) {
    return orderService.create(req); // input is already well-formed
}
```

**Tier 2 — stateful/business checks are `Validator` strategies, not service bloat.** Anything needing the DB or another entity (uniqueness, balance, status transition) stays server-side — but extracted, so the service never becomes a God Class.

❌ **Bad — every rule piled into one service method:**
```java
@Transactional
public OrderResponse create(OrderRequest req) {
    var user = userRepo.findById(req.userId()).orElseThrow(...);
    if (user.getStatus() == UserStatus.LOCKED) throw new ...("Tài khoản đã bị khóa.");
    if (user.getBalance().compareTo(req.amount()) < 0) throw new ...("Số dư không đủ.");
    // + 200 more lines as rules accumulate → unmaintainable, merge-conflict magnet
}
```

✅ **Good — one `@Component` per rule, the service just orchestrates:**
```java
public interface OrderValidator {
    void validate(User user, OrderRequest req);
}

@Component @Order(1)
class AccountActiveValidator implements OrderValidator {
    public void validate(User user, OrderRequest req) {
        if (user.getStatus() == UserStatus.LOCKED)
            throw new AccountLockedException("Tài khoản đã bị khóa.");
    }
}

@Component @Order(2)
class SufficientBalanceValidator implements OrderValidator {
    public void validate(User user, OrderRequest req) {
        if (user.getBalance().compareTo(req.amount()) < 0)
            throw new InsufficientBalanceException("Số dư không đủ để thực hiện giao dịch.");
    }
}

@Service
@RequiredArgsConstructor
public class OrderService {
    private final List<OrderValidator> validators; // Spring injects all @Components, ordered by @Order
    private final UserRepository userRepo;

    @Transactional
    public OrderResponse create(OrderRequest req) {
        var user = userRepo.findById(req.userId())
            .orElseThrow(() -> new NotFoundException("Không tìm thấy người dùng."));
        validators.forEach(v -> v.validate(user, req)); // runs active-check → balance-check, in @Order
        // ... core business logic
    }
}
```

Adding a fraud check tomorrow = a new `@Component`, **zero edits** to `OrderService` (Open/Closed). Each validator is unit-testable in isolation. When sequence matters, `@Order` (or implementing `Ordered`) sets run order — lower value runs first; for strict pass-step-1-before-step-2 gating, use a Chain of Responsibility instead.

---

These examples are illustrative, not exhaustive. The point: **every rule has a concrete failure mode it prevents.** When in doubt, re-read the failure mode.
