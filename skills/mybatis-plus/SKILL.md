---
name: mybatis-plus
description: Use when writing or reviewing Java persistence code based on MyBatis-Plus, especially for entity modeling, DAO query methods, paging, batch writes, and safe partial updates
---

# MyBatis-Plus

## Overview
This skill captures practical MyBatis-Plus usage patterns for Java backend code. It focuses on entity design, DAO conventions, reusable default methods, paging, batch operations, and update safety.

Use this skill when the task is primarily about persistence-layer implementation or review.

Do not use this skill as the source of truth for project-specific package naming, architecture layering, or mandatory base classes that only exist in one repository.

## When to Use
- Creating or modifying `*DO` entity classes
- Creating or modifying `*Dao` or `*Mapper` interfaces
- Adding common query methods with MyBatis-Plus wrappers
- Implementing page queries with MyBatis-Plus `Page`
- Writing batch insert, batch update, or bulk delete logic
- Reviewing update logic for accidental full-row overwrites

## Entity Conventions

### Naming
- Prefer entity classes ending with `DO`
- Keep entity classes aligned with table structure rather than service-layer view models

### Base Class Selection
Choose the smallest base entity that matches the actual auditing requirement.

Typical categories:
- Time audit only: base entity with `id`, `createdTime`, `updatedTime`
- Operator audit required: auditable base entity with creator and updater fields
- Multi-tenant data: tenant-aware base entity
- Multi-tenant plus operator audit: tenant-aware auditable base entity

If the project does not define these exact base classes, apply the same idea rather than forcing the names.

### Optional Fields
- Logical delete field only when the table is actually soft-deleted
- Optimistic lock field only when concurrent update control is needed

Example:

```java
@Data
@TableName("sys_user")
public class SysUserDO extends AbstractAuditableEntity {
    private String username;
}
```

## DAO Conventions
- Prefer a project base mapper such as `CustomBaseMapper<T>` when available
- If no custom base mapper exists, use `BaseMapper<T>`
- Prefer DAO interfaces to remain thin and predictable
- Put repeated single-entity query logic into interface `default` methods when reuse is likely

## Default Method Patterns
Use interface `default` methods to wrap common, low-complexity query logic.

Recommended naming:
- `getBy...`: returns entity or `null`
- `loadBy...`: returns entity or throws when missing
- `existsBy...` or `is...Exists`: returns `boolean`
- `selectBy...`: returns list

Example:

```java
public interface SysUserDao extends CustomBaseMapper<SysUserDO> {
    default SysUserDO getByUsername(String username) {
        return selectOne(Wrappers.lambdaQuery(SysUserDO.class)
            .eq(SysUserDO::getUsername, username));
    }
}
```

Use default methods for:
- Single-entity lookup
- Existence check
- Small reusable query fragments

Do not hide complex business workflows in DAO default methods.

## Paging
Keep transport-layer paging objects decoupled from MyBatis-Plus types where possible.

A common flow is:

```text
Controller: request paging DTO
Service: convert to MyBatis-Plus Page<T>
Dao: execute page query
Service: convert page result to response object
```

Example:

```java
public PagingResult<SysUserVO> querySysUserPage(SysUserQueryDTO queryDTO) {
    Page<SysUserDO> result = sysUserDao.selectUserPage(queryDTO.toMpPage(), queryDTO);
    return PagingResult.from(result, this::convertToVO);
}
```

Guideline:
- Keep `Page<T>` inside service and DAO boundaries
- Expose project-level paging DTO and result objects upward

## Batch Operations
Prefer project-provided batch methods when available.

Typical operations:
- Batch insert ignoring generated primary key columns
- Batch insert including all columns
- Batch update by primary key
- Batch delete by primary key set

For large datasets, split into chunks to control SQL size and transaction pressure.

Example:

```java
int batchSize = 5000;
for (int i = 0; i < list.size(); i += batchSize) {
    dao.insertBatchSomeColumn(list.subList(i, Math.min(i + batchSize, list.size())));
}
```

## Update Safety
Prefer partial update objects over read-modify-write when only a few columns need to change.

Avoid:

```java
SaobeiPaymentOrderDO order = dao.selectById(id);
order.setBatchId(batchId);
dao.updateById(order);
```

Prefer:

```java
SaobeiPaymentOrderDO update = new SaobeiPaymentOrderDO();
update.setId(id);
update.setBatchId(batchId);
dao.updateById(update);
```

Reason:
- Reduces accidental overwrites
- Avoids persisting stale in-memory data
- Makes intent explicit

Use full-object update only when the business flow genuinely depends on the loaded state.

## Common Mistakes
- Loading a whole row just to update one or two fields
- Putting business orchestration into mapper XML or DAO default methods
- Returning MyBatis-Plus paging primitives directly to controller layers without a boundary type
- Running oversized batch SQL without chunking
- Adding logical delete and optimistic lock fields by default without actual need

## Project-Specific Notes
If a repository has mandatory entity base classes, mapper superinterfaces, or utility types like `PagingQuery` and `PagingResult`, keep those exact names in repository rules or project docs. This skill should teach the pattern, not hardcode one repository's class catalog.
