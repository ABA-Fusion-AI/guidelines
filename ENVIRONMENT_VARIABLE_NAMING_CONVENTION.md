# Environment Variable Naming

One rule set so every variable is easy to guess, and we never name the same thing
twice.

**Shape:** `SUBJECT_[QUALIFIER_]TYPE`, in capitals. Only the SUBJECT is required;
add the rest when they help. Examples: `AUTH_API_URL`, `JWT_REFRESH_SECRET`,
`REDIS_URL`.

## Rules

1. **Capitals and `_` only.** Use letters, numbers, and `_`. Do not start with a
   number. (POSIX reserves lower case for apps, and dashes or dots break in
   shells.) Good: `AUTOMATION_API_PORT`. Bad: `automation-api.port`.

2. **Start with the subject.** Use the app or package that reads it:
   `ENGINE_...`, `AUTH_API_...`, `REPO_TREE_...`.
   A thing shared by many services uses the topic, not one app's name:
   `REDIS_URL`, `DATABASE_URL`, `JWT_SECRET`, `CORS_ORIGINS`, `SERVICE_...`.
   When an app and a shared topic overlap, the app name is that service's own
   config; the topic is what everyone shares — `AUTH_API_URL` is the auth-api
   service's address, while `AUTH_ISSUER` is the identity every service trusts.

3. **End with the value type — when it has one.** `_URL`, `_HOST`, `_PORT`,
   `_SECRET`, `_KEY`, `_PATH`, `_FILE`, `_NAME`. Examples: `LOG_FILE_PATH`,
   `SERVICE_TRUST_JWKS_FILE`. Plain nouns like `_ISSUER` or `_ORIGINS` need no
   suffix.

4. **Frontend variables start with `VITE_`.** Only those reach the browser, so
   **never put a secret in a `VITE_` variable.** Server and browser copies are
   different names: `AUTOMATION_API_URL` and `VITE_AUTOMATION_API_URL`.

5. **Write words in full.** Use a short form only when nobody ever writes it out:
   `API`, `URL`, `ID`, `JWT`, `CORS`. Write `FRONTEND` (not `FRONT`) and
   `DATABASE` (not `DB`).

6. **Be exact about the value:**
   - Time carries its unit: `_MS`, `_DAYS`, or a string like `JWT_ACCESS_EXPIRES_IN=15m`.
   - Booleans say yes, not no: `ORBIT_ENABLED=true`, not `DISABLE_ORBIT`.
   - A list is plural: `CORS_ORIGINS`, `SERVICE_TRUST_SUBJECTS`.

7. **One name per thing.** The name is the same in dev, staging, and prod — only
   the value changes. No `DEV_` / `PROD_` prefixes. If two names exist for one
   thing, keep the correct one and delete the other. To find duplicates:
   ```bash
   grep -rhoE --exclude-dir=node_modules \
     '(process\.env|import\.meta\.env)\.[A-Za-z_][A-Za-z0-9_]*' . \
     | sed -E 's/.*env\.//' | sort -u
   ```

## Bad → Good

| Bad                  | Good                     | Why                            |
|----------------------|--------------------------|--------------------------------|
| `CORS_ORIGIN`        | `CORS_ORIGINS`           | it is a list                   |
| `VITE_AUTH_API`      | `VITE_AUTH_API_URL`      | a URL ends in `_URL`           |
| `VITE_AUTH_FRONT_URL`| `VITE_AUTH_FRONTEND_URL` | write `FRONTEND` in full       |
| `DISABLE_ORBIT`      | `ORBIT_ENABLED`          | say yes, not no                |
| `DB_NAME`            | `DATABASE_NAME`          | spell out `DATABASE`           |
| `PROD_DATABASE_URL`  | `DATABASE_URL`           | same name in every environment |

## Secrets

A secret is anything that grants access if leaked: a password, API key, token,
signing key, or database password.

> [!WARNING]
> A secret must **never**:
> - **be committed to git.** Keep the real value only in `.env` (which is
>   git-ignored). Commit a *fake* value in `.env.example`. Git history is
>   permanent — a committed secret stays readable forever and must be rotated.
> - **use a `VITE_` name.** Vite copies every `VITE_*` value into the browser
>   bundle, so it becomes public to every visitor.
>
> Rule of thumb: a secret must be readable by **neither** someone browsing the
> repo **nor** someone browsing the website.

## Declaring

> [!NOTE]
> Put every variable in the app's `.env.example` with a short comment and a fake
> value. The app checks its variables at startup and stops if one is missing.
