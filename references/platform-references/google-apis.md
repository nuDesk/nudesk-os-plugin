# Google APIs — Platform Reference

Last updated: 2026-03-04

## Authentication Patterns

### Service Account (server-to-server)
Used for: Sheets, Drive, Docs — any backend/Cloud Run service.

```python
from google.oauth2 import service_account
from googleapiclient.discovery import build

credentials = service_account.Credentials.from_service_account_file(
    "credentials/service-account-key.json",
    scopes=["https://www.googleapis.com/auth/drive"],
)
service = build("drive", "v3", credentials=credentials)
```

**Credentials are thread-safe.** Create once, share across threads.
**Service objects are NOT thread-safe.** Create one per thread using shared credentials.

```python
import threading

_credentials = None
_thread_local = threading.local()

def get_credentials():
    global _credentials
    if _credentials is None:
        _credentials = service_account.Credentials.from_service_account_file(...)
    return _credentials

def get_drive_service():
    if not hasattr(_thread_local, "service") or _thread_local.service is None:
        _thread_local.service = build("drive", "v3", credentials=get_credentials())
    return _thread_local.service
```

### OAuth (user-context)
Used for: Gmail (sending as a user). Requires OAuth credentials + token.

Token refresh is automatic via the google-auth library, but tokens can expire permanently if the OAuth app consent is revoked or the refresh token is invalidated. Store tokens securely (Secret Manager on Cloud Run, gitignored locally).

## Google Sheets

### Rate Limiting
Sequential reads cause **exponential delay** — 1.6s for the first, escalating to 6+ minutes for multiple tabs.

**Always use `batchGet` to read multiple tabs in one API call:**

```python
result = sheets_service.spreadsheets().values().batchGet(
    spreadsheetId=SHEET_ID,
    ranges=["Tab1!A:Z", "Tab2!A:Z", "Tab3!A:Z"],
).execute()
```

This is a single API call regardless of how many ranges you request.

### Write Limits
- 60 requests per minute per user per project (default)
- Use `batchUpdate` for multiple write operations
- For audit logs with multiple tabs, batch all writes together

## Google Drive

### Shared Drive Files
All API calls involving Shared Drive files MUST include `supportsAllDrives=True`:

```python
service.files().create(
    body=file_metadata,
    media_body=media,
    fields="id, name",
    supportsAllDrives=True,  # Required for Shared Drives
).execute()
```

Without this flag, files in Shared Drives return 404 or permission errors.

### File Uploads
Use `MediaFileUpload` with `resumable=True` for reliability:

```python
from googleapiclient.http import MediaFileUpload

media = MediaFileUpload(
    file_path,
    mimetype="application/pdf",
    resumable=True,
)
```

### Google Docs Export
Export Google Docs as .docx for local manipulation with python-docx:

```python
content = drive_service.files().export_media(
    fileId=DOC_ID,
    mimeType="application/vnd.openxmlformats-officedocument.wordprocessingml.document",
).execute()
```

Cache the template locally — no need to re-download on every call.

### Parallel Uploads
Drive uploads are I/O-bound and parallelize well. Use `ThreadPoolExecutor` with thread-local services:

```python
from concurrent.futures import ThreadPoolExecutor

with ThreadPoolExecutor(max_workers=4) as executor:
    results = list(executor.map(upload_one, items))
```

Each thread needs its own `build("drive", ...)` service (NOT thread-safe), but can share credentials.

## API Enablement

APIs must be explicitly enabled in GCP Console before use. Common ones:
- Google Sheets API
- Google Drive API
- Gmail API
- Google Docs API

If a service account gets permission errors on an API that should work, check Console > APIs & Services > Enabled APIs first.

## Scopes

Use the narrowest scope that works:

| Scope | Access |
|-------|--------|
| `drive.readonly` | Read files, list folders |
| `drive` | Full read/write/delete |
| `spreadsheets.readonly` | Read sheets |
| `spreadsheets` | Read/write sheets |
| `gmail.compose` | Create drafts, send |
| `gmail.readonly` | Read messages |

A service account's scope is the intersection of what's requested in code AND what's granted via domain-wide delegation or resource sharing. If something fails, check both.

## Anti-Patterns

| Anti-Pattern | Why It Fails | Do This Instead |
|-------------|-------------|-----------------|
| Sequential Sheets reads | Rate limiting causes exponential delays | `batchGet` for reads, `batchUpdate` for writes |
| One Drive service shared across threads | Not thread-safe — concurrent calls corrupt state | Thread-local services with shared credentials |
| Creating new credentials per API call | Wasteful — credential creation is slow | Cache credentials at module level |
| Missing `supportsAllDrives=True` | 404s on Shared Drive files | Always include on Shared Drive operations |
| Hardcoded scopes broader than needed | Security risk, compliance concern | Use narrowest scope that works |
| Assuming APIs are enabled | Silent failures or unclear errors | Verify in GCP Console first |
