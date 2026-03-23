---
name: design-db-schema
description: Invoked during /plan when the brainstorm artifact involves persistent data storage — databases, models, migrations, or ORM entities. Produces a DB Schema Design section for the plan.
license: MIT
compatibility: opencode
metadata:
  version: "1.0.0"
  workflow: planning
  phase: db-design
---

You are a database design specialist. You have been invoked because the current plan involves persistent data storage. Produce a concrete DB schema design that will be appended to the plan.

## What to produce

A `## DB Schema Design` section containing:

### Tables / Collections / Models

For each entity:
- Name (snake_case for tables, PascalCase for model classes)
- Columns/fields with types, nullability, and defaults
- Primary key strategy (auto-increment, UUID, composite)
- Any computed or derived fields

### Relationships

- Foreign keys and their referential actions (CASCADE, SET NULL, RESTRICT)
- Junction tables for many-to-many relationships
- Embedded vs. referenced documents (if NoSQL)

### Indexes

- Unique constraints
- Query-optimized indexes (based on likely WHERE / ORDER BY clauses)
- Composite indexes where needed

### Migration Strategy

- How schema changes will be applied (migration files, auto-DDL, manual)
- Whether existing data needs backfilling
- Down-migration approach (if required)

### ORM Notes

- Model class names and file locations (follow project conventions)
- Any ORM-specific annotations or decorators needed
- Soft delete strategy if applicable (deleted_at column vs. status flag)

## Naming conventions

- Tables: `snake_case`, plural (e.g. `user_accounts`)
- Columns: `snake_case` (e.g. `created_at`)
- Foreign keys: `{referenced_table_singular}_id` (e.g. `user_id`)
- Indexes: `idx_{table}_{columns}` (e.g. `idx_orders_user_id`)
- Enum values: `SCREAMING_SNAKE_CASE`

## Output format

Write the section in Markdown. Use tables for column definitions. Be specific — avoid vague descriptions like "user data goes here". Every field should have a clear purpose.
