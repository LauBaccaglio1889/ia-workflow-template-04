---
id: PRD-001
slug: db-analysis-ai-assistant
title: App de Análisis de Bases de Datos con IA
status: draft
base_branch: main
epic_branch: epic/PRD-001-db-analysis-ai-assistant
created: 2026-05-20
updated: 2026-05-20
---

# PRD-001: App de Análisis de Bases de Datos con IA

## 1. Executive Summary

La App de Análisis de Bases de Datos con IA es una herramienta full-stack diseñada para permitir a los usuarios (analistas, administradores o usuarios de negocio) interactuar con una base de datos relacional PostgreSQL utilizando lenguaje natural. En lugar de escribir complejas consultas SQL o depender de técnicos para extraer informes, los usuarios podrán simplemente conversar con un chatbot inteligente para explorar esquemas, obtener estadísticas y realizar preguntas analíticas complejas.

El MVP tiene como objetivo proporcionar una interfaz conversacional fluida y robusta, donde el backend gestione la seguridad de las consultas (read-only), persista el historial de conversaciones en una base de datos local SQLite y delegue el procesamiento lógico del lenguaje natural a un agente especializado construido con `pydantic-ai`.

## 2. Mission & Core Principles

**Misión**: Democratizar el acceso y análisis de datos relacionales garantizando la seguridad, rapidez y precisión a través del poder de agentes inteligentes de IA.

**Principios Core**:
1. **Seguridad Absoluta (Read-Only)**: El acceso a la base PostgreSQL de análisis debe estar estrictamente limitado a lecturas (`SELECT`), bloqueando cualquier intento de alteración o destrucción de datos.
2. **Contexto Inteligente y Eficiente**: Minimizar el uso de tokens y maximizar la precisión inyectando el contexto de esquema bajo demanda a través de herramientas de introspección (`list_tables`, `get_table_metadata`).
3. **Persistencia Fluida**: El usuario debe poder retomar cualquier conversación pasada sintiendo continuidad inmediata en su flujo de trabajo.
4. **Experiencia de Usuario Premium**: La interfaz debe ser viva, rápida, con estados de carga claros (skeletons) y representación estética (tablas dinámicas) de los resultados analíticos.

## 3. Target Users

*   **Analistas de Datos Jr.**: Necesitan acelerar la escritura de reportes y explorar nuevas bases de datos sin tener que memorizar todo el esquema.
*   **Usuarios de Negocio / Managers**: Requieren respuestas analíticas inmediatas (ej. "¿Cuáles fueron los 5 clientes que más gastaron este mes?") sin intermediarios técnicos.
*   **Desarrolladores**: Quieren probar de forma rápida hipótesis sobre sus bases de datos PostgreSQL locales a través de una interfaz de chat ágil.

## 4. MVP Scope

### In Scope (MVP)

*   **Frontend**:
    *   [ ] Chatbot interactivo con burbujas de chat diferenciadas (Usuario vs. IA).
    *   [ ] Sidebar lateral para crear nuevas sesiones, listarlas, renombrarlas y eliminarlas.
    *   [ ] Formateo estético de tablas markdown para los resultados de las consultas analíticas.
    *   [ ] Indicador visual de carga (typing indicator / skeleton) mientras el agente procesa y ejecuta.
*   **Backend**:
    *   [ ] Persistencia relacional de sesiones y mensajes históricos en base de datos SQLite.
    *   [ ] Integración del agente inteligente usando `pydantic-ai`.
    *   [ ] Tool `list_tables` para que el agente inspeccione las tablas PostgreSQL.
    *   [ ] Tool `get_table_metadata` para obtener columnas, tipos y claves foráneas de tablas específicas.
    *   [ ] Tool `execute_query` para ejecutar únicamente consultas `SELECT` validadas estrictamente en el backend.
    *   [ ] Script de inicialización de datos de prueba en PostgreSQL (Dataset Chinook o similar).

### Out of Scope (Post-MVP)

*   [ ] Modificación de la base de datos (queries `INSERT`, `UPDATE`, `DELETE`, `DROP`, `ALTER`).
*   [ ] Soporte para múltiples conexiones PostgreSQL concurrentes o base de datos multi-tenant.
*   [ ] Exportación avanzada a formatos Excel, CSV o PDF.
*   [ ] Generación visual interactiva de gráficos de barras, tortas o líneas (Charts).

## 5. User Stories

### STORY-001: Gestión de Sesiones (Persistencia)
*   **Como** usuario del sistema,
*   **Quiero** crear, renombrar y eliminar diferentes hilos de chat,
*   **Para** poder organizar mis diferentes tareas de análisis y recuperar el historial de mis consultas previas.
*   *Ejemplo*: El usuario hace clic en "+ Nueva Conversación", se genera un chat en blanco titulado automáticamente "Nueva Consulta". El usuario chatea sobre ventas y el backend le actualiza el título a "Ventas de Música".

