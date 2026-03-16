# Apollo.io API — Platform Reference

Last updated: 2026-03-04

> Apollo's API docs are incomplete; many write endpoints are undocumented. This doc captures hard-won learnings.

## Authentication (Critical)

**Base URL:** `https://api.apollo.io`
**API Version:** `/api/v1/` (stable; v2 exists but is less documented — use v1 by default)

```python
HEADERS = {
    "Content-Type": "application/json",
    "X-Api-Key": API_KEY,
    "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36",
    "Cache-Control": "no-cache"
}
```

**The `User-Agent` and `Cache-Control` headers are REQUIRED for PUT/POST operations.** This is undocumented. Without them, Apollo returns `{"error": "Api key required"}` even with a valid API key. This is a Cloudflare bypass quirk — include these headers on every request to avoid mysterious auth failures.

## Endpoints Reference

| Endpoint | Method | Status | Notes |
|----------|--------|--------|-------|
| `/api/v1/emailer_campaigns` | GET | Documented | List sequences (paginated) |
| `/api/v1/emailer_campaigns` | POST | Documented | Create sequence |
| `/api/v1/emailer_campaigns/{id}` | GET | Documented | Fetch sequence + templates + steps + touches |
| `/api/v1/emailer_campaigns/{id}` | PUT | **Undocumented** | Update name, active status |
| `/api/v1/emailer_steps` | POST | Documented | Create step within sequence |
| `/api/v1/emailer_steps/{id}` | PUT | **Undocumented** | Update timing (wait_time, wait_mode) |
| `/api/v1/emailer_templates/{id}` | PUT | **Undocumented** | Update template content |
| `/api/v1/contacts` | POST | Documented | Create single contact |
| `/api/v1/contacts/search` | POST | Documented | Search contacts with filters |
| `/api/v1/contacts/{id}` | DELETE | Documented | Delete contact by ID |

## Key Quirks & Gotchas

**1. Template updates: `body_html` auto-generates `body_text`.**
When you PUT to update `body_html`, Apollo auto-generates `body_text` from the HTML. Do NOT send `body_text` manually — send only `body_html` and let Apollo handle conversion.

**2. HTML string matching: text nodes vs attributes.**
When updating template content, the same string can appear in both an HTML text node and an `<img alt="...">` attribute. Always target the specific context:
```python
# WRONG — matches alt attribute too
idx = body_html.find("Jane Doe")

# CORRECT — targets text node specifically
idx = body_html.find("Jane Doe<br>")
```

**3. Step timing is cumulative, not incremental.**
`wait_time` values represent incremental days from the *previous* step:
- Step 1: `wait_time=0` → sends Day 0
- Step 2: `wait_time=3` → sends Day 3 (0+3)
- Step 3: `wait_time=4` → sends Day 7 (3+4)
- Step 4: `wait_time=5` → sends Day 12 (7+5)

Always document cumulative day numbers in comments to avoid confusion.

**4. Step-to-template mapping uses `emailer_touches`.**
Steps don't have a direct `template_id` field. The mapping is:
- `emailer_touches[i].emailer_step_id` → `emailer_steps[j].id`
- `emailer_touches[i].emailer_template_id` → `emailer_templates[k].id`

**5. Labels auto-create on contact POST.**
If you POST a contact with `label_names` that don't exist, Apollo creates them automatically. No pre-creation step needed.

**6. Bulk create is unreliable on Starter tier.**
Use single-create in a loop with `time.sleep(0.5)` between requests (~2 contacts/sec). Build resume support via CLI argument (e.g., `python import.py 250` to skip first 250).

**7. Contact field mapping:**
- Phone → `direct_phone` (NOT `phone`)
- Company → `organization_name` (NOT `company`)

## Rate Limiting

| Operation | Safe Rate | On 429 |
|-----------|-----------|--------|
| Contact creation | ~2/sec (`sleep(0.5)`) | Wait 60s, retry |
| Contact deletion | ~1/sec (`sleep(1)`) | Wait 10 min (600s) — hourly bucket |
| Template updates | ~2/sec (`sleep(0.5)`) | Wait 60s, retry |
| Search operations | ~1/sec (`sleep(1)`) | Rarely rate-limited |

**Lesson learned:** On 429, wait longer (10 min) rather than retry aggressively. Apollo enforces hourly request buckets.

## Contact Import Workflow

1. Export from CRM to CSV
2. Create contacts one-by-one via POST (NOT bulk create on Starter tier)
3. Include `label_names` for tracking (e.g., `["Campaign-Date-Source"]`)
4. Rate limit: `time.sleep(0.5)` between creates
5. Build resume support: accept start index as CLI argument
6. Log progress every 50 contacts

## Deduplication Workflow

Apollo imports can create duplicates. Proven cleanup workflow:

1. **Detect:** Normalize names (`first + last + company` key), group, flag groups with count > 1
2. **Score:** Rank records by data quality: `email(+2) + phone(+1)` — keep highest score
3. **Review:** Export duplicate groups to CSV before deleting
4. **Delete:** Remove lower-scored duplicates via API with 1sec rate limiting

## Verification Pattern

After every create/update, verify the response — don't assume HTTP 200 means success:

```python
saved = result.get("emailer_template", {})
saved_html = saved.get("body_html", "")

checks = {
    "Sign-off present": "Program Manager" in saved_html,
    "Meeting link": "Schedule a meeting" in saved_html,
}
for label, ok in checks.items():
    print(f"  {label}: {'OK' if ok else 'ISSUE'}")
```

## Sequence Lifecycle

1. Create sequence with `active: False` (sandbox mode)
2. Test with internal contacts
3. Rename from test name to production name
4. Update step timing from sandbox (all Day 0) to production schedule
5. Set `active: True` when ready for live enrollment
