# HubSpot Conventions & Schema Reference

Last updated: 2026-03-04

> This doc captures naming conventions, property schemas, and pipeline structures for HubSpot CRM instances. Customize the IDs and values for your specific HubSpot account.

## Naming Convention

All campaign objects use the prefix `[PRODUCT] —` (em dash U+2014, not hyphen). Examples: `CRE —`, `LOE —`.

## Pipeline Structure

Define your deal pipeline stages with internal IDs. Example structure:

| Stage | Purpose |
|-------|---------|
| New Lead | Freshly created or imported |
| Qualified Lead | Meets ICP criteria |
| Application Submitted | Active application in progress |
| Under Review | With underwriting or lender |
| Approved | Approved, pending funding |
| Funded | Deal closed and funded |
| Declined | Rejected by lender |
| Withdrawn | Withdrawn by borrower |

> Replace with your actual pipeline and stage IDs from HubSpot Settings > Objects > Deals > Pipelines.

## Contact Properties (Custom)

| Property | Internal Name | Type | Description |
|----------|--------------|------|-------------|
| ICP Segment | `icp_segment` | dropdown | Ideal Customer Profile category |
| Lead Status | `hs_lead_status` | dropdown | Current lead lifecycle status |
| Contact Source | `contact_source` | dropdown | How the contact entered the system |
| Data Vendor Import Batch | `apollo_import_batch` | text | Pattern: `[SourceType]-[PartnerName]-[Date]-[Description]` |
| Status Note | `status_note` | text | Free-text status context |
| Contact Source Note | `contact_source_note` | text | Additional source context |

## Deal Properties (Custom)

| Property | Internal Name | Type | Description |
|----------|--------------|------|-------------|
| Loan Product | `loan_product` | dropdown | Product type |
| Loan Amount | `loan_amount` | number | Dollar amount |
| Lender Partner | `lender_partner` | dropdown | Lending institution |
| Referring Partner | `referring_partner` | dropdown | Referral source |
| Lender Contact | `lender_contact` | text | Contact at lending partner |
| Shared with Lender | `shared_with_lender` | dropdown | yes/no flag |
| Declined Reason | `declined_reason` | dropdown | Why the deal was declined |
| Withdrawn Reason | `withdrawn_reason` | dropdown | Why the deal was withdrawn |
| Deal Source Channel | `deal_source_channel` | dropdown | Origination channel |
| Deal Source Campaign | `deal_source_campaign` | dropdown | Specific campaign attribution |
| Deal Status Note | `deal_status_note` | text | Free-text deal context |

## Company Properties (Custom)

| Property | Internal Name | Type |
|----------|--------------|------|
| Partner Status | `partner_status` | dropdown |
| Partner Type | `partner_type` | dropdown |
| Activation Date | `activation_date` | date |

## List Management

### Dynamic Lists
Dynamic lists auto-update based on filter criteria. Common patterns:
- **Active Prospects**: ICP segment known AND lead status not disqualified AND source = target channel
- **Working Leads**: Specific segment AND status = Working
- **Needs Follow-Up**: Source = target channel AND status = Attempted AND no activity in N days
- **Meeting Booked**: Status = Deal created AND source = target channel

### Static Lists
Static lists are manually curated or imported. Use for:
- One-time import batches (with date and source in the name)
- Vendor data imports
- Campaign enrollment snapshots

## Best Practices

- Always check existence before creating properties, lists, or objects
- Use em dash (`—`, U+2014) in naming, not hyphen (`-`)
- Log every API call for auditability
- Dedup contacts upstream — HubSpot only deduplicates on email
- Keep list count under plan limits (25 active for Starter)
- **Type mismatch:** v4 associations batch read returns `toObjectId` as integer; list memberships return `recordId` as string. Always `str()` IDs before comparing across endpoints.
