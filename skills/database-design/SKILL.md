---
name: database-design
description: Use when designing database schemas or writing technical design sections for backend codebases, especially when table structure, field naming, indexing, long-text splitting, redundant business identifiers, or DO/MyBatis-Plus mapping must follow repository conventions
---

# Database Design

## Overview

Use this skill to produce database designs that match repository conventions instead of generic schema advice.

Before giving design recommendations, read `references/project-database-conventions.md`.

## When to Use

- Designing new business tables for a backend project
- Extending an existing table or splitting a large table
- Reviewing field naming, type selection, indexes, or relationship design
- Writing the database section of a technical design document
- Deciding whether to use JSON fields, long-text tables, or redundant business numbers/codes

Do not use this skill for migration rollout steps or general service-layer implementation.

## Workflow

1. Identify the modeling target: main table, relation table, extension table, or long-text table.
2. Apply the project rules from `references/project-database-conventions.md`.
3. Produce a design that can be pasted into the technical plan using the output structure below.
4. Run the self-check before finalizing the proposal.

## Output Structure

```markdown
## Data Model Design

### 1. Table Overview
### 2. Field Design
### 3. Index And Constraint Design
### 4. Relationship Design
### 5. Redundant Identifier Design
### 6. Long-Text And Extension-Table Design
### 7. DO Mapping Notes
### 8. Risks And Open Questions
```

## Self-Check

- Use a single-column `id` primary key unless there is an explicit exception.
- Keep relationship columns on `xxx_id`; redundant `xxx_no` and `xxx_code` are optional mirrors, not default foreign keys.
- Prefer business-semantic field names and avoid SQL/MySQL keywords.
- Ban boolean names with the `is_` prefix.
- Use `*_time` for time columns.
- Default large text to split-table evaluation instead of putting `text` or `longtext` directly on the main table.
- Ensure the schema can map cleanly to a `*DO` class with `@TableName`, repository base entity conventions, and logical-delete rules where applicable.
