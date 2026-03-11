# Sephy-App Database Modeling Conventions

Use this reference when the task is to design or review database models for `sephy-app`.

## 1. Baseline

- Default database target: `MySQL 8 + InnoDB + utf8mb4`
- Start from business responsibility and aggregate boundaries, then design fields and indexes
- Schema decisions must remain compatible with repository DO and MyBatis-Plus conventions

## 2. Table And Primary Key Rules

- Each business table defaults to a single-column primary key named `id`
- `id` defaults to `bigint NOT NULL AUTO_INCREMENT`
- Java mapping defaults to `Long`
- Do not use composite primary keys
- Do not embed business meaning in `id`
- Keep `id` type choices consistent across new tables; avoid creating new mixed conventions
- Relationship columns use `xxx_id` and must match the referenced `id` type
- Relation tables still default to an independent `id` primary key plus business uniqueness constraints

## 3. Field Naming Rules

- Prefer clear business-semantic names over abbreviations
- Use snake_case column names
- Avoid SQL/MySQL keywords and high-conflict names such as `type`, `order`, `level`, `desc`, `key`, and `template`
- If a keyword-like concept is required, expand it into a safer business name such as `status_code`, `sort_order`, or `template_content`
- Boolean fields must not use an `is_` prefix
- Prefer boolean names like `enabled`, `deleted`, `visible`, `disabled`, and `sys_build_in`
- Time fields follow repository style and use `*_time`
- Business numbers use `*_no`
- Business codes use `*_code`
- Parent-child relationships use `parent_id`

## 4. Common Shared Fields

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

- Prefer `NOT NULL` when the business meaning is mandatory and default values are stable
- Allow `NULL` only when "unknown", "not applicable", or deferred fill is meaningful
- Comments are required for tables and fields

## 5. Type Selection Rules

### Identifiers And Relationship Columns

- Primary keys and internal references default to `bigint`
- Keep referenced and referencing columns aligned

### String Fields

- Prefer `varchar` when the upper bound is known or can be constrained
- Typical business labels, names, and codes should remain bounded rather than using `text`

### Boolean-Like Fields

- Use a repository-consistent small numeric or boolean-compatible type as needed by the surrounding table style
- Name the field semantically, not with `is_`

### Status / Type / Code Fields

- Use bounded string columns such as `varchar(20)` or `varchar(32)` when the values are code-like and human readable
- Prefer `*_code` suffixes for business-state fields instead of raw ambiguous names

### JSON Fields

- Use JSON only when the structure is truly nested, variable, or operationally expensive to normalize
- Do not hide clear relational structure inside JSON
- JSON fields require explicit DO mapping with a type handler such as `JacksonTypeHandler`

### Long Text

- `text` and `longtext` are not default choices for the main table
- Default to `varchar` unless the content size is clearly unbounded or very large
- If a field may exceed bounded string limits or is not needed in high-frequency queries, evaluate split-table storage first

## 6. Long-Text And Split-Table Rules

Default rule:

- `text` and `longtext` fields should normally live in an extension table or a dedicated long-text table

Prefer split-table storage when any of the following is true:

- The field can clearly exceed `varchar(2000)` scale
- The field is only needed for detail pages
- The list or page query should not read the field
- The entity would otherwise carry multiple large text fields
- The content is a template body, rich text, large JSON blob, audit snapshot, or message body

Allow keeping large text on the main table only when all of the following are true:

- The field is core to the entity
- It is almost always read with the main record
- Data volume is controlled
- Split-table complexity is not justified

If large text stays on the main table, the design must explain why it was not split.

Reference pattern:

- `sys_long_text` plus `SysLongTextDO` shows an existing repository pattern for dedicated long-text storage

## 7. Index And Constraint Rules

- Every table has a primary key on `id`
- Unique constraints must reflect a real business invariant
- For soft-delete tables, business unique indexes often include `deleted`
- Do not add indexes without a query, lookup, deduplication, or ordering reason
- Prefer concise indexes aligned with actual predicates
- Relation tables should usually constrain duplicate relationships with a unique index

Common examples:

- Unique business code: `UNIQUE KEY (business_code, deleted)` for soft-delete tables
- Lookup index: `KEY idx_parent_id (parent_id)`
- Relation uniqueness: `UNIQUE KEY (order_id, item_id, deleted)` when duplicate relationships are not allowed

## 8. Relationship Rules

- Internal table relationships default to `xxx_id`
- Do not use `xxx_no` or `xxx_code` as the default foreign-key substitute for internal relations
- Parent-child trees use `parent_id`
- Keep relationship columns typed exactly like the referenced table key
- Normalize true one-to-many and many-to-many structures; do not pack them into CSV strings or JSON arrays when relational queries are expected

## 9. Redundant Business Identifier Rules

- `xxx_no` and `xxx_code` may be redundantly stored in related tables
- Redundancy is recommended for key business chains such as orders, payments, refunds, notifications, and reconciliation records
- Redundant identifiers improve direct querying, exports, logs, troubleshooting, reconciliation, and external passthrough
- Redundant identifiers are mirrors, not the default source of truth for referential structure
- If redundancy exists, define where the canonical value comes from and treat the redundant value as read-only by default
- Add indexes to redundant identifiers only when a real query or lookup scenario exists

Recommended pattern:

- Keep `order_id` as the internal relation
- Optionally also store `order_no`
- Keep `payment_id` as the internal relation
- Optionally also store `payment_no` or `payment_code`

## 10. DO And MyBatis-Plus Alignment

- Entity classes should end with `DO`
- Use `@TableName` on each DO
- New tables should normally be designed to fit `AbstractAuditableEntity` unless there is a justified alternative
- Soft-delete tables should use a `deleted` field compatible with repository logical-delete conventions
- JSON columns need explicit `@TableField(typeHandler = ...)` handling
- Column names should map cleanly to Java camelCase fields without awkward exceptions
- Avoid schema names that would force excessive escaping or confusing DO field names

## 11. Technical Design Output Checklist

A good design answer should cover:

1. What tables are needed, and which one is the aggregate root
2. Which fields are mandatory, optional, derived, or redundant
3. Why each unique constraint or index exists
4. Whether large text should be split out
5. Whether JSON is justified
6. How the schema maps to DO fields and repository base entities
7. Which open questions still need business confirmation

## 12. Common Anti-Patterns

- Using composite primary keys
- Using business numbers or codes as the default internal relation key
- Storing large text on a hot main table without justification
- Naming booleans with `is_`
- Mixing `*_time` and `*_at`
- Using keyword or near-keyword field names when a clearer business name exists
- Adding indexes "just in case"
- Hiding relational data inside JSON with no clear reason
