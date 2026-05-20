---
id: STORY-001
prd: PRD-001
slug: db-session-model-and-migrations
title: Modelo de Datos de Sesión y Migraciones SQLite
type: technical
priority: high
complexity: small
phase: 1
status: todo
labels: [backend, database]
epic_branch: epic/PRD-001-db-analysis-ai-assistant
plan: null
report: null
commit: null
depends_on: []
blocks: [STORY-002]
skills: []
created: 2026-05-20
updated: 2026-05-20
---

# STORY-001: Modelo de Datos de Sesión y Migraciones SQLite

## Description

As a developer, I want to create the SQLite session and message schemas in SQLAlchemy, so that conversational history can be reliably persisted between application restarts.

## Acceptance Criteria

- [ ] Given a SQLite database, when the application starts, then the `sessions` table (with `id`, `title`, `created_at`, `updated_at`) and the `messages` table (with `id`, `session_id`, `role`, `content`, `created_at`) are successfully created or checked for existence.
- [ ] Given a SQLAlchemy session, when a new session record is saved, then it has a unique UUID as its primary key.
- [ ] Given a session record, when it is deleted, then all related message records in the `messages` table are deleted via database cascade.

## Technical Notes

- Define the SQLAlchemy models in `backend/app/models/session.py` (or check existing database engine configurations under `backend/app/core/database.py`).
- Implement helper methods or engine checks to run `Base.metadata.create_all(bind=engine)` upon application startup.
- Follow `snake_case` database field names and `PascalCase` class names as specified in `AGENTS.md`.

## Dependencies

- **Blocked by**: None
- **Blocks**: STORY-002

## PRD Reference

Source: [`PRD-001/PRD.md`](../../PRDs/PRD-001-db-analysis-ai-assistant/PRD.md) — Section 3 & 6 (Core Architecture & SQLite schema)
