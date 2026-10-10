# 🐳 Docker & PostgreSQL Workflow Cheat Sheet

A practical reference for local development, container management, database inspection, and schema migrations using Docker, Docker Compose, PostgreSQL, FastAPI, SQLAlchemy, Alembic, and `uv`.

## 1. 🛠️ Nuclear Reset — Complete Environment Cleanup

**Warning:** The following commands can permanently delete containers, images, build caches, networks, and database volumes. They affect the entire Docker environment, not just the current project. Do not run them on a machine hosting other projects or important data unless you intend to remove those resources.

### Complete Docker cleanup

```bash
# Remove all containers, including running and stopped containers
docker rm -f $(docker ps -aq) 2>/dev/null

# Remove all images
docker rmi -f $(docker images -aq) 2>/dev/null

# Remove unused resources, including unused volumes
docker system prune -a --volumes -f
```

| Command | Effect |
|---|---|
| `docker rm -f $(docker ps -aq)` | Forcefully removes all containers, including running and stopped containers. |
| `docker rmi -f $(docker images -aq)` | Attempts to remove all locally available images. Images referenced by remaining containers may require those containers to be removed first. |
| `docker system prune -a --volumes -f` | Removes unused containers, networks, images, build cache, and eligible anonymous volumes without prompting. |

**Important:** Docker cleanup is not equivalent to resetting a database schema. Removing volumes can permanently destroy PostgreSQL data. External volumes and resources used by other projects also require careful consideration.

### Safer project-specific reset

To stop and remove the current Compose project's containers and networks:

```bash
docker compose down
```

To also remove its Compose-managed volumes:

```bash
docker compose down -v
```

Use `down -v` only when you intend to delete the database's persisted state.

---

## 2. 📦 Container Management

### Local environment controls

| Command | Description |
|---|---|
| `docker compose up --build` | Build images if necessary and start the project's services. |
| `docker compose up -d --build` | Build and start services in the background. |
| `docker compose down` | Stop and remove project containers and networks. |
| `docker compose down -v` | Also remove Compose-managed volumes. |
| `docker compose stop` | Stop services without removing their containers. |
| `docker compose start` | Start previously created, stopped containers. |
| `docker compose restart` | Restart existing service containers. |
| `docker compose logs -f` | Follow logs from all project services. |
| `docker compose logs -f web` | Follow logs from the `web` service. |
| `docker compose config` | Validate and display the resolved Compose configuration, including interpolated environment variables. |
| `docker compose ps` | Show the project's containers and their status. |

To use the legacy Docker builder where supported:

```bash
COMPOSE_DOCKER_CLI_BUILD=0 docker compose up --build
```

This setting affects the Docker CLI build path in compatible setups. It is not a universal guarantee that every modern Compose installation will use the legacy builder.

### Diagnostic and inspection tools

| Command | Description |
|---|---|
| `docker ps` | List running containers. |
| `docker ps -a` | List all containers, including stopped and failed ones. |
| `docker inspect <container>` | Inspect detailed container configuration and state. |
| `docker stats` | Display live CPU, memory, network, and I/O statistics. |
| `docker logs <container>` | Display container logs. |
| `docker logs -f <container>` | Follow container logs in real time. |
| `docker exec -it <container> bash` | Open an interactive Bash shell if Bash is installed. |
| `docker exec -it <container> sh` | Open an interactive shell when `sh` is available. |

Examples using the container names `fast_web` and `fast_db`:

```bash
docker exec -it fast_web bash
docker exec -it fast_db sh
```

Container names depend on your Compose configuration. Check `docker compose ps` or `docker ps -a` if these names do not exist.

---

## 3. 🐘 PostgreSQL CLI Inspection (`psql`)

### Execute queries from the host terminal

Run SQL commands directly inside the PostgreSQL container without opening an interactive shell.

```bash
# List tables in the application database
docker exec -it fast_db \
  psql -U fast_user -d fast_db -c "\dt"

# Describe a table's structure
docker exec -it fast_db \
  psql -U fast_user -d fast_db -c "\d table_name"

# Query a table
docker exec -it fast_db \
  psql -U fast_user -d fast_db -c "SELECT * FROM users LIMIT 5;"

# Display the current database and connected user
docker exec -it fast_db \
  psql -U fast_user -d fast_db -c "SELECT current_database(), current_user;"
```

Replace `fast_db`, `fast_user`, and the table names with the values configured for your environment.

### Open an interactive PostgreSQL session

```bash
docker exec -it fast_db psql -U fast_user -d fast_db
```

Inside the `psql` prompt, use the following commands:

| Command | Action |
|---|---|
| `\dt` | List tables in the current schema. |
| `\dt *.*` | List tables across accessible schemas. |
| `\d table_name` | Describe columns, constraints, indexes, and related table details. |
| `\d+ table_name` | Show additional table information. |
| `\dn` | List schemas. |
| `\l` | List databases. |
| `\du` | List roles and their attributes. |
| `\conninfo` | Display connection information. |
| `\di` | List indexes. |
| `\x` | Toggle expanded query output. |
| `\timing` | Toggle query execution timing. |
| `\q` | Exit `psql`. |

### Useful SQL queries

