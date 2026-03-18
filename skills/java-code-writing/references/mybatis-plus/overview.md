# MyBatis-Plus Overview

Read this file when the task clearly touches MyBatis-Plus persistence code. Use it to choose the narrowest deeper reference instead of loading every persistence rule at once.

## When To Continue Deeper
- Read `entity-conventions.md` for `*DO` design, base entity selection, logical delete, optimistic lock, or `@TableName` questions.
- Read `dao-patterns.md` for `*Dao`, `*Mapper`, wrapper-based queries, mapper superinterfaces, or default-method questions.
- Read `paging-batch-update.md` for `Page<T>`, page-query boundaries, batch insert or update logic, bulk deletes, or partial update safety.

## Default Routing
- If the task changes fields or inheritance on a DO, start with `entity-conventions.md`.
- If the task changes query methods or DAO signatures, start with `dao-patterns.md`.
- If the task changes paging, batch work, or `updateById` patterns, start with `paging-batch-update.md`.

## Persistence Scope
- Keep MyBatis-Plus guidance focused on persistence structure and data access.
- Do not move service orchestration or controller transport concerns into DAO rules.

## Self-Check
- Did you read only the deeper file that matches the task?
- Are you keeping persistence guidance out of unrelated service or controller work?
- If the task also changes schema shape, did you read `../database-conventions.md` from the main skill root path?