### STORY-002: Interfaz Conversacional de Chat (UI)
*   **Como** usuario no técnico,
*   **Quiero** chatear en una interfaz limpia con un input de texto y burbujas de diálogo legibles,
*   **Para** poder consultar a la base de datos en lenguaje natural sin fricciones cognitivas.
*   *Ejemplo*: El usuario escribe "¿Cuántos artistas tenemos?" y ve inmediatamente un estado de carga "El agente está analizando..." antes de que aparezca la respuesta en un globo verde/oscuro premium.

### STORY-003: Herramientas de Introspección del Esquema (Backend Tools)
*   **Como** agente inteligente de base de datos,
*   **Quiero** tener acceso a las herramientas `list_tables` y `get_table_metadata`,
*   **Para** auto-descubrir las tablas de PostgreSQL y responder preguntas sin que el desarrollador tenga que inyectarme todo el esquema estáticamente en el System Prompt.
*   *Ejemplo*: Cuando el usuario pregunta sobre los clientes, el agente ejecuta internamente `list_tables` para detectar la tabla `Customer` y luego invoca `get_table_metadata` para saber que tiene las columnas `CustomerId`, `FirstName` y `LastName`.

### STORY-004: Ejecución Segura de Consultas SELECT (Backend Secure Query)
*   **Como** administrador del sistema,
*   **Quiero** que el backend analice y filtre estrictamente cualquier consulta SQL generada por el agente antes de enviarla a PostgreSQL,
*   **Para** mitigar riesgos de inyección SQL destructiva, timeouts de consultas masivas y fugas de memoria.
*   *Ejemplo*: Si el agente intenta ejecutar `DROP TABLE Track;` o `SELECT * FROM Track; UPDATE Track SET Name='Hack' WHERE TrackId=1;`, el backend rechaza la ejecución arrojando un error de violación de seguridad.

### STORY-005: Integración End-to-End y Coherencia de Contexto (Agent & Memory)
*   **Como** usuario,
*   **Quiero** que el chatbot recuerde mis preguntas y respuestas anteriores dentro de la misma sesión activa,
*   **Para** poder hacer preguntas de seguimiento dinámicas sin tener que volver a explicar el contexto en cada mensaje.
*   *Ejemplo*:
    *   *Mensaje 1*: "¿Quién es el cliente con mayor facturación?" (Agente responde: "Luís Rocha").
    *   *Mensaje 2*: "¿De qué país es?" (Agente infiere que se refiere a "Luís Rocha" y responde: "Brasil").

## 6. Core Architecture & Patterns

El proyecto está diseñado como un monorepo completamente desacoplado (Frontend React y Backend FastAPI).

```
ai-template-clase04/
├── frontend/                       # React 19 SPA
│   ├── src/
│   │   ├── components/             # Sidebar, ChatArea, TableRenderer
│   │   ├── layouts/                # RootLayout
│   │   └── App.jsx                 # Routing y Core State
├── backend/                        # FastAPI
│   ├── app/
│   │   ├── core/                   # Configuración, DB (SQLite/Postgres engines)
│   │   ├── models/                 # Modelos SQLAlchemy para SQLite (Sessions, Messages)
│   │   ├── repositories/           # Capa de datos para SQLite
│   │   ├── routers/                # Endpoints API (sessions.py, chat.py)
│   │   ├── services/               # Lógica del agente Pydantic AI y herramientas Postgres
│   │   └── main.py                 # Punto de entrada FastAPI
```

### Patrones de Diseño
*   **Agent Tools Pattern**: El agente `pydantic-ai` utiliza el patrón de herramientas dinámicas (tools) decoradas para interactuar de forma segura e incremental con la base PostgreSQL externa.
*   **Layered Repository Pattern**: Acceso a la base SQLite del backend encapsulado en la capa Repository para facilitar la legibilidad y testing unitario.
*   **Strict SQL Parsing Middleware**: Filtro de sanitización y control de seguridad antes de la ejecución de queries en PostgreSQL.
*   **Verificación de Interfaz E2E**: El flujo visual de pantallas, creación de sesiones y visualización de tablas en la UI frontend será verificado sistemáticamente mediante automatizaciones de navegación interactiva documentadas en la skill `agent-browser` (usando comandos secuenciales de apertura, snapshot, click y completado de formularios).

## 7. Tools & Features Specification

### Agente `pydantic-ai`
Definido en `backend/app/services/agent.py` utilizando un modelo de lenguaje adecuado (ej. OpenRouter Claude / GPT).