```sql
-- List tables in the current schema
SELECT tablename
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY tablename;

-- Inspect the current Alembic revision
SELECT * FROM alembic_version;

-- Count rows in a table
SELECT COUNT(*) FROM users;

-- Inspect the current database and user
SELECT current_database(), current_user;
```

The `alembic_version` table normally stores the database's current migration revision. Its exact structure can vary with the migration configuration, including when multiple heads are involved.

---

## 4. 💾 Image & Volume Maintenance

### Images

| Command | Description |
|---|---|
| `docker images` | List locally available images. |
| `docker images -a` | Include intermediate image layers where applicable. |
| `docker image inspect <image>` | Inspect an image's configuration. |
| `docker image prune` | Remove dangling images after confirmation. |
| `docker image prune -a` | Remove images not referenced by existing containers, subject to Docker's cleanup rules. |
| `docker system df` | Show Docker disk usage. |

Use the force flag to skip confirmation:

```bash
docker image prune -f
```

### Volumes

| Command | Description |
|---|---|
| `docker volume ls` | List Docker-managed volumes. |
| `docker volume inspect <volume>` | Inspect volume configuration and mountpoint information. |
| `docker volume prune` | Remove eligible unused volumes after confirmation. |
| `docker volume prune -a` | Remove all eligible unused local volumes, including unused named volumes where supported. |

To prune eligible unused volumes without confirmation:

```bash
docker volume prune -f
```

**Important:** A volume's host mountpoint is generally useful for inspection, but directly editing database files is unsafe while PostgreSQL is running. Use PostgreSQL tools to modify database contents.

---

## 5. 🚀 Database Migrations — Alembic + `uv`

Run the following commands inside your application container, assuming it contains the project, its dependencies, the Alembic configuration, and the database connection settings.

Open a shell:

```bash
docker exec -it fast_web bash
```

### Standard migration workflow

```bash
# 1. Synchronize dependencies with the lockfile
uv sync --frozen

# 2. Generate a migration from SQLAlchemy model changes
uv run alembic revision --autogenerate -m "add email to users"

# 3. Review the generated migration file before applying it

# 4. Apply all pending migrations up to the current head
uv run alembic upgrade head

# 5. Check the current database revision
uv run alembic current
```

**Important:** `uv sync --frozen` synchronizes the environment using the existing lockfile without updating it. If dependencies have changed, update the lockfile deliberately before running this command.

Autogeneration compares SQLAlchemy metadata against the connected database schema. It does not automatically detect every change correctly, so review the generated `upgrade()` and `downgrade()` functions.

### Rollback and migration history

```bash
# Roll back one revision
uv run alembic downgrade -1

# Roll back all migrations to the base
uv run alembic downgrade base

# Display migration history
uv run alembic history

# Display current revision heads
uv run alembic heads

# Inspect a particular revision
uv run alembic show <revision_id>
```

Downgrades execute migration rollback code and can remove schema objects or data. Verify the migration's `downgrade()` implementation before running rollback commands.

### Initialize Alembic

For a synchronous SQLAlchemy project:

```bash
uv run alembic init alembic
```

For an asynchronous SQLAlchemy project:

```bash
uv run alembic init -t async alembic
```

Initialize Alembic only once per migration environment.

### Run a one-off migration command without opening a shell

```bash
# Generate a migration
docker exec -it fast_web \
  uv run alembic revision --autogenerate -m "add email to users"

# Apply migrations
docker exec -it fast_web \
  uv run alembic upgrade head

# Check migration status
docker exec -it fast_web \
  uv run alembic current
```

These examples assume `uv` is available in the container and the container has the correct project working directory and configuration.

### Important Docker networking note

Inside the application container, PostgreSQL should generally be addressed using the Compose service name and its internal port, for example:

```text
postgresql+psycopg://fast_user:password@db:5432/fast_db
```

Here, `db` is an example Compose service name, not necessarily the container name. Replace it with your actual service name.

From the host machine, the connection typically uses the published host port, such as `localhost:5432`, if configured.

Do not use `localhost` as the PostgreSQL hostname from the application container unless PostgreSQL is running inside that same container or the network is configured specifically for it.

---

## 6. 🔧 Troubleshooting Quick Reference

| Problem | Useful command or action |
|---|---|
| Container is not running | `docker compose ps -a` |
| Application crashes during startup | `docker compose logs -f web` |
| PostgreSQL fails to start | `docker compose logs -f db` |
| Cannot connect to PostgreSQL | Check the service hostname, port, credentials, and database readiness. |
| Need to inspect environment configuration | `docker compose config` |
| Database schema is outdated | Run `uv run alembic current`, then `uv run alembic upgrade head`. |
| Alembic cannot detect model changes | Verify `target_metadata` and import all model modules in `env.py`. |
| Migration fails | Inspect the exception, database schema, current revision, and migration operations before retrying. |
| Docker consumes too much disk space | `docker system df` |
| Need a clean development database | Use `docker compose down -v` only if deleting the project's persisted database state is intentional. |

**Core workflow:** Start services with Docker Compose, develop models in SQLAlchemy, generate and review Alembic migrations, apply migrations to PostgreSQL, and inspect logs or database state when troubleshooting. Keep migrations under version control and avoid destructive Docker cleanup commands unless the consequences are understood.