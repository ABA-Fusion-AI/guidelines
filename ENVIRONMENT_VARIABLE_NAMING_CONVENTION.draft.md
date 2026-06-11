# Environment Variable Naming Convention

> A shared standard for naming environment variables across the Fusion AI monorepo.
> The goal is **predictability**: any dev should be able to guess the name of a variable
> without grepping, and we should never end up with two names for the same thing
> (e.g. `CORS_ORIGIN` vs `CORS_ORIGINS`, `VITE_AUTH_API` vs `VITE_AUTH_API_URL`).

---

## 1. Core rules

1. **`UPPER_SNAKE_CASE` only.** Letters, digits, underscores. No camelCase, no hyphens, no dots.
2. **Words are separated by single underscores.** `USE_POLLING`, never `USEPOLLING`.
3. **Spell words out in full.** `FRONTEND` not `FRONT`, `DATABASE` not `DB` (see §4 for the approved abbreviation list — anything not on it gets spelled out).
4. **Singular vs plural follows the value.** A single value is singular (`CORS_ORIGIN`); a comma/space-separated list is plural (`CORS_ORIGINS`). Pick one per concept and document it — do not ship both.
5. **No redundant suffixes.** If it is a URL, it ends in `_URL`. Don't also keep a bare version around.

---

## 2. Naming structure

Every variable follows this shape:

```
[<SCOPE>_]<SUBJECT>_<QUALIFIER>[_<TYPE>]
```

| Part        | Required | Meaning                                                  | Examples                          |
|-------------|----------|----------------------------------------------------------|-----------------------------------|
| `SCOPE`     | sometimes| Build/runtime scope prefix (see §3)                      | `VITE_`                           |
| `SUBJECT`   | yes      | The service, system, or domain the var belongs to        | `AUTH_API`, `REDIS`, `JWT`        |
| `QUALIFIER` | usually  | What aspect of the subject                               | `CONNECTION`, `ACCESS`, `ROTATION`|
| `TYPE`      | usually  | The kind of value (see §5)                               | `_URL`, `_SECRET`, `_PORT`, `_MS` |

**Examples that follow the structure:**

```
AUTH_API_URL                     # SUBJECT=AUTH_API, TYPE=URL
AUTOMATION_API_MONGODB_DB_NAME   # SUBJECT=AUTOMATION_API, QUALIFIER=MONGODB_DB, TYPE=NAME
JWT_ACCESS_EXPIRES_IN            # SUBJECT=JWT, QUALIFIER=ACCESS, TYPE=EXPIRES_IN
VITE_ORBIT_API_URL               # SCOPE=VITE, SUBJECT=ORBIT_API, TYPE=URL
KEY_ROTATION_INTERVAL_DAYS       # SUBJECT=KEY, QUALIFIER=ROTATION_INTERVAL, TYPE=DAYS
```

---

## 3. Scope prefixes

| Prefix    | When to use                                                              |
|-----------|--------------------------------------------------------------------------|
| `VITE_`   | **Required** for any var that must be exposed to a Vite frontend bundle. Vite only injects vars prefixed with `VITE_`. Never put a secret behind `VITE_` — it ships to the browser. |
| *(none)*  | Server-side / backend variables.                                         |

> If a value exists in both a backend and a frontend, they are two different variables:
> `AUTOMATION_API_URL` (backend) and `VITE_AUTOMATION_API_URL` (frontend). They may hold
> the same value but they are named, declared, and documented separately.

---

## 4. Service / subject prefixes

Use the **app or package directory name**, upper-cased, as the subject prefix. This keeps the
variable traceable back to where it's consumed.

| Directory            | Prefix              |
|----------------------|---------------------|
| `apps/auth-api`      | `AUTH_API_`         |
| `apps/automation-api`| `AUTOMATION_API_`   |
| `apps/engine`        | `ENGINE_`           |
| `apps/repo-tree`     | `REPO_TREE_`        |
| `apps/orbit-frontend`| `VITE_ORBIT_`       |
| `apps/auth-frontend` | `VITE_AUTH_`        |
| `packages/registry`  | `REGISTRY_`         |
| `nodes/*`            | `<NODE_NAME>_`      |

**Cross-cutting subjects** (used by many services) keep their own stable prefix, not a per-app one:

```
REDIS_        DATABASE_       JWT_        AWS_        CORS_
LOG_          GRPC_           KEY_        SERVICE_    NODE_ENV
```

### Approved abbreviations (the *only* ones — spell out everything else)

| Abbreviation | Means              |
|--------------|--------------------|
| `API`        | API                |
| `URL`        | URL                |
| `DB`         | database (in compound names like `DB_NAME`, `DB_ADAPTER`) |
| `WS`         | WebSocket          |
| `MS`         | milliseconds (unit suffix) |
| `ID`         | identifier         |
| `ENV`        | environment        |
| `JWT`        | JSON Web Token     |
| `GRPC`       | gRPC               |
| `CORS`       | CORS               |
| `AWS`        | AWS                |

Note: `FRONTEND`, `WEBHOOK`, `CONNECTION`, `INTERVAL`, `TIMEOUT`, `SECRET`, `REGION`,
`ORIGIN` etc. are **always spelled out**.

---

## 5. Type / unit suffixes

End every variable with a suffix that tells the reader what kind of value it holds.

