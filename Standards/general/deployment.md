# Deployment Standards — Cloud Run + GitHub CI/CD

Claude-facing rules for deploying containerized services (FastAPI and similar) to
Google Cloud Run with automatic deployment from GitHub. The build engine is
**Cloud Build**; **Cloud Run "Continuous deployment"** wraps it so each push to a
branch rebuilds and redeploys.

## 1. Execution & deploy authorization
- CI/CD is **not always used**. Depending on the project, deploys are either
  automatic (push to a branch → Cloud Build rebuilds → Cloud Run updates) or
  **manual via commands** (`gcloud run deploy`, `gcloud builds submit`, etc.).
- You **may** run deploy commands yourself through the Bash/cmd/PowerShell tool,
  as if the user had typed them — but **ask first, every time:**
  "¿Ejecuto el deploy yo o lo haces tú?" Run only if the user confirms; otherwise
  output the commands for the user to run.
- Same authorship rule as `git-workflow.md`: any git push that triggers a deploy
  must read as fully human-authored (configured identity, no `Co-Authored-By`).

## 2. Branch → environment mapping
Mirror the Git workflow (see `git-workflow.md`):

| Branch | Cloud Run service | Updates on |
|---|---|---|
| `develop` | `dev-<service-name>` | every push to `develop` |
| `main` | `<service-name>` (prod) | merge of an approved PR |

One service per environment, each with its own env vars/secrets and its own
build trigger. Never point both branches at the same service.

## 3. Container build — Dockerfile required
Prefer an explicit `Dockerfile` over buildpacks (reproducible base image, no
entrypoint guessing). Mandatory requirements:

- App **must listen on `$PORT`** (Cloud Run injects it; default 8080) and bind to
  `0.0.0.0`, not `127.0.0.1`.
- Copy `requirements.txt` and install **before** copying app code (layer cache).
- Use the shell form for `CMD` when the start command references `$PORT`, so the
  variable expands. Example: `CMD ["sh","-c","uvicorn main:app --host 0.0.0.0 --port ${PORT:-8080}"]`.
- Ship a `.dockerignore` (mirror `.gcloudignore`) so `.git/`, `.venv/`, and
  especially `.env` secrets never enter the image.
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

## 6. Pre-flight checklist
1. Repo is on GitHub and the target branch is pushed.
2. APIs enabled: Cloud Run, Cloud Build, Artifact Registry.
3. `Dockerfile` + `.dockerignore` present and committed.
4. Service account has the required IAM roles.
5. Env vars / secrets configured on the service (not in code).

---
**Version:** 1.1
**Last Updated:** 2026-06-12
