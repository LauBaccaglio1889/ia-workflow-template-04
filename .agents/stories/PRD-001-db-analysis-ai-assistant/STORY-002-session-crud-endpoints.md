---
id: STORY-002
prd: PRD-001
slug: session-crud-endpoints
title: Endpoints CRUD de Gestión de Sesiones en FastAPI
type: feature
priority: high
complexity: medium
phase: 1
status: todo
labels: [backend, api]
epic_branch: epic/PRD-001-db-analysis-ai-assistant
plan: null
report: null
commit: null
depends_on: [STORY-001]
blocks: [STORY-003, STORY-006]
skills: []
created: 2026-05-20
updated: 2026-05-20
---

# STORY-002: Endpoints CRUD de Gestión de Sesiones en FastAPI

## Description

As a user, I want to manage my chat sessions via REST API endpoints, so that the web interface can fetch, create, rename, and delete conversation threads.

## Acceptance Criteria

- [ ] Given a POST request to `/api/v1/sessions`, when the request is sent, then a new session is created with a default title like "Nueva Consulta" and returned with a 201 status code.
- [ ] Given a GET request to `/api/v1/sessions`, when requested, then it returns a list of all active sessions sorted chronologically by `updated_at` descending.
- [ ] Given a PUT request to `/api/v1/sessions/{id}` with a `{"title": "Nuevo Título"}` body, when executed, then the session title is updated and returned successfully.
- [ ] Given a DELETE request to `/api/v1/sessions/{id}`, when executed, then the session and its history are deleted and a 204 status is returned.
- [ ] Given a GET request to `/api/v1/sessions/{id}/messages`, when executed, then the chronologically sorted history of messages is returned.

## Technical Notes

- Implement routers in `backend/app/routers/sessions.py`.
- Define request and response schemas in `backend/app/schemas/session.py`.
- Register the new router in `backend/app/main.py` under the `/api/v1` prefix.
- Implement corresponding database CRUD operations using SQLAlchemy session dependencies.

## Dependencies

- **Blocked by**: STORY-001
- **Blocks**: STORY-003, STORY-006

## PRD Reference

Source: [`PRD-001/PRD.md`](../../PRDs/PRD-001-db-analysis-ai-assistant/PRD.md) — Section 10 (API Specification)
