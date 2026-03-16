# GCP Cloud Run — Platform Reference

Last updated: 2026-03-04

## Known Constraints

| Constraint | Limit | Notes |
|-----------|-------|-------|
| **Request timeout** | 900s (configurable) | Set via `--timeout` in deploy script. Max is 3600s. |
| **Memory** | 2Gi (configurable) | Heavy dependencies like LibreOffice need ~512MB. Adjust based on workload. |
| **CPU** | 2 (configurable) | CPU is only allocated during request processing (default). |
| **Max instances** | 10 (configurable) | Controls cost. Scale down for dev, up for production bursts. |
| **Container startup** | Cold starts ~5-10s | First request after idle may be slow. Heavy imports add to startup. |
| **Filesystem** | Ephemeral `/tmp` only | No persistent storage. Files in `/tmp` disappear between requests. Use Drive/GCS for persistence. |
| **Concurrency** | 80 (default) | Requests per container instance. Lower if workload is CPU-heavy. |

## Deploy Pattern

```bash
# Typical deploy script flow:
# 1. Check prerequisites (gcloud, Docker, credentials)
# 2. Enable required APIs (Cloud Build, Cloud Run, Artifact Registry, Secret Manager)
# 3. Create/update secrets from local credentials/ files
# 4. Build container via gcloud builds submit (remote build, no local Docker needed)
# 5. Deploy to Cloud Run with secret mounts
```

### Credentials on Cloud Run
Credentials are mounted via Secret Manager, NOT baked into the image:

```
/app/credentials/service-account-key.json  ← Secret: service-account-key
/app/credentials/oauth_credentials.json    ← Secret: oauth-credentials
/app/credentials/gmail_token.pickle        ← Secret: gmail-token
```

Never put credentials in the Docker image or git repo.

## Docker / Container Notes

### Layer Ordering for Cache Efficiency
```dockerfile
# Install system dependencies first (cached layer)
RUN apt-get update && apt-get install -y [system-deps]

# Then language dependencies (cached unless requirements file changes)
COPY requirements.txt .
RUN pip install -r requirements.txt

# Then application code (changes most frequently)
COPY . .
```

### LibreOffice in Docker (if needed for PDF conversion)
- `apt-get install -y libreoffice` adds ~400MB to the image
- LibreOffice headless mode: `soffice --headless --convert-to pdf --outdir /tmp/pdf file.docx`
- Batch conversion (multiple files in one call) is ~2.5x faster than individual calls
- LibreOffice writes to a user profile dir — concurrent instances can conflict unless isolated

## Pre-Deploy Checklist

Before triggering a Cloud Run deploy:

1. Run local API server and test health endpoint
2. Test API with realistic payloads against local server
3. Measure response times — will they complete within the timeout?
4. Check `gcloud auth list` — is the auth token still valid?
5. Only then deploy

## Troubleshooting

### `gcloud auth` expired
```
ERROR: Reauthentication failed. cannot prompt during non-interactive execution.
```
Fix: Run `gcloud auth login` in a terminal (requires browser).

### Container fails to start
Check logs: `gcloud run logs read SERVICE_NAME --region REGION --limit 50`

Common causes:
- Missing credentials (Secret Manager mount failed)
- Python import errors (dependency not in requirements.txt)
- Port mismatch (Cloud Run expects `PORT` env var, default 8080)

### Slow cold starts
Consider:
- `--min-instances 1` to keep one instance warm (costs more)
- Lazy imports for heavy dependencies
- Smaller base image if heavy dependencies aren't needed for all endpoints

## Cost Notes

- Cloud Run bills per request + CPU/memory time
- `gcloud builds submit` bills Cloud Build minutes (~$0.003/build-minute)
- Artifact Registry bills storage (~$0.10/GB/month)
- Keep `--max-instances` low for dev/test to control costs