*   **System Prompt**:
    *   Rol: "Eres un experto analista de bases de datos PostgreSQL de lectura."
    *   Instrucciones estrictas: "Solo respondes preguntas sobre los datos disponibles en la base. Si te piden escribir, alterar, borrar o hacer cualquier acción no analítica, debes rechazarla amablemente."
    *   Estrategia de resolución: "Antes de responder, explora el esquema usando `list_tables` y `get_table_metadata`. No inventes tablas ni columnas."

### Tools de Base de Datos
1.  **`list_tables`**:
    *   Firma: `list_tables() -> List[str]`
    *   Implementación: Usa el Inspector de SQLAlchemy para retornar los nombres de las tablas en el esquema público de la base Postgres activa.
2.  **`get_table_metadata`**:
    *   Firma: `get_table_metadata(table_name: str) -> Dict[str, Any]`
    *   Implementación: Extrae columnas (nombre, tipo de dato), claves primarias y claves foráneas de la tabla dada.
3.  **`execute_query`**:
    *   Firma: `execute_query(sql_query: str) -> List[Dict[str, Any]]`
    *   Implementación: Valida la query en la capa de seguridad, la ejecuta a través de una conexión read-only a Postgres y retorna una lista de diccionarios con los resultados.

## 8. Technology Stack

*   **Frontend**:
    *   React 19 + Vite (SPA)
    *   React Router 7
    *   Tailwind CSS v4 + shadcn/ui
    *   Lucide React (Iconografía premium)
*   **Backend**:
    *   FastAPI 0.115+
    *   `pydantic-ai` 0.0.18+ (para la orquestación del agente)
    *   SQLAlchemy 2.x (ORM tanto para SQLite de sesiones como para conexión PostgreSQL de análisis)
    *   `psycopg2-binary` o `asyncpg` (driver Postgres)
    *   SQLite (Base de datos local embebida para persistencia de sesiones)
*   **Testing & E2E Verification**:
    *   `agent-browser` (Herramienta interactiva de simulación de navegador para validación funcional de la UI).

## 9. Security & Configuration

### Seguridad de Consultas (Read-Only SQL Enforcement)
El backend implementará un sistema multicapa de mitigación de riesgos:
1.  **Conexión Restringida**: La URI de conexión de Postgres configurada en el backend DEBE apuntar a un usuario con privilegios exclusivos de lectura (`GRANT SELECT ON ALL TABLES IN SCHEMA public TO readonly_user`).
2.  **Filtro Sintáctico Estricto**:
    *   Se eliminan espacios en blanco al inicio.
    *   La query debe empezar obligatoriamente con la palabra reservada `SELECT`.
    *   Búsqueda case-insensitive de tokens prohibidos (`INSERT`, `UPDATE`, `DELETE`, `DROP`, `ALTER`, `TRUNCATE`, `CREATE`, `REPLACE`, `GRANT`, `REVOKE`, `INTO`, `COPY`).
    *   Bloqueo del carácter `;` para prevenir inyección multi-query.
3.  **Límite y Timeout**:
    *   Se establece un timeout de consulta en SQLAlchemy/Postgres de máximo 5 segundos.
    *   Se inyecta automáticamente un `LIMIT 100` si la consulta carece de cláusula limitadora.

### Configuración (Variables de Entorno)
El archivo `backend/.env` debe contar con:
*   `DATABASE_URL_SQLITE` (ej: `sqlite:///./sessions.db`)
*   `DATABASE_URL_POSTGRES` (ej: `postgresql://readonly_user:password@localhost:5432/chinook`)
*   `OPENROUTER_API_KEY` o `OPENAI_API_KEY` (clave para el modelo del agente)
*   `ALLOWED_ORIGINS` (ej: `http://localhost:5173`)

## 10. API Specification

Todas las rutas living bajo el prefijo `/api/v1`.

### Sesiones
*   `POST /sessions` -> Crea una nueva sesión. Retorna el UUID y el título default.
*   `GET /sessions` -> Lista todas las sesiones activas ordenadas por `updated_at` desc.
*   `PUT /sessions/{id}` -> Permite renombrar la sesión (cambiar título).
*   `DELETE /sessions/{id}` -> Elimina la sesión y todos sus mensajes asociados en cascada.

### Chat conversacional
*   `POST /sessions/{id}/chat` -> Envía un mensaje del usuario. Retorna la respuesta generada por el agente.
    *   *Request Body*: `{"message": "string"}`
    *   *Response Body*: `{"id": "string", "role": "model", "content": "string", "created_at": "datetime"}`
*   `GET /sessions/{id}/messages` -> Recupera el historial completo de la conversación de una sesión en orden cronológico.

## 11. Success Criteria

