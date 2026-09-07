# Python Cloud Services — GCP

How to integrate GCP services (Firestore, Cloud Storage, Firebase Auth, Cloud Logging) from Python, and how to configure deployment with Cloud Build.

---

## 1. Credentials & Setup

All GCP SDKs read credentials from the `GOOGLE_APPLICATION_CREDENTIALS` environment variable. Set it once; every SDK picks it up automatically.

```python
# ✅ GOOD — credentials from environment variable
import os
from google.auth import default

# SDK reads GOOGLE_APPLICATION_CREDENTIALS automatically — no code needed
# Just ensure the variable is set before running the script


# ❌ BAD — hardcoded path in code
os.environ["GOOGLE_APPLICATION_CREDENTIALS"] = "C:/Users/me/secrets/service-account.json"

# ❌ BAD — credentials file committed to the repo
# service-account.json in the project root → never do this
```

**Decision rule — Firebase Admin SDK vs direct GCP clients:**

| Situation | Use |
|---|---|
| Project uses Firebase Auth (verifying user tokens) | `firebase_admin` SDK — one init covers Auth + Firestore + Storage |
| Only Firestore, no Auth | `google-cloud-firestore` direct client |
| Only Cloud Storage, no Auth | `google-cloud-storage` direct client |
| Multiple GCP services + Auth | `firebase_admin` SDK for all |

**Firebase Admin SDK initialization (when used):**

```python
import firebase_admin
from firebase_admin import credentials

# ✅ GOOD — guard against double initialization
def init_firebase() -> None:
    try:
        firebase_admin.get_app()
    except ValueError:
        cred = credentials.ApplicationDefault()
        firebase_admin.initialize_app(cred)


# ❌ BAD — calling initialize_app() unconditionally (raises ValueError on second call)
firebase_admin.initialize_app()
firebase_admin.initialize_app()  # ValueError: The default Firebase app already exists
```

**Rules:**
- `GOOGLE_APPLICATION_CREDENTIALS` always set via environment variable — never hardcoded in code or committed to the repo.
- Firebase Admin SDK: initialize once per process using the `get_app()` guard.
- Call `init_firebase()` at the start of `main()`, before any service is used.

---

## 2. Firestore

**Getting the client:**

```python
# ✅ GOOD — Admin SDK (when firebase_admin is already initialized)
from firebase_admin import firestore

db = firestore.client()


# ✅ GOOD — direct client (when not using Firebase Auth)
from google.cloud import firestore

db = firestore.Client()


# ❌ BAD — creating a new client on every function call
def get_user(uid: str) -> dict:
    db = firestore.client()  # new client every call
    ...
```

**Read:**

```python
# ✅ GOOD — check .exists before accessing data
def get_user(uid: str) -> dict | None:
    doc_ref = db.collection("users").document(uid)
    doc = doc_ref.get()

    if not doc.exists:
        return None

    return doc.to_dict()


# ❌ BAD — assuming the document exists
def get_user(uid: str) -> dict:
    return db.collection("users").document(uid).get().to_dict()  # AttributeError if missing
```

**Write:**

```python
# set() — creates or overwrites the entire document
db.collection("users").document(uid).set({"name": "Ana", "role": "admin"})

# update() — updates only specified fields (document must exist)
db.collection("users").document(uid).update({"role": "viewer"})

# create() — creates only if the document does not exist (raises AlreadyExists otherwise)
db.collection("users").document(uid).create({"name": "Ana"})
```

**Decision rule — set vs update vs create:**

| Situation | Use |
|---|---|
| Replace the entire document (or create it if missing) | `set()` |
| Modify specific fields on an existing document | `update()` |
| Create only if the document does not already exist | `create()` |

**Query:**

```python
# ✅ GOOD — filter + limit before streaming
def get_active_users(limit: int = 100) -> list[dict]:
    docs = (
        db.collection("users")
        .where("active", "==", True)
        .limit(limit)
        .stream()
    )
    return [doc.to_dict() for doc in docs]


# ❌ BAD — streaming the entire collection without a filter or limit
def get_all_users() -> list[dict]:
    return [doc.to_dict() for doc in db.collection("users").stream()]
```

**Delete:**

```python
# ✅ GOOD — verify existence before deleting
def delete_user(uid: str) -> bool:
    doc_ref = db.collection("users").document(uid)
    if not doc_ref.get().exists:
        return False
    doc_ref.delete()
    return True
```

