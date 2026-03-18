# Database Conventions

Use this file when the task is to design or review database models for repository-aligned Java backend code.

## Baseline
- Default database target: `MySQL 8 + InnoDB + utf8mb4`
- Start from business responsibility and aggregate boundaries, then design fields and indexes.
- Schema decisions must remain compatible with repository DO and MyBatis-Plus conventions.

## Table And Primary Key Rules
- Each business table defaults to a single-column primary key named `id`.
- `id` defaults to `bigint NOT NULL AUTO_INCREMENT`.
- Java mapping defaults to `Long`.
- Do not use composite primary keys.
- Do not embed business meaning in `id`.
- Keep `id` type choices consistent across new tables.
- Relationship columns use `xxx_id` and must match the referenced `id` type.
- Relation tables still default to an independent `id` primary key plus business uniqueness constraints.

## Field Naming Rules
- Prefer clear business-semantic names over abbreviations.
- Use snake_case column names.
- Avoid SQL or MySQL keywords and high-conflict names such as `type`, `order`, `level`, `desc`, `key`, and `template`.
- If a keyword-like concept is required, expand it into a safer business name such as `status_code`, `sort_order`, or `template_content`.
- Boolean fields must not use an `is_` prefix.
- Prefer semantic boolean names like `enabled`, `deleted`, `visible`, `disabled`, and `sys_build_in`.
- Time fields use `*_time`.
- Business numbers use `*_no`.
- Business codes use `*_code`.
- Parent-child relationships use `parent_id`.

## Shared Fields
Default audit fields for most business tables:
- `created_by`
- `created_time`
- `updated_by`
- `updated_time`

Soft-delete tables also use:
- `deleted`

Common optional fields:
- `remark`
- `description`
- `sort`
- `status_code`

Guidance:
- Prefer `NOT NULL` when the business meaning is mandatory and default values are stable.
- Allow `NULL` only when unknown, not applicable, or deferred fill is meaningful.
- Comments are required for tables and fields.

## Type Selection
### Identifiers And Relationship Columns
- Primary keys and internal references default to `bigint`.
- Keep referenced and referencing columns aligned.

### String Fields
- Prefer `varchar` when the upper bound is known or can be constrained.
- Keep typical business labels, names, and codes bounded instead of using `text`.

### Boolean-Like Fields
- Use the repository-consistent small numeric or boolean-compatible type required by the surrounding table style.
- Name the field semantically, not with `is_`.

### Status And Code Fields
- Use bounded string columns such as `varchar(20)` or `varchar(32)` when values are code-like and human-readable.
- Prefer `*_code` suffixes for business-state fields instead of ambiguous raw names.

### JSON Fields
- Use JSON only when the structure is truly nested, variable, or operationally expensive to normalize.
- Do not hide clear relational structure inside JSON.
- JSON fields require explicit DO mapping with a type handler such as `JacksonTypeHandler`.

### Long Text
- `text` and `longtext` are not default choices for the main table.
- Default to `varchar` unless the content size is clearly unbounded or very large.
- If a field may exceed bounded string limits or is not needed in high-frequency queries, evaluate split-table storage first.

## Long-Text And Split-Table Rules
- `text` and `longtext` fields should normally live in an extension table or dedicated long-text table.

Prefer split-table storage when:
- The field can clearly exceed `varchar(2000)` scale.
- The field is only needed for detail pages.
- The list or page query should not read the field.
- The entity would otherwise carry multiple large text fields.
- The content is a template body, rich text, large JSON blob, audit snapshot, or message body.

Allow large text on the main table only when it is core to the entity, almost always read with it, data volume is controlled, and split-table complexity is not justified.

## Index And Constraint Rules
- Every table has a primary key on `id`.
- Unique constraints must reflect a real business invariant.
- For soft-delete tables, business unique indexes often include `deleted`.
- Do not add indexes without a query, lookup, deduplication, or ordering reason.
- Prefer concise indexes aligned with actual predicates.
- Relation tables should usually block duplicate relationships with a unique index.

## Relationship Rules
- Internal table relationships default to `xxx_id`.
- Do not use `xxx_no` or `xxx_code` as the default foreign-key substitute for internal relations.
- Parent-child trees use `parent_id`.
- Keep relationship columns typed exactly like the referenced table key.
- Normalize true one-to-many and many-to-many structures instead of hiding them in CSV strings or JSON arrays when relational queries are expected.

## Redundant Business Identifier Rules
- `xxx_no` and `xxx_code` may be redundantly stored in related tables.
- Redundancy is recommended for key business chains such as orders, payments, refunds, notifications, and reconciliation records.
- Redundant identifiers are mirrors, not the default source of truth for referential structure.
- If redundancy exists, define where the canonical value comes from and treat the redundant value as read-only by default.
- Add indexes to redundant identifiers only when a real lookup scenario exists.

## DO And MyBatis-Plus Alignment
- Entity classes should end with `DO`.
- Use `@TableName` on each DO.
- New tables should normally fit the repository audit base entity unless there is a justified alternative.
- Soft-delete tables should use a `deleted` field only when compatible with repository logical-delete conventions.
- JSON columns need explicit `@TableField(typeHandler = ...)` handling.
- Column names should map cleanly to Java camelCase fields without awkward exceptions.

## Self-Check
- Did you keep a single-column `id` primary key unless there is an explicit exception?
- Are relationship columns on `xxx_id`, with `xxx_no` or `xxx_code` used only as mirrors?
- Did you avoid boolean names with the `is_` prefix and use `*_time` for time columns?
- Did you evaluate split-table storage before putting large text on the main table?
- Will the schema map cleanly to a `*DO` class and repository base entity conventions?
