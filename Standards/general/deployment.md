# Deployment Standards — Cloud Run + GitHub CI/CD

Claude-facing rules for deploying containerized services (FastAPI and similar) to
Google Cloud Run with automatic deployment from GitHub. The build engine is
**Cloud Build**; **Cloud Run "Continuous deployment"** wraps it so each push to a
branch rebuilds and redeploys.

## 1. Execution & deploy authorization
- Every project that connects a repo to Cloud Run uses **CI/CD, always** — no
  manual deploys once it's set up. A **Cloud Build trigger** fires on every push
  to the relevant branch, builds the image, and updates the Cloud Run service.
  Never use GitHub Actions for this — Cloud Build is the only build engine.
- You **may** run the one-time setup commands yourself (`gcloud builds triggers
  create`, `gcloud run deploy` for the first manual creation, etc.) through the
  Bash/cmd/PowerShell tool, as if the user had typed them — but **ask first,
  every time:** "¿Ejecuto el setup yo o lo haces tú?" Run only if the user
  confirms; otherwise output the commands for the user to run.
- Same authorship rule as `git-workflow.md`: any git push that triggers a deploy
  must read as fully human-authored (configured identity, no `Co-Authored-By`).

## 2. Branch → environment mapping
Mirror the branch model decided in `git-workflow.md`:

**Two-branch model** (`master` + `develop`):

| Branch | Cloud Run service | Updates on |
|---|---|---|
| `develop` | `<service-name>-dev` | every push to `develop` |
| `master` | `<service-name>-prod` | every push to `master` |

**Single-branch model** (`master` only):

| Branch | Cloud Run service | Updates on |
|---|---|---|
| `master` | `<service-name>` (no suffix) | every push to `master` |

One service per environment, each with its own env vars/secrets and its own
Cloud Build trigger. Never point both branches at the same service.

**Service naming:** propose a base name similar to the project name and
**ask the user to confirm it before creating any trigger** — e.g. "¿Te parece
bien `<service-name>` como nombre base?" Append `-dev` / `-prod` only when the
two-branch model applies; the single-branch model uses the base name as-is.

## 3. Container build — Dockerfile required
Prefer an explicit `Dockerfile` over buildpacks (reproducible base image, no
entrypoint guessing). Mandatory requirements:

- App **must listen on `$PORT`** (Cloud Run injects it; default 8080) and bind to
  `0.0.0.0`, not `127.0.0.1`.
- Copy `requirements.txt` and install **before** copying app code (layer cache).
- Use the shell form for `CMD` when the start command references `$PORT`, so the
  variable expands. Example: `CMD ["sh","-c","uvicorn main:app --host 0.0.0.0 --port ${PORT:-8080}"]`.
- Ship a `.dockerignore` (mirror `.gcloudignore`) so `.git/`, `.venv/`,
  `.env` secrets, `CLAUDE.md`, and `docs/superpowers/` never enter the image
  (same exclusions as `.gitignore` per `git-workflow.md` §7).
- Pin every dependency; remove unused ones to keep builds fast and the surface small.

## 4. Cloud Run service configuration
Recommended baseline for an internal API on the free tier:

| Setting | Value | Why |
|---|---|---|
| Authentication | Require authentication (prefer) | API is internal; avoid public access unless needed |
| Billing | Request-based | Pay only while serving requests |
| Min instances | 0 | Free tier; accept cold starts (set 1 only if latency-critical) |
| Ingress | All / Internal | Internal if only other GCP services call it |
| Container port | `8080` | Or leave blank to use the image's `$PORT` |
| Memory / CPU | 512 MiB / 1 | Raise only if profiling shows need |
| Request timeout | Match the slowest endpoint | Long jobs (e.g. media processing) may need > 300s |

## 5. Credentials & secrets
- Cloud Run runs as a **service account**; the Google client libraries pick it up
  via Application Default Credentials. **Do not** bake key files into the image.
- Grant that service account only the roles it needs (Firestore, Drive/Sheets, etc.).
- App secrets (e.g. API keys) go in **Secret Manager**, mounted as env vars in the
  service — never in the repo, never in the Dockerfile.
- **Naming when a secret has multiple dataset/environment variants** (e.g. a QA
  database vs. a production database): suffix the secret name with the variant
  (`_QA`, `_PROD`, etc.), and always mount it on Cloud Run under an env var with
  the **exact same name** as the secret — `--set-secrets
  "MYSQL_HOST_X_QA=MYSQL_HOST_X_QA:latest"`, never a renamed env var on one side.
  Each service/branch only gets the secrets for the variant it actually uses.
- It's fine to create and mount a secret on a service **before the code reads
  it** (e.g. staging `_PROD` credentials on a service that's still only using
  `_QA`) — as long as the app's settings loader ignores unknown env vars
  (e.g. `pydantic-settings` with `extra="ignore"`), the unused secret is inert
  and won't break startup. This is a valid way to have credentials ready ahead
  of the code that will consume them.

## 6. Pre-flight checklist
1. Repo is on GitHub and the target branch is pushed.
2. APIs enabled: Cloud Run, Cloud Build, Artifact Registry.
3. `Dockerfile` + `.dockerignore` present and committed.
4. Service account has the required IAM roles.
5. Env vars / secrets configured on the service (not in code).

---
**Version:** 1.4
**Last Updated:** 2026-06-17
