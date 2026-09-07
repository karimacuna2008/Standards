# Deployment Standards — Cloud Build + GitHub CI/CD

Claude-facing rules for deploying to Google Cloud from GitHub via **Cloud
Build** — containerized API services on Cloud Run (§1-6, §9), desktop apps
(§8), and one-off/gated jobs like DB migrations (§10). For a project with more
than one of these deploying separately from the same repo, see the monorepo
pattern in §7.

**Always write your own `cloudbuild.yaml` and `gcloud builds triggers
create`, never Cloud Run's built-in "Continuously deploy from a repository"
feature.** That feature manages its own hidden build config and only really
fits a single build→push→deploy service — it can't express a multi-step
pipeline (upload a desktop installer, run a gated migration job, deploy to a
Cloud Run Job instead of a Service). A hand-written `cloudbuild.yaml` triggered
by a regular Cloud Build GitHub trigger covers every case in this doc with one
consistent mechanism.

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
- **`cloudbuild.yaml` needs an explicit `docker push` step between `docker
  build` and `gcloud run deploy`** (or `gcloud run jobs deploy`, §10). The
  top-level `images:` list at the end of the file only pushes to the registry
  *after every step finishes* — a deploy step that runs before that will fail
  with `Image ... not found`, because the image doesn't exist in the registry
  yet when that step executes. Three steps, always, in this order: build,
  push, deploy.

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
- **Which project hosts the secret:** always a shared-services project, never the
  app's own compute project — `prod-servicios-compartidos` for production secrets,
  `qa-servicios-compartidos` for QA/dev-tier secrets once that environment exists
  (create the project only when actually needed, not preemptively). Grant the
  consuming service's SA `roles/secretmanager.secretAccessor` on each secret
  individually (a cross-project IAM binding on the secret) — never copy the
  secret into the app's own project just to simplify IAM, and never scatter an
  app's secrets across more than one project.
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
- **`gcloud run deploy`/`gcloud run jobs deploy --set-secrets` needs the
  secret's project as a *number*, not its project ID string**, whenever the
  secret lives in a different project than the Cloud Run resource (the normal
  case per this section): `projects/<PROJECT_NUMBER>/secrets/<NAME>:latest`,
  not `projects/<project-id>/secrets/<NAME>:latest` — the string form fails
  with `is not a valid secret name`. Get the number with `gcloud projects
  describe <project-id> --format="value(projectNumber)"`. This is specific to
  the `gcloud run` deploy flags; a `cloudbuild.yaml` step that needs a secret
  **only during the build itself** (not on the deployed resource) uses a
  different, older mechanism — a top-level `availableSecrets.secretManager`
  block mapped to a step's `secretEnv` — which accepts the project ID string
  fine. Don't assume the two behave the same just because both say "Secret
  Manager".
- Sanity-check a freshly-created secret's raw bytes before trusting it:
  `gcloud secrets versions access latest --secret=<NAME> | wc -c` should match
  the expected length exactly. A secret piped through the wrong tool (e.g.
  a Windows shell redirect) can silently pick up trailing `\r\n` junk, which
  makes every comparison against it fail even though the "visible" value
  looks correct.

## 6. Trigger config, always
Every `gcloud builds triggers create github` (or `update`) needs:
- `--branch-pattern="^<branch>$"` matching the branch from §2.
- `--build-config="<piece folder>/cloudbuild.yaml"` — the file lives inside the
  piece it builds, never at the repo root when the monorepo pattern (§7)
  applies.
- A dedicated deploy service account (`--service-account=`), not the default
  Compute Engine SA — grant it exactly the roles each step needs (§5,
  `roles/run.admin` to deploy a Service or Job, `roles/artifactregistry.writer`
  to push images, `roles/iam.serviceAccountUser` **on itself** if a build step
  deploys a resource that runs as that same SA — Cloud Run's `actAs` check
  applies even when the runtime SA and the deploying SA are identical).

## 7. Monorepo — one trigger per folder, always filtered by path
When a repo ships more than one independently-deployable piece (see the
monorepo layout in `project-structure.md`), give each piece its own trigger,
each filtered to only fire on changes inside that piece:

```
gcloud builds triggers create github <trigger-name> \
  --repository=<connection> \
  --branch-pattern="^master$" \
  --build-config="<Project> - <Piece>/cloudbuild.yaml" \
  --included-files="<Project> - <Piece>/**" \
  --service-account=<deploy-sa>
```

- **`--included-files` (`includedFiles`) is not optional — a trigger created
  without it fires on every push to the branch, regardless of what changed.**
  Confirmed real bug: a monorepo trigger created without this filter rebuilt
  and republished a desktop app's installer three times in a row from pushes
  that only touched a sibling API/DB folder — harmless here (identical
  rebuild), but wasted build minutes and could just as easily republish
  something broken. Always verify the filter is live with `gcloud builds
  triggers describe <name> --format="yaml(includedFiles)"` right after
  creating it, not just when writing the create command.
- Name each trigger `deploy-<project>-<piece>` (`migrate-<project>-<piece>`
  for a gated job trigger, §10).
- If `gcloud builds triggers update` on an existing 2nd-gen GitHub-connection
  trigger rejects `--included-files` with a bare `INVALID_ARGUMENT` and no
  further detail, don't fight the `update` subcommand — export the trigger
  (`gcloud beta builds triggers export <name> --destination=file.yaml`), add
  `includedFiles: [...]` to the YAML, and `gcloud beta builds triggers import
  --source=file.yaml`. The `update github` subcommand's flag set targets the
  older 1st-gen `github.owner`/`github.name` trigger shape and can silently
  refuse valid fields on a trigger created against a 2nd-gen
  `repositoryEventConfig` connection.

