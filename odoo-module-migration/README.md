# noden · Module Migration Workspace

AI-assisted workspace to migrate Odoo modules (v12 → v19) with a Dockerized env to validate the result. Set `ODOO_VERSION` in `.env` to target any version (e.g. `15.0` for a v14→v15 migration, `18.0` for v17→v18).

## Layout

```
odoo-module-migration/
├── AGENTS.md                # Workspace guide for AI agents
├── CLAUDE.md                # Imports AGENTS.md
├── docker-compose.yaml      # Odoo + PostgreSQL, version from .env
├── Dockerfile
├── .env.example
├── src/                     # Source modules (you create)
└── out/                     # Migrated modules → /mnt/extra-addons (you create)
```

The migration rules are **not** in this directory. They live in the `odoo-module-migration` skill at [`../.claude/skills/odoo-module-migration/`](../.claude/skills/odoo-module-migration/), split into one reference per version hop so an agent loads only the hops it needs.

## 1. Setup

```bash
cp .env.example .env
mkdir -p src out
```

## 2. Migrate

Open your AI coding agent here. Claude Code loads `CLAUDE.md` → `AGENTS.md` automatically and picks up the `odoo-module-migration` skill. Other agents: point them at `AGENTS.md`.

> Migrate `src/my_module` from v12 to v19. Save it in `out/my_module`.

The agent copies the module, applies the rules hop by hop, writes a `CHANGELOG.md`, and flags uncertain code with `# TODO_AI:`.

## 3. Validate

```bash
docker compose up -d --build

docker compose exec odoo odoo \
  -d migration_test -i my_module --stop-after-init --no-http

docker compose logs -f odoo
```

Odoo: <http://localhost:8069>.

`--stop-after-init` proves the module loads; it does not exercise runtime flows.

## 4. Iterate

Paste tracebacks back to the agent; it fixes only the affected files in `out/`.

## Anti-hallucination

The agent never invents code. When unsure, it keeps the original and adds `# TODO_AI: [reason]`, then describes it in the module's `CHANGELOG.md`. Grep `out/` after each run:

```bash
grep -rn 'TODO_AI' out/
```

---

## About

**noden** — Odoo Engineering

Odoo engineering for complex operations. A Sentilis company.

- Website — <https://sentilis.me/noden>
- GitHub — <https://github.com/nodenlabs>
- DevX — <https://github.com/nodenlabs/devx>
- Email — <noden@sentilis.me>

Copyright © 2026 Noden Labs. See [LICENSE](../LICENSE).
