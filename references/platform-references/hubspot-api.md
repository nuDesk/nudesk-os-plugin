# HubSpot API — Platform Reference

Last updated: 2026-03-04

## API Conventions

- **Base URL:** `https://api.hubapi.com`
- **Auth:** `Authorization: Bearer $HUBSPOT_ACCESS_TOKEN` (sourced from `.env`)
- **Idempotency:** Always query existing objects before creating (search lists, search properties)
- **Logging:** Log every API call — method, endpoint, HTTP status, response summary
- **Rate limit:** 100 requests per 10 seconds (Private App tier)
- **Key endpoints:**
  - Lists: `POST /crm/v3/lists/` | `POST /crm/v3/lists/search`
  - Properties: `GET/POST /crm/v3/properties/{objectType}`
  - Objects: `GET/POST /crm/v3/objects/{objectType}`
- **objectTypeId:** Contacts = `0-1`, Companies = `0-2`, Deals = `0-3`

**Environment variables:**
- Store `HUBSPOT_ACCESS_TOKEN` in `.env` file (gitignored)
- Load using: `TOKEN = os.environ.get("HUBSPOT_ACCESS_TOKEN")`
- NEVER commit tokens to git
- `.env` format: `HUBSPOT_ACCESS_TOKEN=pat-na2-xxxxx`

## Batch API Endpoints

### `POST /crm/v3/objects/contacts/batch/create`
Create contacts in batches of up to 100. Standard for new contact imports. Returns created objects with IDs.

### `POST /crm/v3/objects/contacts/batch/upsert`
Create-or-update contacts by a unique identifier (typically `email`). Use when re-importing data that may overlap with existing records. **Note:** Only works when contacts have a valid dedup key (email). Contacts without email always create new records regardless of upsert — HubSpot deduplicates on email only.

### `POST /crm/v3/objects/contacts/batch/archive`
Soft-delete (archive) contacts in batches of up to 100. **Gotcha:** Returns `204 No Content` with empty response body on success — do not attempt to parse JSON from the response. Archived contacts are recoverable from HubSpot Recycle Bin for 90 days.

### `PUT /crm/v3/lists/{listId}/memberships/remove`
Remove contacts from a static list. Request body is a plain JSON array of record IDs (NOT wrapped in an object):
```json
["12345", "67890", "11111"]
```
**Note:** This does NOT delete or archive the contacts — only removes list membership.

## Deduplication Behavior

**HubSpot deduplicates on email only.** Contacts without an email address will ALWAYS create a new record, even if firstname + lastname + company match an existing contact. This means:
- Independent imports of overlapping contact lists (same people, no emails) will create duplicates
- `batch/upsert` does not help when contacts lack email
- Prevention must happen upstream (dedup logic before import)

## Starter Plan Constraints

- 2 pipelines max
- 25 active lists max
- No email sequences (requires Sales Hub Professional)
- No workflow automation (requires Sales Hub Professional)
- No custom report dashboards (requires Sales Hub Professional)
- No custom call outcomes (requires Sales Hub Professional)

## Naming Convention

All campaign objects use the prefix `[PRODUCT] —` (em dash U+2014, not hyphen). Examples: `CRE —`, `LOE —`.

## Anti-Patterns

| Anti-Pattern | Why It Fails | Do This Instead |
|-------------|-------------|-----------------|
| Parsing JSON from batch/archive response | Returns 204 with empty body — JSON parse fails | Check status code only |
| Wrapping list membership removal in an object | API expects a plain array | Send `["id1", "id2"]` directly |
| Assuming upsert prevents duplicates without email | HubSpot deduplicates on email only | Dedup upstream before import |
| Sequential API calls when batch endpoints exist | Rate-limited at 100/10s | Use batch endpoints for bulk operations |
| Creating objects without checking existence | Duplicate records | Always search first, then create/update |
