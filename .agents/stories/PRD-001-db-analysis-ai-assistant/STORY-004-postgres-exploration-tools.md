---
id: STORY-004
prd: PRD-001
slug: postgres-exploration-tools
title: Herramientas de Introspección y Exploración Postgres
type: technical
priority: high
complexity: medium
phase: 2
status: todo
labels: [backend, database]
epic_branch: epic/PRD-001-db-analysis-ai-assistant
plan: null
report: null
commit: null
depends_on: []
blocks: [STORY-005]
skills: []
created: 2026-05-20
updated: 2026-05-20
---

# STORY-004: Herramientas de Introspección y Exploración Postgres

## Description

As an agent system developer, I want to implement connection settings and schema introspection tools for the PostgreSQL database, so that the AI agent can dynamically discover tables and columns under demand.

## Acceptance Criteria

- [ ] Given a database connection configuration, when the backend starts, then it connects successfully to the target PostgreSQL database using environment variables.
- [ ] Given the `list_tables` tool, when executed, then it uses SQLAlchemy inspector to query and return a list of all table names available in the public schema.
- [ ] Given the `get_table_metadata` tool with a table name, when executed, then it queries and returns table fields (name, type), primary keys, and foreign keys in a structured format.

## Technical Notes

- Implement target Postgres connection engine in `backend/app/core/database.py`.
- Define introspection functions under `backend/app/services/db_tools.py` using standard SQLAlchemy inspection (`inspect(engine)`).
- Handle common errors gracefully (e.g. target table not found, lack of connection privileges).

## Dependencies

- **Blocked by**: None
- **Blocks**: STORY-005

## PRD Reference

Source: [`PRD-001/PRD.md`](../../PRDs/PRD-001-db-analysis-ai-assistant/PRD.md) — Section 7 (Tools & Features Specification - Tools de Base de Datos)
