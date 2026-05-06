# Consigna — Sprint 1: App de Análisis de Bases de Datos con IA

## Contexto

Construir una aplicación full-stack que permita a un usuario conversar con una base de datos PostgreSQL mediante un agente de IA. El agente debe poder explorar el esquema y ejecutar consultas `SELECT` para responder preguntas analíticas en lenguaje natural.

Este repositorio (`ai-template-clase04`) es la base de partida. Cada alumno debe usar el workflow definido en `workflow.md` para generar su PRD, dividirlo en stories, planificar e implementar.

---

## Objetivo del Sprint 1

Generar el **PRD** (Product Requirements Document) de la aplicación usando el comando `/create-prd`, completar un Q/A con el agente para refinar requerimientos, y dejar el documento listo para la fase de stories.

---

## Requerimientos funcionales

### Frontend

1. Pantalla con un **chatbot** de IA donde el usuario pueda escribir mensajes y recibir respuestas.
2. **Persistencia de sesiones**: el usuario puede retomar conversaciones previas. Cada sesión guarda su historial de mensajes.
3. El usuario chatea con los datos expuestos por el backend FastAPI — no escribe SQL, escribe preguntas en lenguaje natural.

### Backend

1. **Dos conexiones a base de datos**:
   - **SQLite**: persistencia de sesiones de conversación e historial de mensajes.
   - **PostgreSQL**: base de datos sobre la que el agente ejecuta consultas analíticas (read-only).
2. **Agente `pydantic-ai`** con acceso a la base PostgreSQL.
3. Herramientas (tools) del agente:
   - `list_tables` — listar tablas disponibles en la base.
   - `get_table_metadata` — obtener metadata (columnas, tipos, constraints) de una tabla relevante.
   - `execute_query` — ejecutar queries **solo `SELECT`** (validación estricta, sin DML/DDL).
4. **Prompt de sistema** específico: el agente solo responde sobre análisis de datos. Rechaza pedidos fuera de scope (escritura, modificación, temas no relacionados).

---

## Stack sugerido

- **Backend**: FastAPI, `pydantic-ai`, SQLAlchemy/SQLModel, `psycopg` o `asyncpg` para Postgres, SQLite para sesiones.
- **Frontend**: a elección del alumno React.
- **LLM**: Openrouter, yo les voy a dar la api-key.

---

## Workflow esperado

1. Leer `AGENTS.md` y entender el flujo `.agents/`.
2. Ejecutar `/create-prd` y mantener una sesión de **Q/A con el agente** para definir:
   - Alcance del MVP vs. fuera de alcance.
   - User stories concretas.
   - Modelo de datos de sesiones (esquema SQLite).
   - Esquema de la base PostgreSQL de prueba (qué dataset usar).
   - Política de seguridad para `execute_query` (whitelist `SELECT`, timeout, límite de filas).
   - Manejo de errores del agente (query inválida, tabla inexistente, etc.).
   - Estrategia de contexto: ¿cómo se le pasa el historial al LLM en cada turno?
3. Validar el PRD generado en `.agents/PRDs/PRD-XXX-{slug}/PRD.md`.
4. Generar stories con `/create-stories` cuando el PRD esté aprobado.