---
id: STORY-005
prd: PRD-001
slug: secure-query-execution-tool
title: Herramienta de Ejecución Segura de Consultas SELECT
type: technical
priority: high
complexity: medium
phase: 2
status: todo
labels: [backend, database, security]
epic_branch: epic/PRD-001-db-analysis-ai-assistant
plan: null
report: null
commit: null
depends_on: [STORY-004]
blocks: [STORY-006]
skills: []
created: 2026-05-20
updated: 2026-05-20
---

# STORY-005: Herramienta de Ejecución Segura de Consultas SELECT

## Description

As a system administrator, I want the backend to strictly sanitize, validate, and constrain any SQL query before sending it to PostgreSQL, so that malicious queries are rejected before causing data mutations or denial of service.

## Acceptance Criteria

- [ ] Given a SQL query, when parsed by the validation middleware, then it must strictly start with the `SELECT` keyword (ignoring leading whitespace and comments), rejecting any other start token.
- [ ] Given a SQL query, when validated, then it is rejected if it contains any forbidden keywords like `INSERT`, `UPDATE`, `DELETE`, `DROP`, `ALTER`, `TRUNCATE`, `CREATE`, `REPLACE`, `GRANT`, `REVOKE`, `INTO` (case-insensitive check) or the semicolon character `;`.
- [ ] Given a valid `SELECT` query, when evaluated, then it is automatically modified to include a `LIMIT 100` clause if no limits are specified, and executed with a connection timeout of 5 seconds.
- [ ] Given an invalid or blocked query, when executed, then it raises an explicit custom validation error explaining the security policy.

## Technical Notes

- Build query sanitation functions in `backend/app/services/query_validator.py`.
- Integrate validation logic into the `execute_query` tool in `backend/app/services/db_tools.py`.
- Throw clean exceptions that can be safely processed by the Pydantic AI agent or returned to the client.

## Dependencies

- **Blocked by**: STORY-004
- **Blocks**: STORY-006

## PRD Reference

Source: [`PRD-001/PRD.md`](../../PRDs/PRD-001-db-analysis-ai-assistant/PRD.md) — Section 9 (Security & Configuration - Seguridad de Consultas)
