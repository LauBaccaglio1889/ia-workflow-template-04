---
id: STORY-003
prd: PRD-001
slug: session-sidebar-and-ui
title: Sidebar Lateral de Sesiones en React
type: feature
priority: high
complexity: medium
phase: 3
status: todo
labels: [frontend, ui]
epic_branch: epic/PRD-001-db-analysis-ai-assistant
plan: null
report: null
commit: null
depends_on: [STORY-002]
blocks: [STORY-007]
skills: [agent-browser]
created: 2026-05-20
updated: 2026-05-20
---

# STORY-003: Sidebar Lateral de Sesiones en React

## Description

As a user, I want a premium and functional sidebar in the web interface, so that I can easily create, select, rename, and delete my chat sessions with smooth visual feedback.

## Acceptance Criteria

- [ ] Given a web user, when they open the dashboard, then a modern sidebar is rendered on the left, displaying the list of all active sessions fetched from the backend.
- [ ] Given the sidebar list, when a session is clicked, then it becomes the active session, loading its message history into the main layout.
- [ ] Given a "+ Nueva Conversación" button, when clicked, then it sends a request to the backend and adds a new session smoothly to the top of the list without page reload.
- [ ] Given an active session, when the user clicks the edit or delete icon next to it, then they can rename or delete the session with instantaneous UI update.
- [ ] Visual Layout: Responsive design using HSL tailored colors and a professional dark mode as per `AGENTS.md` guidelines.

## Technical Notes

- Build components in `frontend/src/components/Sidebar.jsx` (or a similar folder structure matching `@/components`).
- Maintain core selected session state in `frontend/src/App.jsx` or a context provider.
- Utilise standard Lucide React icons for delete and edit actions.
- E2E Verification constraint from skill `agent-browser`: "Automates browser interactions for web testing, form filling, screenshots, and data extraction. Use when the user needs to navigate websites, interact with web pages, fill forms, take screenshots, test web applications, or extract information from web pages."
- Verify the sidebar rendering, session creation, selection, and deletion flows with a series of automated `agent-browser` navigation steps (open local dev server port, take snapshots, simulate click on add button, fill session name).

## Dependencies

- **Blocked by**: STORY-002
- **Blocks**: STORY-007

## PRD Reference

Source: [`PRD-001/PRD.md`](../../PRDs/PRD-001-db-analysis-ai-assistant/PRD.md) — Section 4 (MVP Scope - Frontend) & Section 11 (Success Criteria)