## 8. Desktop app (Electron) — build, package, self-update
Deploying a desktop app means shipping a built installer to users' machines,
not a server. Pattern: **GCS bucket + electron-builder + electron-updater.**

- `electron-builder.yml`: `nsis.perMachine: false` (mandatory — `true` makes
  every auto-update prompt UAC), `publish.provider: generic`, `publish.url`
  pointing at a public path in a GCS bucket.
- Auto-update: `electron-updater` in the main process — `autoDownload: true`,
  install silently and relaunch on `update-downloaded`, never block app
  startup on an update `error`.
- Bucket: `allUsers` → `roles/storage.objectViewer` (requires disabling
  **Public Access Prevention** at the bucket level first — that's GCS's own
  per-bucket default, not an org policy, so it's safe to turn off per-bucket
  without escalating further).
- `cloudbuild.yaml` for this piece: `npm ci` → the bundler build (e.g.
  `npx vite build`) → `electron-builder` (produces the installer) → upload to
  the bucket. **The bundler build step is easy to leave out** when writing
  this by hand — without it, `electron-builder` packages a stale or missing
  build output, silently producing a corrupt installer archive that still
  "succeeds" as a build.
- Upload the release twice: once to a flat `releases/` path (the only one
  `electron-updater` reads) and once to `releases/<version>/` (an archived
  copy, for quick rollback or diffing). Pull `<version>` from `package.json`
  via `node -p` in the build step and pass it to the upload step through a
  workspace file — Cloud Build steps run in separate containers and don't
  share shell env directly.
- Verify end-to-end before trusting it: build a real installer locally
  (e.g. `npm run dist`), install it silently (`/S` on NSIS) and confirm no UAC
  prompt fires and the install path is per-user (`%LOCALAPPDATA%`, not
  `Program Files`).

## 9. API / container service on Cloud Run — recap
Section 3's Dockerfile rules and section 5's secrets rules cover this fully;
the only addition here is the build order from §3: **build → push → deploy**,
always three explicit steps in `cloudbuild.yaml`, never rely on the top-level
`images:` list to push in time for a deploy step.

## 10. Scheduled/gated jobs (DB migrations, one-off scripts) — Cloud Run Job, not a Private Pool
When a CI step needs to reach a **firewalled resource** (e.g. a database that
only whitelists one static IP) and you're tempted to reach for a Cloud Build
**Private Pool** with `NO_PUBLIC_EGRESS` peered to your VPC + Cloud NAT for the
static IP: **don't — it doesn't work for anything that also needs the public
internet** (cloning the repo, `pip install`, etc.). Cloud NAT cannot NAT
traffic from a Private Pool's workers, because they run in a Google-managed
"producer" network that's only *peered* with your VPC — Cloud NAT never
extends across that peering (confirmed against Google's own docs). A pool
like this fails even before your job's code runs, at the `git clone` of the
build's own source.

**What actually works:** separate the build (needs the public internet) from
the run (needs the static IP), using a **Cloud Run Job**:

1. Build the job's image on Cloud Build's **default pool** (public egress, no
   restriction) — `docker build` + `docker push`, same as §3/§9.
2. `gcloud run jobs deploy <job-name> --image=... --network=<vpc>
   --subnet=<subnet> --vpc-egress=all-traffic --service-account=<sa>` — Cloud
   Run's **Direct VPC Egress** genuinely attaches the job to your VPC subnet
   (unlike a Private Pool, this is not a peered network), so Cloud NAT on that
   subnet's Cloud Router *does* apply, and the job gets the same static IP a
   VM in that VPC would get.
3. `gcloud run jobs execute <job-name> --wait` to run it and block until done.

Rules for this pattern:
- Gate it behind manual approval on the trigger
  (`--require-approval` on `gcloud builds triggers create`, or
  `approvalConfig.approvalRequired: true` in the trigger's YAML) whenever the
  job can mutate production state (schema migrations, data backfills) — never
  auto-run those on every push. Approve with `gcloud beta builds approve
  <build-id> --comment=...` (regional builds need the REST API directly —
  `POST .../v1/projects/<p>/locations/<region>/builds/<id>:approve` — if
  `gcloud beta builds approve` 404s on a build you can otherwise `describe`
  with `--region`, that's why: the beta command targets the global builds API,
  not the regional one).
- **Give the DB client a generous `connect_timeout`** (60s, not the driver's
  default — e.g. PyMySQL defaults to 10s). A fresh Cloud Run Job execution is
  a cold start with no prior request to warm up the Direct VPC Egress path,
  and Google documents 30s+ cold-start latency for it; a short client-side
  timeout will fail the very first connection attempt even though the network
  path is correctly configured. A long-lived Cloud Run *Service* usually
  doesn't hit this in practice, because by the time it handles a request that
  needs the DB, an earlier request has often already warmed the same path —
  a Job never gets that warm-up.
- Once the Cloud Run Job pattern works, delete any Private Pool you created
  for the earlier approach (`gcloud builds worker-pools delete`) — it's dead
  weight with no traffic ever routing through it.

## 11. Pre-flight checklist
1. Repo is on GitHub and the target branch is pushed.
2. APIs enabled: Cloud Run, Cloud Build, Artifact Registry.
3. `Dockerfile` + `.dockerignore` present and committed.
4. Service account has the required IAM roles (§5, §6).
5. Env vars / secrets configured on the service (not in code), cross-project
   refs use the numeric project ID where required (§5).
6. Each trigger's `--included-files` verified live, not just written (§7).
7. Any job touching production state (§10) is gated behind manual approval.

---
**Version:** 1.6
**Last Updated:** 2026-08-18
