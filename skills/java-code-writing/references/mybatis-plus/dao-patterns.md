# DAO Patterns

Use this file for `*Dao` or `*Mapper` design and review.

## Mapper Superinterface
- Prefer a project base mapper such as `CustomBaseMapper<T>` when available.
- If no custom base mapper exists, use `BaseMapper<T>`.

## DAO Shape
- Keep DAO interfaces thin and predictable.
- Put repeated single-entity query logic into interface `default` methods when reuse is likely.
- Do not hide complex business workflows in DAO default methods or mapper XML.

## Default Method Patterns
Use interface `default` methods for low-complexity query logic.

Recommended naming:
- `getBy...`: returns an entity or `null`
- `loadBy...`: returns an entity or throws when missing
- `existsBy...` or `is...Exists`: returns `boolean`
- `selectBy...`: returns a list

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
- Existence checks
- Small reusable query fragments

## Boundary Rules
- DAO code should encapsulate data access, not service orchestration.
- Keep transport DTO conversion outside DAO interfaces.

## Self-Check
- Is the DAO still thin after the change?
- Are default methods limited to low-complexity query helpers?
- Did you avoid moving business orchestration into DAO code?
- Are method names aligned with the actual return contract?
