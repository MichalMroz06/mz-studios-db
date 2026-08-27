# MZ Studios Database

Database schema and migration repository for MZ Studios. The project uses PostgreSQL 17, Docker Compose for local database infrastructure, and Redgate Flyway for versioned schema migrations.

## Current state

- One versioned migration is currently available: `V001__create_user_table.sql`.
- Docker Compose starts one PostgreSQL 17 container.
- The initialization script creates two databases:
  - `mz_studios_db_dev` for development
  - `mz_studios_db_prod` for production-like use
- Flyway environments are configured for `dev` and `prod`.

## Requirements

- [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- [Redgate Flyway CLI](https://documentation.red-gate.com/flyway/reference/usage/command-line)
- Git

Verify the Flyway installation with:

```bash
flyway -v
```

## Project structure

```text
.
├── docker/
│   └── initdb/
│       └── 01_create_databases.sql # DB init file
├── migrations/ # Folder with migrations
├── docker-compose.yml # Local PostgreSQL configuration
├── flyway.toml  # Shared Flyway configuration
├── flyway.user.toml.example  # Configuration template
└── README.md
```

## Quick start

### 1. Start PostgreSQL

From the repository root, start the database container:

```bash
docker compose up -d
```

Check the container status:

```bash
docker compose ps
```

The local PostgreSQL instance is available at `localhost:5432` with these development credentials:

| Setting | Value |
| --- | --- |
| Host | `localhost` |
| Port | `5432` |
| User | `postgres` |
| Password | `postgres` |
| Development database | `mz_studios_db_dev` |
| Production database | `mz_studios_db_prod` |

These credentials are intended for local development only.

### 2. Configure Flyway

Copy the example user configuration:

```bash
copy flyway.user.toml.example flyway.user.toml
```

On macOS or Linux, use:

```bash
cp flyway.user.toml.example flyway.user.toml
```

The development environment is ready to use after the copy. Before using `prod`, replace the placeholder URL, username, and password in `flyway.user.toml` with real, securely managed connection details.

`flyway.user.toml` contains environment-specific credentials and should not be committed to source control.

### 3. Apply migrations

Run the current migrations against the development database:

```bash
flyway migrate -environment=dev
```

The command creates Flyway's history table and applies pending migration files from `migrations/`.

## Flyway commands

Replace `dev` with `prod` only when you intentionally target the production environment.

```bash
# Show migration status
flyway info -environment=dev

# Validate applied migrations and local migration files
flyway validate -environment=dev

# Apply pending migrations
flyway migrate -environment=dev

# Repair Flyway metadata after a failed or manually corrected migration
flyway repair -environment=dev
```

To see detailed diagnostic output, add `-X`:

```bash
flyway migrate -environment=dev -X
```

## Database reset

The initialization scripts in `docker/initdb/` run only when the PostgreSQL data volume is created for the first time. To remove the local database data and recreate both databases from scratch:

```bash
docker compose down -v
docker compose up -d
```

The `-v` option permanently deletes the local PostgreSQL volume. Use it only when a full reset is intended.

## Migrations

Migration files use Flyway's versioned migration naming convention:

```text
V<version>__<description>.sql
```

For example:

```text
V001__create_user_table.sql
```

Guidelines for new migrations:

- Use a new, monotonically increasing version number.
- Keep each migration focused on one logical database change.
- Test it locally against a clean database and an already migrated database.
- Do not edit a migration that has already been applied in a shared environment; add a new migration instead.
- Review destructive changes carefully before applying them to production.

### Existing migration

`V001__create_user_table.sql` currently:

- creates the `user_gender` enum with `M` and `K` values;
- creates the `users` table with identity, account, status, and timestamp fields;
- adds indexes for `username` and `email`;
- creates a trigger that updates `updated_at` whenever a user is updated.

## Stopping the database

Stop the container while keeping the data volume:

```bash
docker compose down
```

Start it again later with:

```bash
docker compose up -d
```

## Useful documentation

- [Flyway documentation](https://documentation.red-gate.com/flyway)
- [PostgreSQL documentation](https://www.postgresql.org/docs/)
- [Docker Compose documentation](https://docs.docker.com/compose/)