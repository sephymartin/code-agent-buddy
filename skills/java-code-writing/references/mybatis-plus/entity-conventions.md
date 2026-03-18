# Entity Conventions

Use this file for `*DO` design and review.

## Naming
- Prefer entity classes ending with `DO`.
- Keep entity classes aligned with table structure rather than service-layer view models.
- Use `@TableName` on each DO.

Example:

```java
@Data
@TableName("sys_user")
public class SysUserDO extends AbstractAuditableEntity {

    private String username;
}
```

## Base Class Selection
Choose the smallest base entity that matches the actual auditing requirement.

Typical categories:
- Time audit only: base entity with `id`, `createdTime`, and `updatedTime`
- Operator audit required: auditable base entity with creator and updater fields
- Multi-tenant data: tenant-aware base entity
- Multi-tenant plus operator audit: tenant-aware auditable base entity

If the project does not define these exact base classes, apply the same idea rather than forcing the names.

## Optional Persistence Fields
- Add a logical delete field only when the table is actually soft-deleted.
- Add an optimistic lock field only when concurrent update control is required.
- Keep DO fields mapped cleanly from column names to camelCase Java names.

## Boundary Rules
- DOs model persistence shape, not controller or service response shape.
- Avoid turning DOs into transport objects with presentation-only fields.

## Self-Check
- Does the class end with `DO` and map cleanly to the table?
- Is the chosen base entity the smallest one that fits the audit requirement?
- Did you avoid adding `deleted` or optimistic lock fields by default?
- Are persistence fields aligned with schema names rather than view-model needs?
