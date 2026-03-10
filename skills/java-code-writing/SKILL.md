---
name: java-code-writing
description: Use when writing or reviewing ordinary Java application code, especially for code style, Lombok usage, dependency injection, transaction boundaries, utility classes, and clear synchronous service-layer design
---

# Java Code Writing

## Overview
This skill captures general Java application coding conventions that are broadly reusable across Spring-based backend projects. It focuses on code shape and maintainability rather than repository-specific package structure.

Use this skill for ordinary Java coding tasks outside the persistence-specific patterns covered by the MyBatis-Plus skill.

Do not use this skill as the source of truth for project-specific layer names, mandatory class suffixes, or package paths.

## When to Use
- Writing new Java classes in business applications
- Reviewing code style and maintainability
- Adding Spring services or controllers
- Choosing Lombok annotations
- Implementing transaction boundaries
- Writing utility classes
- Reviewing whether async or reactive types are leaking into synchronous service layers

## File Header
- Do not generate license or copyright boilerplate unless the repository explicitly requires it
- Start directly from `package` or from a class comment if one is needed

## Javadoc
- Use blank lines to separate paragraphs
- Avoid HTML tags like `<br>` and `<p>` in ordinary Javadoc
- Write comments only where intent is not obvious from names and types

## Enum Formatting
- Put each enum constant on its own line
- Keep a trailing comma after each constant except where project formatter forbids it
- Add concise `//` comments only when the constant meaning is not obvious
- Put the terminating semicolon on its own line when the enum defines fields or methods

## Lombok
Use Lombok to reduce boilerplate, but keep annotation combinations intentional.

Typical defaults:
- Entities: `@Data` if mutable entity objects are the established pattern
- DTOs: `@Data`, plus constructors when transport usage needs them
- Service implementations: `@Slf4j` and `@RequiredArgsConstructor`
- Controllers: `@RequiredArgsConstructor`

### Builder Usage
- Use `@Builder` only when the call site clearly benefits
- Add `@Jacksonized` only when builder-based JSON deserialization is required
- Avoid redundant combinations like `@Builder` with both no-args and all-args constructors unless a framework truly needs them

### Null Safety
- Use `@NonNull` for parameter validation only when the codebase already uses Lombok null contracts consistently

## Dependency Injection
- Prefer constructor injection
- In Lombok-based Spring projects, prefer `@RequiredArgsConstructor` with `final` fields
- Avoid field injection such as `@Autowired` or `@Resource` on fields

Example:

```java
@Service
@RequiredArgsConstructor
public class OrderServiceImpl implements OrderService {

    private final OrderDao orderDao;
}
```

## Transactions
- Put transaction boundaries on service methods, not DAO methods
- When checked exceptions should also trigger rollback, use `@Transactional(rollbackFor = Exception.class)`
- Do not rely on bare `@Transactional` if the business rule requires rollback beyond runtime exceptions

Example:

```java
@Transactional(rollbackFor = Exception.class)
public void createOrder(CreateOrderCmd cmd) {
    // transactional work
}
```

## Utility Classes
For utility-only classes:
- Do not mark them as Spring components
- Make methods `public static`
- Make the class `final` or `abstract`
- Add a private constructor to prevent instantiation

Example:

```java
public final class DateFormatUtils {

    private DateFormatUtils() {
    }

    public static String format(LocalDate date) {
        return date.toString();
    }
}
```

## Synchronous Service Boundaries
In ordinary business applications with synchronous service layers:
- Keep reactive or async primitives contained at integration edges
- Convert low-level reactive calls into ordinary return values before they reach service or controller layers
- Do not leak `Mono`, `Flux`, or `CompletableFuture` upward unless the whole application is intentionally designed around async flows

Example:

```java
public PaymentOrderResponse queryOrder(PaymentOrderRequest request) {
    return webClient.post().uri(url).bodyValue(request).retrieve()
        .bodyToMono(PaymentOrderResponse.class)
        .block(Duration.ofSeconds(30));
}
```

This is appropriate only when the application model is intentionally synchronous.

## Common Mistakes
- Field injection instead of constructor injection
- Blanket use of `@Builder` on every DTO or entity
- Utility classes turned into Spring beans
- Transaction annotation placed too low in the stack
- Commenting obvious code while leaving important invariants undocumented
- Mixing synchronous service design with partially leaked reactive return types

## Project-Specific Notes
Exact layer naming, package layout, command/query conventions, and class suffix rules vary by repository. Keep those as repository rules instead of hardcoding them into a general Java writing skill.
