# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Setup

```bash
cp .env.example .env
# Completar INVGATE_URL, INVGATE_USER, INVGATE_PASS y los demás valores
```

## Commands

```bash
# Levantar todos los servicios (build + background)
docker compose up -d --build

# Ver logs del extractor en tiempo real
docker compose logs -f extractor

# Forzar recarga completa de la DB (descarta incremental y re-descarga todo)
docker compose exec extractor python extract.py --full

# Conectar a SQLite directamente
docker compose exec extractor sqlite3 /data/invgate.db

# Estado de servicios
docker compose ps

# Reiniciar un solo servicio
docker compose restart api

# Destruir todo incluyendo la DB
docker compose down -v
```

## Architecture

Cuatro componentes orquestados por Docker Compose, todos sobre la misma red interna `internal`:

```
InvGate API → extractor (Python + cron) → SQLite (WAL) ← api (Flask) ← nginx ← browser
```

**extractor** (`extractor/extract.py`): Script Python que corre al arranque y luego via cron. Extrae tickets de la API de InvGate de forma incremental: descarga lookups (estados, prioridades, tipos, categorías, agentes), obtiene todos los IDs actuales por estado, compara con la DB para calcular el delta (tickets nuevos o modificados desde `last_run_ts`), y descarga en paralelo con `ThreadPoolExecutor`. Persiste en SQLite con upsert. Guarda un `status_snapshot` en la tabla `meta` para el dashboard. El argumento `--full` ignora el incremental y re-descarga todo.

**SQLite compartido** (`/data/invgate.db` en el volumen `db_data`): Compartido entre `extractor` y `api` usando WAL mode para lectura concurrente. Tres tablas propias del extractor: `tickets` (con `raw_json` completo), `lookups`, `meta`. Dos tablas propias de la API: `dashboard_users` y `dashboard_views` (vistas guardadas por usuario).

**api** (`api/api.py`): Flask app en puerto 5000, no expuesta al exterior. Carga **todos** los tickets en memoria Python y filtra ahí mismo (no usa SQL para los filtros de período/área). El área de cada ticket se extrae en runtime desde `raw_json.custom_fields[AREA_CUSTOM_FIELD_ID]` porque no está normalizada como columna. Expone:
- `GET /api/metrics?period=<daily|weekly|monthly|prev_month|historical|custom>&area_id=<id>`
- `GET /api/health`
- `GET/POST /api/users/<email>/views`
- `DELETE /api/users/<email>/views/<name>`

**nginx** (`nginx/`): Sirve el HTML estático del dashboard y hace proxy de `/api/` a `http://api:5000/api/`.

## Key design decisions

**Extracción incremental**: El extractor compara `last_update` de cada ticket en la DB contra `last_run_ts`. Si `last_update > last_run_ts`, el ticket se vuelve a descargar. Esto significa que si InvGate actualiza un ticket entre corridas, se re-descarga completo.

**Lock de extracción**: `entrypoint.sh` usa `mkdir` para crear un lock de directorio en `/tmp/invgate-extract.lock`, previniendo corridas solapadas del cron.

**Área desde raw_json**: El campo de área no es un campo estándar de InvGate sino un campo personalizado configurable (`AREA_CUSTOM_FIELD_ID`). Se parsea en cada request de la API desde `raw_json` para no perder flexibilidad si el campo cambia de formato.

**Rate limiting de InvGate**: El extractor tiene throttling global via `REQUEST_LOCK` y `NEXT_REQUEST_AT` (garantiza mínimo `REQUEST_DELAY_SECONDS` entre requests), reintentos con backoff exponencial para 5xx, y respeta el header `Retry-After` en respuestas 429.

## Environment variables

| Variable | Default | Descripción |
|---|---|---|
| `INVGATE_URL` | — | URL base del portal (sin `/api`) |
| `INVGATE_USER` / `INVGATE_PASS` | — | Credenciales básicas |
| `DB_PATH` | `/data/invgate.db` | Ruta al SQLite |
| `AREA_CUSTOM_FIELD_ID` | `72` | ID del campo personalizado de área |
| `MAX_WORKERS` | `3` | Hilos paralelos para descarga |
| `HTTP_MAX_RETRIES` | `6` | Reintentos por request |
| `REQUEST_DELAY_SECONDS` | `0.25` | Pausa mínima entre llamadas HTTP |
| `RATE_LIMIT_BACKOFF_SECONDS` | `30` | Espera al recibir 429 sin `Retry-After` |
| `CRON_SCHEDULE` | `0 * * * *` | Frecuencia de extracción incremental |
| `DASHBOARD_PORT` | `8080` | Puerto expuesto por nginx al host |