**Rules:**
- Always check `.exists` before calling `.to_dict()` — a missing document returns an empty snapshot, not `None`.
- Never call `.stream()` without `.limit()` on collections that can grow unbounded.
- `update()` raises `NotFound` if the document does not exist — use `set()` when the document may not exist yet.
- Create the Firestore client once at module level (or inject it as a dependency) — not inside every function.

---

## 3. Cloud Storage

**Getting the client and bucket:**

```python
import os
from google.cloud import storage

# ✅ GOOD — bucket name from environment variable
client = storage.Client()
bucket = client.bucket(os.environ["GCS_BUCKET_NAME"])


# ❌ BAD — hardcoded bucket name
bucket = client.bucket("my-production-bucket")
```

**Upload:**

```python
# ✅ GOOD — upload from local file
def upload_file(local_path: str, blob_name: str) -> None:
    blob = bucket.blob(blob_name)
    blob.upload_from_filename(local_path)


# ✅ GOOD — upload from bytes/string (when there is no local file)
def upload_content(content: str, blob_name: str) -> None:
    blob = bucket.blob(blob_name)
    blob.upload_from_string(content, content_type="text/plain")
```

**Download:**

```python
# ✅ GOOD — download to a local file
def download_file(blob_name: str, local_path: str) -> None:
    blob = bucket.blob(blob_name)
    blob.download_to_filename(local_path)


# ✅ GOOD — download as bytes (when processing in memory)
def download_content(blob_name: str) -> bytes:
    blob = bucket.blob(blob_name)
    return blob.download_as_bytes()
```

**List:**

```python
# ✅ GOOD — always use prefix to scope the listing
def list_files(folder: str) -> list[str]:
    blobs = bucket.list_blobs(prefix=f"{folder}/")
    return [blob.name for blob in blobs]


# ❌ BAD — listing the entire bucket
def list_all_files() -> list[str]:
    return [blob.name for blob in bucket.list_blobs()]
```

**Delete:**

```python
# ✅ GOOD — check existence before deleting
def delete_file(blob_name: str) -> bool:
    blob = bucket.blob(blob_name)
    if not blob.exists():
        return False
    blob.delete()
    return True
```

**Decision rule — public URL vs signed URL:**

| Situation | Use |
|---|---|
| Bucket is public and the file is accessible to anyone | Public URL: `blob.public_url` |
| Bucket is private, temporary access needed | Signed URL: `blob.generate_signed_url(expiration=timedelta(hours=1))` |
| Permanent private access from a trusted backend | Download directly with `download_*` — no URL needed |

**Blob naming convention:**

```python
# ✅ GOOD — organized with folder structure
blob_name = f"exports/{user_id}/report-2026-06.csv"
blob_name = f"uploads/images/{uuid4()}.jpg"

# ❌ BAD — flat at the bucket root
blob_name = "report.csv"
blob_name = "image.jpg"
```

**Rules:**
- Bucket name always from environment variable — never hardcoded.
- Always use `prefix` with `list_blobs()` — never list the entire bucket.
- Blob names use `folder/subfolder/file.ext` structure — never flat at the root.
- Use `upload_from_filename` for local files, `upload_from_string` for in-memory content.

---

## 4. Firebase Auth

Verify user tokens sent by the frontend and perform user management operations from the backend.

**Verify token:**

```python
from firebase_admin import auth

# ✅ GOOD — verify token and extract uid from the payload
def get_uid_from_token(id_token: str) -> str | None:
    try:
        decoded = auth.verify_id_token(id_token)
        return decoded["uid"]
    except auth.ExpiredIdTokenError:
        print("  ✗ Token expired")
        return None
    except auth.InvalidIdTokenError:
        print("  ✗ Invalid token")
        return None


# ❌ BAD — trusting the uid sent directly by the client
def get_user_data(uid: str) -> dict:  # uid comes from request body — never trust this
    ...


# ❌ BAD — logging the full token
def verify(token: str) -> None:
    logging.info(f"Verifying token: {token}")  # tokens are sensitive — log only uid
    decoded = auth.verify_id_token(token)
    logging.info(f"Token valid for uid: {decoded['uid']}")  # ✅ only the uid
```

**Token source — always from the Authorization header:**

```python
# ✅ GOOD — extract from Authorization: Bearer <token>
def get_token_from_header(authorization: str) -> str | None:
    if not authorization.startswith("Bearer "):
        return None
    return authorization.removeprefix("Bearer ")


# ❌ BAD — token from the request body
token = request.json.get("token")
```

**User management:**

```python
# Get user by uid
user = auth.get_user(uid)
print(user.email, user.display_name)

# Create user
user = auth.create_user(email="user@example.com", password="secure123")

# Delete user
auth.delete_user(uid)

# Update user
auth.update_user(uid, display_name="Ana García", disabled=False)
```