| Suffix        | Value type                                  | Example                              |
|---------------|---------------------------------------------|--------------------------------------|
| `_URL`        | Full URL (with scheme)                      | `REDIS_URL`, `AUTH_API_URL`          |
| `_HOST`       | Hostname only (no scheme/port)              | `AUTOMATION_API_HOST`                |
| `_PORT`       | Port number                                 | `GRPC_PORT`, `AUTOMATION_API_PORT`   |
| `_SECRET`     | Sensitive credential — never in `VITE_`     | `JWT_SECRET`, `LICENSE_SECRET`       |
| `_KEY`        | API key / public key                        | `API_KEY`, `OPENFDA_API_KEY`         |
| `_NAME`       | A human/string identifier                   | `AUTH_SERVICE_NAME`, `DB_NAME`       |
| `_PATH`       | Filesystem path                             | `LOG_FILE_PATH`                      |
| `_ENABLED` / `_DISABLED` | Boolean feature flag             | `DISABLE_ORBIT` → prefer `ORBIT_ENABLED` |
| `_INTERVAL_MS`/`_TIMEOUT_MS` | Duration in milliseconds     | `ENGINE_GRPC_WAIT_TIMEOUT_MS`        |
| `_INTERVAL_DAYS` / `_DAYS`   | Duration in days             | `KEY_ROTATION_INTERVAL_DAYS`         |
| `_EXPIRES_IN` | Duration string (`15m`, `7d`)               | `JWT_ACCESS_EXPIRES_IN`              |
| `_MAX_*` / `_MAX`  | Upper bound                            | `MAX_FILE_SIZE`, `ENGINE_MAX_CONCURRENT_WORKFLOWS` |

### Duration rule
Always encode the **unit** in the name. A bare `POLL_INTERVAL` is ambiguous — is it ms?
seconds? Use `POLL_INTERVAL_MS`. The only exception is `_EXPIRES_IN`-style values that carry
their own unit in the string (`"15m"`).

### Boolean rule
Booleans read as a positive assertion: `<SUBJECT>_ENABLED`. Avoid `DISABLE_*` — negated flags
make `DISABLE_ORBIT=false` (double negative) confusing. Migrate `DISABLE_ORBIT` →
`ORBIT_ENABLED`, `DISABLE_PULL` → `PULL_ENABLED`.

---

## 6. Anti-patterns found in the current codebase

These exist today and should be consolidated. **Pick the bolded canonical name.**

| Variants in use                                          | Canonical                       | Why |
|----------------------------------------------------------|---------------------------------|-----|
| `CORS_ORIGIN`, `CORS_ORIGINS`                            | **`CORS_ORIGINS`**              | It's a list. Plural. |
| `CHOKIDAR_USE_POLLING`, `CHOKIDAR_USEPOLLING`            | **`CHOKIDAR_USE_POLLING`**      | Words get underscores. |
| `VITE_AUTH_API`, `VITE_AUTH_API_URL`                     | **`VITE_AUTH_API_URL`**         | A URL ends in `_URL`. |
| `VITE_AUTH_FRONTEND_URL`, `VITE_AUTH_FRONT_URL`          | **`VITE_AUTH_FRONTEND_URL`**    | Spell out `FRONTEND`. |
| `AUTH_BASE_URL`, `BASE_URL`, `VITE_API_URL`              | scope + subject + `_URL`        | Bare `BASE_URL` has no subject. |

> When you rename, do it in one PR: update the `.env.example`, the code that reads it,
> and this table. Leave a temporary fallback (`process.env.NEW ?? process.env.OLD`) only
> if a deploy can't be coordinated, and open a ticket to remove it.

---

## 7. Declaration & documentation rules

1. **Every variable must appear in the consuming app's `.env.example`** with a comment
   describing it and a safe placeholder (never a real secret).
2. **Validate at startup.** Each service parses its env through a schema (Zod or equivalent)
   and fails fast on missing/malformed values. No silent `undefined`.
3. **Group in `.env.example`** by subject, with a header comment per group.
4. **Secrets never get committed**, never get a `VITE_` prefix, and never appear in logs.

Example `.env.example` block:

```bash
# ── Auth API ──────────────────────────────────────────────
AUTH_API_URL=http://localhost:4001
AUTH_API_GRPC_URL=localhost:50051
AUTH_ISSUER=https://auth.fusion.local
AUTH_SERVICE_NAME=auth-api
AUTH_SERVICE_SECRET=changeme            # secret — do not commit real value

# ── JWT ───────────────────────────────────────────────────
JWT_SECRET=changeme                     # secret
JWT_REFRESH_SECRET=changeme             # secret
JWT_ACCESS_EXPIRES_IN=15m
JWT_REFRESH_EXPIRES_IN=7d
```

---

## 8. Quick checklist before adding a new variable

- [ ] `UPPER_SNAKE_CASE`, single underscores, full words.
- [ ] Subject prefix matches the app/package directory or an approved cross-cutting prefix.
- [ ] `VITE_` prefix **if and only if** the frontend bundle needs it — and it's not a secret.
- [ ] Type/unit suffix present (`_URL`, `_PORT`, `_MS`, `_SECRET`, …).
- [ ] Durations carry a unit; booleans are positive (`_ENABLED`).
- [ ] Not a duplicate of an existing concept (grep first — see below).
- [ ] Added to `.env.example` with a comment and to the startup schema.

```bash
# grep all referenced env vars (excluding node_modules) to check for duplicates
grep -rhoE --exclude-dir=node_modules \
  '(process\.env|import\.meta\.env)\.[A-Za-z_][A-Za-z0-9_]*' . \
  | sed -E 's/.*env\.//' | sort -u
```
