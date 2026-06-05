# RedPandaFlow Infra

Docker Compose stack for RedPandaFlow, a collaborative kanban application.

## Services

- **db** — PostgreSQL 16 (alpine)
- **pgadmin** — pgAdmin 4, web UI for the database
- **backend** — ASP.NET Core API built from [redpandaflow-backend](https://github.com/RedPandaFlow/redpandaflow-backend)
- **frontend** — React SPA built from [redpandaflow-frontend](https://github.com/RedPandaFlow/redpandaflow-frontend), served by nginx

## Run locally

The compose file builds the backend and frontend images from sibling repos, so
clone all three side by side:

```bash
RedPandaFlow/
├── redpandaflow-backend/
├── redpandaflow-frontend/
└── redpandaflow-infra/
```

Copy `.env.example` to `.env`, fill in the values (DB credentials, JWT secret,
ports), then:

```bash
docker compose up -d --build
```

## Environment variables

| Variable          | Purpose                                                    |
| ----------------- | ---------------------------------------------------------- |
| `DB_USER`         | PostgreSQL user                                            |
| `DB_PASSWORD`     | PostgreSQL password                                        |
| `DB_REDPANDAFLOW` | PostgreSQL database name                                   |
| `DB_PORT`         | Host port mapped to PostgreSQL                             |
| `PGADMIN_EMAIL`   | pgAdmin login email                                        |
| `PGADMIN_PASSWORD`| pgAdmin login password                                     |
| `PGADMIN_PORT`    | Host port mapped to pgAdmin                                |
| `BACKEND_PORT`    | Host port mapped to the API                                |
| `FRONTEND_PORT`   | Host port mapped to the SPA                                |
| `JWT_SECRET_KEY`  | Signing key for JWT (`openssl rand -base64 48` to generate)|
| `VITE_API_URL`    | API URL baked into the frontend at build time              |

## Default ports

- Frontend → `http://localhost:5000`
- Backend → `http://localhost:5090`
- pgAdmin → `http://localhost:80`
- PostgreSQL → `localhost:5432`

## Persistent volumes

- `postgres_data` — database files
- `pgadmin_data` — pgAdmin configuration

Database content survives `docker compose down`. Use `docker compose down -v`
to wipe everything.

## Backup & restore

Two scripts in `scripts/` dump and restore the PostgreSQL database. They read
the credentials and database name from the running `db` container, so no extra
configuration is needed — the stack just has to be up (`docker compose up -d`).

Create a backup (custom-format dump written to `scripts/../backups/`):

```bash
./scripts/backup.sh
```

Restore a backup (drops and recreates the existing objects):

```bash
./scripts/restore.sh backups/redpandaflow_db-20260605-110131.dump
```

The `backups/` directory is git-ignored. Override the defaults with the
`DB_CONTAINER` and `BACKUP_DIR` environment variables if needed.

## Related repos

- [redpandaflow-backend](https://github.com/RedPandaFlow/redpandaflow-backend) — ASP.NET Core API
- [redpandaflow-frontend](https://github.com/RedPandaFlow/redpandaflow-frontend) — React
- [documentation](https://github.com/RedPandaFlow/documentation) — project documentation