**Decision rule — verify on every request vs cache:**

| Situation | Use |
|---|---|
| Stateless API (Cloud Run, Cloud Functions) | Verify token on every request — tokens are short-lived |
| High-traffic endpoint where latency matters | Cache verification result by token hash (max 5 min — matches Firebase token expiry window) |

**Rules:**
- Always verify the token server-side — never trust `uid` or user data sent directly in the request body.
- Handle `ExpiredIdTokenError` and `InvalidIdTokenError` separately — they require different responses to the client.
- Never log the raw token — only log the resulting `uid`.
- Token always comes from `Authorization: Bearer <token>` header, never from the body.

---

## 5. Cloud Logging

**Setup:**

```python
import logging
import google.cloud.logging

# ✅ GOOD — initialize once at startup, before any getLogger() call
def setup_logging() -> None:
    client = google.cloud.logging.Client()
    client.setup_logging()  # redirects Python's logging module to Cloud Logging


# Call at the start of main(), before any other setup
def main() -> None:
    setup_logging()
    logger = logging.getLogger(__name__)
    ...


# ❌ BAD — initializing inside a function that gets called repeatedly
def process_record(record: dict) -> None:
    client = google.cloud.logging.Client()
    client.setup_logging()  # re-initializes on every call
    ...
```

**Writing logs:**

```python
import logging

logger = logging.getLogger(__name__)

# ✅ GOOD — structured metadata via json_fields (filterable in Cloud Logging Console)
logger.info("User action processed", extra={"json_fields": {"uid": uid, "action": "export", "records": 42}})
logger.warning("Quota nearing limit", extra={"json_fields": {"used": 950, "limit": 1000}})
logger.error("Firestore write failed", extra={"json_fields": {"collection": "users", "doc_id": uid}})


# ❌ BAD — print() after setup_logging() (bypasses Cloud Logging, loses structure)
print(f"Processing user {uid}")

# ❌ BAD — building the message string with all context (not filterable)
logger.info(f"User {uid} exported 42 records at {timestamp}")
```

**Log levels:**

| Level | When to use |
|---|---|
| `DEBUG` | Detailed info for local development only |
| `INFO` | Normal operations: start, completion, key state changes |
| `WARNING` | Unexpected but recoverable situation |
| `ERROR` | Operation failed — needs attention |
| `CRITICAL` | System cannot continue — immediate action required |

**Rules:**
- Call `setup_logging()` once at startup, before any `getLogger()` call.
- Always use `logging.getLogger(__name__)` — the module name becomes a label in Cloud Logging.
- Add context via `extra={"json_fields": {...}}` — makes logs filterable in the console.
- Never use `print()` for operational output after `setup_logging()` is called.
- General logging principles (what to log, message wording, sensitive data redaction) live in `observability.md`.

---

## 6. .gcloudignore

Cloud Build uploads the source directory to a GCS bucket before building. Without `.gcloudignore`, everything gets uploaded — including credentials, virtual environments, and local dev files.

```
# ✅ GOOD — explicit .gcloudignore at the project root

# Python
__pycache__/
*.pyc
*.pyo
*.pyd
.Python

# Virtual environments
venv/
.venv/
env/
.env/

# Environment and secrets — never upload these
.env
*.env
secrets/
*service-account*.json
*credentials*.json

# Development tools
.idea/
.vscode/
*.log
.DS_Store

# Tests and docs (not needed in the build)
tests/
docs/

# Local build artifacts
dist/
build/
*.egg-info/
```

```python
# ❌ BAD — no .gcloudignore file in the project
# Cloud Build uploads venv/ (hundreds of MB), .env files, and any local credential JSONs
```

**Rules:**
- Always create an explicit `.gcloudignore` — never rely on `.gitignore` as a fallback (Cloud Build uses it if `.gcloudignore` is absent, but behavior can differ).
- Credential JSON files (`*service-account*.json`, `*credentials*.json`) are always excluded.
- Virtual environment directories (`venv/`, `.venv/`) are always excluded — dependencies are installed during the build step.
- `.gcloudignore` is committed to the repo.

---

## 7. Cloud Build Deploy

Automated deploy: push to GitHub → Cloud Build trigger fires → build and deploy to Cloud Run.

**`cloudbuild.yaml` structure:**

