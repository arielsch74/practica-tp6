[![CI](https://github.com/arielsch74/practica-tp5/actions/workflows/ci.yml/badge.svg)](https://github.com/arielsch74/practica-tp5/actions/workflows/ci.yml)

# demo-fullstack

Sample full-stack de la cátedra **Ingeniería del Software 3 — UCC**. Es la app de referencia que se usa en las demos en vivo de las clases: chica a propósito, con la misma estructura que las guías de TP asumen para tu app del semestre.

**Stack**: backend **.NET 8** (minimal API) · frontend **React + Vite** · base de datos **PostgreSQL**.

## Qué hace

Una lista de tareas mínima. La API expone tres endpoints CRUD:

| Método | Ruta | Descripción |
|---|---|---|
| `GET` | `/api/tareas` | Lista todas las tareas |
| `POST` | `/api/tareas` | Crea una tarea (`{ "titulo": "..." }`) |
| `DELETE` | `/api/tareas/{id}` | Borra una tarea |

Además: `GET /health` para chequeos de disponibilidad.

## Estructura

```
├── backend/            # Solución .NET: DemoApi (API) + DemoApi.Tests (xUnit) + Dockerfile
├── frontend/           # SPA React con Vite (tests con vitest) + Dockerfile + nginx.conf
└── docker-compose.yml  # Sistema completo: db + backend + frontend
```

## Correr con Docker (recomendado)

```bash
cp .env.example .env    # password local de la BD (git ignora .env)
docker compose up -d --build
```

- Frontend: <http://localhost:3000> (nginx proxea `/api` al backend por la red interna)
- API: <http://localhost:8080/api/tareas> · Health: <http://localhost:8080/health>
- Los datos persisten en el volumen `db_data` (sobreviven a `docker compose down`; se borran con `down -v`)
- La BD arranca con `healthcheck` y el backend espera `service_healthy` (listo ≠ arrancado)

## Correr en local (sin Docker)

Necesitás un **PostgreSQL local** escuchando en `localhost:5432` con una database `app` (usuario `postgres` / password `postgres` — o ajustá `backend/DemoApi/appsettings.json`).

```bash
# Backend (crea el schema al arrancar) → http://localhost:8080
cd backend
dotnet run --project DemoApi

# Frontend (en otra terminal) → http://localhost:5173
cd frontend
npm install
npm run dev     # las llamadas a /api las proxea Vite al backend (vite.config.js)
```

## Tests

```bash
# Backend: 4 unit tests xUnit sobre la lógica de validación
cd backend && dotnet test Backend.sln

# Frontend: 4 unit tests vitest sobre lógica pura (sin DOM)
cd frontend && npm test -- --run
```

## CI

`.github/workflows/ci.yml`: en cada PR y push a `main` corren **dos jobs en paralelo** — backend (restore → build → test, con el reporte `.trx` como artefacto, subido aunque los tests fallen) y frontend (`npm ci` con cache de dependencias → test → build, con `dist/` como artefacto). El badge de arriba muestra el estado del último build de `main`.

## Ramas de demo (uso interno de la cátedra)

| Rama | Estado del repo | La usa |
|---|---|---|
| `demo-c2-inicio` | La app **sin nada de Docker** | Demo de la Clase 2 (dockerización en vivo) |
| `demo-c4-inicio` | Con Docker, **sin workflow de CI** | Demo de la Clase 4 (CI en vivo) |
| `demo-c4-paso1..3` | Workflow construido incrementalmente | Corridas pre-calentadas de la demo C4 (PRs abiertos contra `demo-c4-inicio`) |

> Nota: en las ramas `demo-c4-paso*` el trigger `pull_request` va **sin filtro de base** para que las corridas se disparen en PRs contra `demo-c4-inicio` (el YAML que enseña la guía filtra a `main`).
