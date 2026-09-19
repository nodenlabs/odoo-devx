# Module Migration Workspace — Agent Guide

A Dockerized workspace for migrating Odoo modules between versions and validating the result by installing them.

**The migration rules are not here.** They live in the `odoo-module-migration` skill: [`../.claude/skills/odoo-module-migration/SKILL.md`](../.claude/skills/odoo-module-migration/SKILL.md). Skill-aware agents load it automatically; otherwise read it and follow it. It routes you to one reference file per version hop, so a v16 → v19 migration loads only the v16/v17/v18 hops.

This file covers only what is specific to *this workspace*: paths and the test environment.

## Layout

```
module-migration/
├── AGENTS.md            # this file
├── CLAUDE.md            # imports AGENTS.md
├── docker-compose.yaml  # Odoo + PostgreSQL, version from .env
├── Dockerfile
├── .env.example
├── src/                 # source modules — input, never modified
└── out/                 # migrated modules — output, mounted at /mnt/extra-addons
```

## Paths

- **Input**: `src/<module_name>`. Read-only — never edit a module in place.
- **Output**: `out/<module_name>`, unless the user gives another path.
- Each migrated module gets a `CHANGELOG.md` at its root.

> **Note**: `src/.gitkeep` and `out/.gitkeep` are currently *directories*, not files, so an existing source module may sit at `src/.gitkeep/<module_name>/`. Check the actual path before assuming `src/<module_name>`.

## Setup

```bash
cp .env.example .env
mkdir -p src out
```

Set `ODOO_VERSION` in `.env` to the **target** version (`19.0`, `18.0`, `15.0`, …). `POSTGRES_VERSION` defaults to `16`.

## Test environment

```bash
docker compose up -d --build

docker compose exec odoo odoo \
  -d migration_test -i <module_name> --stop-after-init --no-http

docker compose logs -f odoo
```

Use `-u <module_name>` instead of `-i` to upgrade an already-installed module. Odoo is served at <http://localhost:8069>.

Compose names the container `noden-odoo-migration-${ODOO_VERSION}` (e.g. `noden-odoo-migration-19.0`). Prefer `docker compose exec odoo` over a hardcoded container name.

`--stop-after-init` proves the module loads — schema, views, and data parse and install. It does **not** exercise runtime flows. State that limit when reporting a successful install.

## Iterating

Paste tracebacks back to the agent; it fixes only the affected files in `out/`. Re-run the install to confirm.

After each run, check what needs human review:

```bash
grep -rn 'TODO_AI' out/
```

Uncertain code is never invented — it is left as-is and flagged `# TODO_AI: [reason]`, then described in the module's `CHANGELOG.md`.
