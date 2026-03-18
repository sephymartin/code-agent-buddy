---
name: java-code-writing
description: Use when writing or reviewing Java backend code, especially when tasks involve Spring services or controllers, MyBatis-Plus DO or DAO patterns, paging or batch persistence logic, or repository-aligned schema design.
---

# Java Code Writing

## Overview
Use this skill for ordinary Java and Spring backend work. Keep the main file focused on common coding rules, and load deeper references only when the task clearly moves into MyBatis-Plus persistence or schema design.

Do not use this skill as the source of truth for repository-specific package names or layer names unless a referenced file says otherwise.

## When to Use
- Writing or reviewing Java backend classes
- Adding services, controllers, DTOs, enums, or utility classes
- Choosing Lombok annotations or dependency injection style
- Setting transaction boundaries
- Reviewing synchronous service-layer design
- Working on MyBatis-Plus DO, DAO, paging, batch, or partial update code
- Designing tables, fields, indexes, or DO mappings that should follow repository conventions

## Progressive Loading
- Stay in this file for DTOs, enums, services, controllers, utility classes, and ordinary Java code review.
- Read `references/mybatis-plus/overview.md` when the task mentions `*DO`, `*Dao`, `*Mapper`, `BaseMapper`, wrappers, paging, batch operations, or partial updates.
- From `references/mybatis-plus/overview.md`, continue into the narrower MyBatis-Plus reference that matches the task.
- Read `references/database-conventions.md` when the task involves table design, field naming, index design, long-text placement, JSON columns, redundant business identifiers, or DO mapping constraints.

## File Header
- Do not generate license or copyright boilerplate unless the repository explicitly requires it.
- Start directly from `package` or from a short class comment when one is needed.

## Javadoc
- Use blank lines to separate paragraphs.
- Avoid HTML tags like `<br>` and `<p>` in ordinary Javadoc.
- Write comments only where names and types do not already make intent clear.

## Comments
- Add short comments for non-obvious business intent, invariants, or multi-step logic where the code alone is not enough.
- Do not add comments that only restate the next line of code.
- When a block exists to prevent a historical bug, enforce an invariant, or encode a business exception, say that explicitly in the comment.

### Test Scenario Comments
- Test code must include a scenario comment above the test method.
- Prefer a short block comment or Javadoc-style comment.
- The comment must explicitly state the test input and the expected output.
- This is required even when the test method name is already descriptive.
- For parameterized tests or multi-branch tests, make each scenario comment distinguish the case under test.
- When the scenario depends on time, amounts, status changes, or a historical bug boundary, write those facts explicitly in the comment.

Example:

```java
/**
 * Test scenario: current date is 2026-03-13 and the overdue start date is 2026-03-12.
 * Input: overdue start date = 2026-03-12, current date = 2026-03-13, daily amount = 10.00.
 * Expected output: overdue days = 1 and overdue fee = 10.00.
 */
@Test
void shouldCalculateOneDayOverdueFee() {
    ...
}
```

## Enum Formatting
- Put each enum constant on its own line.
- Keep a trailing comma after each constant except where the formatter forbids it.
- Add concise `//` comments only when the constant meaning is not obvious.
- Put the terminating semicolon on its own line when the enum defines fields or methods.

## Lombok
Use Lombok intentionally rather than by default.

Typical defaults:
- DTOs: `@Data`, plus constructors only when transport or serialization usage needs them
- Service implementations: `@Slf4j` and `@RequiredArgsConstructor`
- Controllers: `@RequiredArgsConstructor`

### Builder Usage
- Use `@Builder` only when call sites become meaningfully clearer.
- Add `@Jacksonized` only when builder-based JSON deserialization is required.
- Avoid redundant combinations like `@Builder` with both no-args and all-args constructors unless a framework truly needs them.

### Null Safety
- Use `@NonNull` for parameter validation only when the codebase already uses Lombok null contracts consistently.

## Dependency Injection
- Prefer constructor injection.
- In Lombok-based Spring projects, prefer `@RequiredArgsConstructor` with `final` fields.
- Avoid field injection such as `@Autowired` or `@Resource` on fields.

Example:

```java
@Service
@RequiredArgsConstructor
public class OrderServiceImpl implements OrderService {

    private final OrderDao orderDao;
}
```

## Transactions
- Put transaction boundaries on service methods, not DAO methods.
- When checked exceptions should also trigger rollback, use `@Transactional(rollbackFor = Exception.class)`.
- Do not rely on bare `@Transactional` if the business rule requires rollback beyond runtime exceptions.

Example:

```java
@Transactional(rollbackFor = Exception.class)
public void createOrder(CreateOrderCmd cmd) {
    // transactional work
}
```

## Utility Classes
For utility-only classes:
- Do not mark them as Spring components.
- Make methods `public static`.
- Make the class `final` or `abstract`.
- Add a private constructor to prevent instantiation.

## Synchronous Service Boundaries
In ordinary business applications with synchronous service layers:
- Keep reactive or async primitives contained at integration edges.
- Convert low-level reactive calls into ordinary return values before they reach service or controller layers.
- Do not leak `Mono`, `Flux`, or `CompletableFuture` upward unless the whole application is intentionally designed around async flows.

## Logging
- Add logs at key decision points that materially affect the execution path or business result.
- Typical log-worthy points include state transitions, idempotent skips, retries or degradation, external-call failures, rule-based rejections, and data-drop branches.
- Include stable business identifiers, the decision taken, and the reason that led to it.
- Do not log secrets, tokens, passwords, or unnecessary personal data.
- Avoid noisy logs inside hot loops or on obvious branches where the surrounding context already makes the path clear.

### Log Level Guidance
- Use `info` for important state changes and business decisions that should be traceable in normal operation.
- Use `warn` for abnormal but recoverable situations that deserve attention.
- Use `error` for failures that block the current operation or require active troubleshooting.
- Do not use `error` for expected validation outcomes or ordinary business rejections.

## Gotchas
- Do not assume a table has a `deleted` field unless the schema or repository conventions actually require soft delete.
- Prefer partial update objects over loading a full row just to update one or two columns.
- Keep `Page<T>` and similar persistence paging types inside service and DAO boundaries instead of returning them from controllers.
- When schema naming becomes relevant, avoid boolean fields with the `is_` prefix and prefer `*_time` for time columns.

## Common Mistakes
- Field injection instead of constructor injection
- Blanket use of `@Builder` on every DTO or entity
- Utility classes turned into Spring beans
- Transaction annotations placed too low in the stack
- Commenting obvious code while leaving important invariants undocumented
- Writing tests without a scenario comment block that states the input and expected output
- Mixing synchronous service design with partially leaked reactive return types
- Adding noisy branch-by-branch logs without stable identifiers or decision reasons
- Pulling MyBatis-Plus or schema rules into ordinary Java tasks when the code does not touch persistence