```yaml
# ✅ GOOD — substitution variables, secrets from Secret Manager, no hardcoded values
steps:
  - name: 'gcr.io/cloud-builders/docker'
    args:
      - 'build'
      - '-t'
      - 'gcr.io/$PROJECT_ID/$_SERVICE_NAME:$COMMIT_SHA'
      - '.'

  - name: 'gcr.io/cloud-builders/docker'
    args:
      - 'push'
      - 'gcr.io/$PROJECT_ID/$_SERVICE_NAME:$COMMIT_SHA'

  - name: 'gcr.io/google.com/cloudsdktool/cloud-sdk'
    entrypoint: 'gcloud'
    args:
      - 'run'
      - 'deploy'
      - '$_SERVICE_NAME'
      - '--image=gcr.io/$PROJECT_ID/$_SERVICE_NAME:$COMMIT_SHA'
      - '--region=$_REGION'
      - '--platform=managed'

images:
  - 'gcr.io/$PROJECT_ID/$_SERVICE_NAME:$COMMIT_SHA'

substitutions:
  _SERVICE_NAME: my-service
  _REGION: us-central1
```

```yaml
# ❌ BAD — hardcoded values, secrets inline
steps:
  - name: 'gcr.io/google.com/cloudsdktool/cloud-sdk'
    env:
      - 'DB_PASSWORD=mypassword123'   # never — visible in Cloud Build logs
    args:
      - 'run'
      - 'deploy'
      - 'my-production-service'       # hardcoded service name
      - '--region=us-central1'        # hardcoded region
```

**Decision rule — where each value belongs:**

| Type of value | Where it goes |
|---|---|
| Service name, region, image tag | `substitutions` in `cloudbuild.yaml` or trigger config |
| API keys, passwords, tokens | Secret Manager — referenced as `secretEnv` in the step |
| Build-time constants (non-sensitive) | `substitutions` in `cloudbuild.yaml` |
| Runtime environment variables | Cloud Run service configuration — not in `cloudbuild.yaml` |

**Accessing Secret Manager in a build step:**

```yaml
# ✅ GOOD — secret fetched from Secret Manager, exposed as env var in the step
steps:
  - name: 'gcr.io/google.com/cloudsdktool/cloud-sdk'
    secretEnv: ['API_KEY']
    args: [...]

availableSecrets:
  secretManager:
    - versionName: projects/$PROJECT_ID/secrets/api-key/versions/latest
      env: 'API_KEY'
```

**GitHub trigger setup (one-time, via GCP Console):**

1. Cloud Build → Triggers → Connect Repository → select GitHub
2. Authenticate and select the repo
3. Create trigger: set branch filter (e.g., `^main$`), point to `cloudbuild.yaml`
4. Set substitution variables in the trigger config (overrides `cloudbuild.yaml` defaults)

This is a console configuration step — not code. The `cloudbuild.yaml` file is what lives in the repo.

**Rules:**
- `cloudbuild.yaml` is committed to the repo and contains no sensitive values.
- All secrets go to Secret Manager and are referenced via `availableSecrets.secretManager`.
- Runtime environment variables for the Cloud Run service are configured in the service settings — not passed through `cloudbuild.yaml`.
- Use `$COMMIT_SHA` as the image tag — enables rollback to any previous commit.
- The GitHub connection is configured once in the GCP Console; the trigger fires automatically on every push to the configured branch.

---

## Summary — Quick Reference

| Topic | Rule |
|---|---|
| Credentials | `GOOGLE_APPLICATION_CREDENTIALS` env var — never hardcode paths or commit credential files |
| SDK choice | Admin SDK if Firebase Auth is used; direct GCP client otherwise |
| Admin SDK init | Guard with `get_app()` — initialize once, at the start of `main()` |
| Firestore read | Always check `.exists` before `.to_dict()` |
| Firestore query | Always `.where().limit()` before `.stream()` — never stream unbounded |
| Firestore write | `set()` = create or overwrite; `update()` = partial update (doc must exist); `create()` = only if absent |
| Cloud Storage | Bucket name from env var; always `prefix` on `list_blobs()`; blob names use folder structure |
| Signed URL | Private bucket + temporary access → `generate_signed_url()`; public bucket → `public_url` |
| Firebase Auth | Verify token server-side every request; never trust uid from client; log only uid, not token |
| Cloud Logging | `setup_logging()` once at startup; `getLogger(__name__)`; use `json_fields` for metadata; no `print()` |
| .gcloudignore | Always explicit — exclude venv, .env, credential JSONs; committed to repo |
| Cloud Build | No hardcoded values in `cloudbuild.yaml`; secrets via Secret Manager; `$COMMIT_SHA` as image tag |

---

**Version:** 1.0  
**Last Updated:** 2026-06-28
