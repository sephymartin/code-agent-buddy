# Paging, Batch, And Update Safety

Use this file when the task focuses on page queries, batch writes, bulk deletes, or safe partial updates.

## Paging
Keep transport-layer paging objects decoupled from MyBatis-Plus types where possible.

Common flow:

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
- Keep `Page<T>` inside service and DAO boundaries.
- Expose project-level paging DTO and result objects upward.

## Batch Operations
- Prefer project-provided batch methods when available.
- Split large datasets into chunks to control SQL size and transaction pressure.

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

## Self-Check
- Are MyBatis-Plus paging types contained below controller boundaries?
- Does the batch logic chunk large collections?
- Did you avoid loading a whole row just to update one or two fields?
- If using a full-object update, is that dependency on loaded state explicit and necessary?