*   **Criterio de MVP Funcional**: El usuario puede crear una sesión, preguntar "¿Qué tablas tenemos disponibles?", recibir una respuesta correcta, y luego preguntar "¿Cuántos clientes viven en Brasil?" obteniendo el número preciso derivado de la base de datos PostgreSQL de prueba.
*   **Indicadores de Calidad Visual**:
    *   Uso de sombras elegantes y modo oscuro consistente (Tailwind CSS v4).
    *   Cero parpadeos bruscos en el frontend gracias al uso de transiciones CSS suaves y skeleton loaders durante la carga de respuestas.
    *   Tablas SQL visualizadas estéticamente en grids auto-ajustables en el chat de frontend.
*   **Robustez de Seguridad**: Intentos de ingresar queries destructivas (ej. `Borra la tabla de artistas`) deben ser neutralizados y responder amablemente con un mensaje de rechazo de operación no permitida.
*   **Automatización de Testing**: Todas las pantallas interactivas de frontend (creación de sesión, selección de sesión, envío del formulario de chat y renderizado de tablas) deben ser testeables de punta a punta de manera exitosa con el pipeline interactivo de `agent-browser`.

## 12. Implementation Phases

### Fase 1: Cimientos de Persistencia (SQLite & FastAPI local)
*   **Objetivo**: Montar el backend con la SQLite de sesiones, esquemas Pydantic y controladores API CRUD `/sessions` y `/sessions/{id}/messages`.
*   **Entregables**: Modelos de base de datos SQLAlchemy, migraciones y endpoints REST funcionales.
*   **Validación**: Solicitudes `curl` exitosas de creación, listado y eliminación de sesiones.

### Fase 2: Agente Inteligente y Tools de Introspección
*   **Objetivo**: Implementar el agente `pydantic-ai` con acceso de lectura seguro a PostgreSQL y las herramientas de introspección de esquemas (`list_tables`, `get_table_metadata`).
*   **Entregables**: Lógica de agente en `backend/app/services/agent.py`, capa de seguridad para la ejecución de queries con whitelist e inyección de límites.
*   **Validación**: Scripts locales de backend que ejecuten consultas complejas de prueba a Postgres mediante el agente de IA y muestren el comportamiento frente a intentos maliciosos.

### Fase 3: Interfaz Web Conversacional (React Frontend)
*   **Objetivo**: Construir la interfaz de chat en React 19, conectándola a los endpoints del backend.
*   **Entregables**: Componente Sidebar lateral dinámico, área principal de Chat conversacional con scroll automático, visualizador interactivo de tablas.
*   **Validación**: Validación fluida de flujos visuales interactivos usando la herramienta `agent-browser`.

### Fase 4: Pulido, Seguridad E2E y Cierre
*   **Objetivo**: Ajuste de timeouts, manejo de errores robustos conversacionales, estética de UI (skeletons y modo oscuro) y preparación de scripts de demo.
*   **Entregables**: Repositorio completo con todas las piezas comunicándose limpiamente y scripts de carga del dataset Postgres Chinook.
*   **Validación**: Ejecución exitosa de preguntas analíticas guiadas por el cliente.

## 13. Future Considerations

*   Soporte para gráficos interactivos (Charts como Chart.js o Recharts) de forma directa en las burbujas de chat.
*   Indexación semántica (Embeddings) de la base de datos para que el agente entienda sinónimos complejos en las consultas de negocio sin consultar la metadata repetidamente.
*   Caché semántica para consultas idénticas ejecutadas con anterioridad, reduciendo costos de LLM y DB.

## 14. Risks & Mitigations

*   **Riesgo 1**: Inyección SQL destructiva o fuga de datos a través de bypasses avanzados del prompt del sistema del agente.
    *   *Mitigación*: Validación estricta a nivel backend de la query SQL cruda (regex, bloqueo de semicolon, tokens prohibidos) sumado a la configuración imperativa de privilegios read-only a nivel base de datos PostgreSQL.
*   **Riesgo 2**: Consumo excesivo de tokens de contexto si el agente lee repetidamente la metadata de múltiples tablas grandes.
    *   *Mitigación*: Implementación de herramientas bajo demanda (`get_table_metadata` se invoca únicamente para las tablas de interés y no en cada llamada inicial).
*   **Riesgo 3**: Bloqueo de la interfaz si una consulta tarda demasiado en ejecutarse en PostgreSQL.
    *   *Mitigación*: Limitación estricta de tiempo de ejecución (Timeout 5 segundos) en el driver de conexión PostgreSQL y estados de carga no bloqueantes en la UI con indicador visual de detención.
*   **Riesgo 4**: Falta de base PostgreSQL local para pruebas por parte del estudiante o reviewers.
    *   *Mitigación*: Proporcionar un script en backend `app/core/setup_dev_db.py` que configure y cargue automáticamente un set de datos Chinook en SQLite o un Postgres local de forma inmediata.

## 15. Appendix

*   **Dependencias Críticas**:
    *   `pydantic-ai`
    *   `fastapi`
    *   `sqlalchemy`
    *   `psycopg2-binary`
*   **Skills referenced**: `agent-browser`
