# Control-Action Map

Maps every nuDesk SOC 2 control (AC-01 through AI-08) to its enforcement mechanism, evidence source, and responsible system.

**Categories:**
- **A — Automatable** (28 controls): Hooks, scheduled tasks, auto-evidence via Asana
- **B — Semi-Automated** (31 controls): Human-triggered commands with Asana workflow execution
- **C — Policy-Only** (32 controls): Scheduled review reminders + acknowledgment tracking

---

## Access Control (AC-01 through AC-14)

| Control | Statement | Cat | Enforcement | Evidence Source | Command/Hook | Asana Project | Cadence |
|---------|-----------|-----|-------------|----------------|-------------|---------------|---------|
| AC-01 | Least privilege via RBAC | B | Permission reviews | IAM config exports | `/compliance-status` | — | Ongoing |
| AC-02 | MFA for privileged access | A | Vanta automated test | Vanta test results | — | — | Ongoing |
| AC-03 | Encrypted remote connections | A | Vanta automated test | TLS config scan | — | — | Ongoing |
| AC-04 | Unique user IDs, no sharing | C | Policy acknowledgment | Vanta acknowledgments | — | — | Ongoing |
| AC-05 | Separate admin/normal accounts | B | Access review | IAM audit | `/compliance-status` | — | Ongoing |
| AC-06 | Documented registration/deregistration | B | Onboarding workflow | Asana + Vanta | — | — | Per event |
| AC-07 | No prod access before HR completion | C | Onboarding workflow | HR system records | — | — | Per event |
| AC-08 | Permission changes documented 1yr | A | Vanta automated test | IAM change logs | — | — | Ongoing |
| AC-09 | Quarterly access reviews | B | Scheduled review | Vanta access review | `/compliance-status` | — | Quarterly |
| AC-10 | Access removed within 5 days of termination | B | Offboarding workflow | Vanta + Asana | — | — | Per event |
| AC-11 | Password complexity + lockout | A | Vanta automated test | IdP config | — | — | Ongoing |
| AC-12 | Remove default accounts | B | Deploy checklist | Change Log subtask | `/evidence-collect` | Change Log | Per deploy |
| AC-13 | Source code access restricted/logged | A | Git access controls | Git access logs | `/evidence-collect` | Change Log | Ongoing |
| AC-14 | Privileged utility programs restricted | B | Permission reviews | Access logs | `/compliance-status` | — | Ongoing |

## Asset Management (AM-01 through AM-06)

| Control | Statement | Cat | Enforcement | Evidence Source | Command/Hook | Asana Project | Cadence |
|---------|-----------|-----|-------------|----------------|-------------|---------------|---------|
| AM-01 | Asset inventory with ownership | B | Scheduled review | Asset inventory doc | `/compliance-report` | — | Ongoing |
| AM-02 | Privileged access to sensitive assets | B | Access review | IAM config | `/compliance-status` | — | Ongoing |
| AM-03 | BYOD: screen lock, encryption, MDM | A | Vanta automated test | MDM policy status | — | — | Ongoing |
| AM-04 | Report loss/theft immediately | C | Policy + training | Incident reports | `/incident-log` | Incident Response | Per event |
| AM-05 | Secure disposal per NIST 800-88 | C | Policy | Disposal certificates | — | — | Per event |
| AM-06 | Assets returned on termination | C | Offboarding workflow | Offboarding checklist | — | — | Per event |

## Business Continuity / DR (BC-01 through BC-03)

| Control | Statement | Cat | Enforcement | Evidence Source | Command/Hook | Asana Project | Cadence |
|---------|-----------|-----|-------------|----------------|-------------|---------------|---------|
| BC-01 | Annual DR test with backup restore | B | Scheduled task | DR test report | `/evidence-collect` | — | Annual |
| BC-02 | Executive notification during disaster | C | Incident response | Communication logs | `/incident-log` | Incident Response | Per event |
| BC-03 | RTOs and RPOs defined | C | Annual review | BC/DR plan doc | `/compliance-report` | — | Annual |

## Code of Conduct (CC-01 through CC-02)

| Control | Statement | Cat | Enforcement | Evidence Source | Command/Hook | Asana Project | Cadence |
|---------|-----------|-----|-------------|----------------|-------------|---------------|---------|
| CC-01 | Weapons prohibition | C | Policy | Policy acknowledgment | — | — | Ongoing |
| CC-02 | Report unacceptable behavior | C | Policy + training | Vanta acknowledgments | — | — | Per event |

## Cryptography (CR-01 through CR-04)

| Control | Statement | Cat | Enforcement | Evidence Source | Command/Hook | Asana Project | Cadence |
|---------|-----------|-----|-------------|----------------|-------------|---------------|---------|
| CR-01 | AES-256 at rest, TLS B+ in transit | A | Config scan | Infrastructure config | `/security-check` | Change Log | Ongoing |
| CR-02 | Password hashing (bcrypt/etc.) | A | Code review | Code scan results | `/security-check` | — | Ongoing |
| CR-03 | Web certs RSA 2048+, 1yr max | A | Vanta automated test | Certificate scan | — | — | Annual |
| CR-04 | No confidential data on unencrypted media | C | Policy + training | Policy acknowledgment | — | — | Ongoing |

## Data Management (DM-01 through DM-05)

| Control | Statement | Cat | Enforcement | Evidence Source | Command/Hook | Asana Project | Cadence |
|---------|-----------|-----|-------------|----------------|-------------|---------------|---------|
| DM-01 | Three-tier data classification | C | Policy | Classification guide | — | — | Ongoing |
| DM-02 | Confidential data encrypted, no personal devices | A | PII scan hook + Vanta test | Config scan + hook logs | PII hook | — | Ongoing |
| DM-03 | PII deleted when no longer needed | B | Data retention review | Deletion records | `/compliance-report` | — | Per event |
| DM-04 | Customer data deleted within 60 days of termination | B | Offboarding workflow | Deletion confirmation | — | — | Per event |
| DM-05 | Annual data retention review | B | Scheduled review | Review minutes | `/compliance-report` | — | Annual |

## Human Resource Security (HR-01 through HR-05)

| Control | Statement | Cat | Enforcement | Evidence Source | Command/Hook | Asana Project | Cadence |
|---------|-----------|-----|-------------|----------------|-------------|---------------|---------|
| HR-01 | Background checks | C | HR process | Background check records | — | — | Per hire |
| HR-02 | NDA/confidentiality signed at hire | C | Vanta acknowledgment | Signed documents | — | — | Per hire |
| HR-03 | Annual security training | B | Scheduled task + Vanta | Training completion | — | — | Annual |
| HR-04 | Annual review includes policy adherence | C | HR process | Review records | — | — | Annual |
| HR-05 | Access revoked on termination | B | Offboarding workflow | Vanta + Asana | — | — | Per event |

## Incident Response (IR-01 through IR-05)

| Control | Statement | Cat | Enforcement | Evidence Source | Command/Hook | Asana Project | Cadence |
|---------|-----------|-----|-------------|----------------|-------------|---------------|---------|
| IR-01 | Incidents reported immediately | B | `/incident-log` | Asana incident tasks | `/incident-log` | Incident Response | Per event |
| IR-02 | Severity classification (P0-P3) | A | `/incident-log` auto-classifies | Severity custom field | `/incident-log` | Incident Response | Per event |
| IR-03 | Six-phase response lifecycle | A | Subtask template | Phase completion status | `/incident-log` | Incident Response | Per event |
| IR-04 | HIPAA breach procedures | B | `/incident-log` HIPAA check | HIPAA subtasks | `/incident-log` | Incident Response | Per event |
| IR-05 | External comms require legal/exec approval | C | Policy | Approval records | — | Incident Response | Per event |

## Information Security (IS-01 through IS-06)

| Control | Statement | Cat | Enforcement | Evidence Source | Command/Hook | Asana Project | Cadence |
|---------|-----------|-----|-------------|----------------|-------------|---------------|---------|
| IS-01 | Report incidents to security lead | C | Policy + training | Incident reports | `/incident-log` | — | Per event |
| IS-02 | Device lock + 5min auto-lock | A | Vanta automated test / MDM | MDM compliance | — | — | Ongoing |
| IS-03 | VPN for confidential data on public WiFi | C | Policy | VPN config evidence | — | — | Ongoing |
| IS-04 | Anti-virus with auto-updates | A | Vanta automated test | AV status | — | — | Ongoing |
| IS-05 | Anonymous whistleblower reporting | C | Policy | Google Form exists | — | — | Ongoing |
| IS-06 | Roles and responsibilities defined | C | Annual review | Org chart + role definitions | — | — | Annual |

## Operations Security (OS-01 through OS-15)

| Control | Statement | Cat | Enforcement | Evidence Source | Command/Hook | Asana Project | Cadence |
|---------|-----------|-----|-------------|----------------|-------------|---------------|---------|
| OS-01 | Changes tested, reviewed, approved before deploy | A | Pre-deploy hook + Change Log | Change Log subtask checklist | Pre-deploy hook | Change Log | Per change |
| OS-02 | Emergency changes: retrospective review | B | Change Log template | Emergency change records | `/evidence-collect` | Change Log | Per event |
| OS-03 | Separate dev/staging/prod | A | Infrastructure config | Environment config | `/security-check` | — | Ongoing |
| OS-04 | No confidential data in non-prod | A | PII scan hook | Hook scan logs | PII hook | — | Ongoing |
| OS-05 | Anti-malware on endpoints | A | Vanta automated test | Endpoint status | — | — | Ongoing |
| OS-06 | Daily backups, separate region, annual test | B | Vanta test + scheduled review | Backup config + test report | — | — | Daily/Annual |
| OS-07 | Prod logs retained 30 days | A | Cloud provider config | Log retention settings | `/evidence-collect` | — | Ongoing |
| OS-08 | Quarterly vuln scans | B | Scheduled task | Scan reports | `/compliance-status` | — | Quarterly |
| OS-09 | Vuln remediation SLAs (30/90/180 days) | B | Vanta tracking | Remediation records | `/compliance-status` | — | Per finding |
| OS-10 | Annual network access rule review | B | Scheduled review | Firewall/security group config | `/evidence-collect` | — | Annual |
| OS-11 | Remote session timeout 2 hours | A | Config setting | Session config | — | — | Ongoing |
| OS-12 | Authorized container base images only | A | Dockerfile scan | Image scan results | `/security-check` | Change Log | Per build |
| OS-13 | Data masking for PII | B | Code review + PII hook | Masking implementation | PII hook | — | Ongoing |
| OS-14 | Annual network security assessment | B | Scheduled task | Assessment report | `/compliance-report` | — | Annual |
| OS-15 | File integrity monitoring with alerting | A | Cloud provider config | FIM alerts config | — | — | Ongoing |

## Physical Security (PS-01 through PS-04)

| Control | Statement | Cat | Enforcement | Evidence Source | Command/Hook | Asana Project | Cadence |
|---------|-----------|-----|-------------|----------------|-------------|---------------|---------|
| PS-01 | Physical security perimeter | C | Facility management | Physical security docs | — | — | Ongoing |
| PS-02 | Secure area access controls | C | Facility management | Access logs, camera footage | — | — | Ongoing |
| PS-03 | Visitor escort + sign-in | C | Policy | Visitor logs | — | — | Per visit |
| PS-04 | Fire/climate/backup power | C | Facility management | Maintenance records | — | — | Ongoing |

## Risk Management (RM-01 through RM-03)

| Control | Statement | Cat | Enforcement | Evidence Source | Command/Hook | Asana Project | Cadence |
|---------|-----------|-----|-------------|----------------|-------------|---------------|---------|
| RM-01 | Annual risk assessment | B | Scheduled task | Risk assessment report | `/compliance-report` | Risk Register | Annual |
| RM-02 | Risk Register maintained | A | Asana Risk Register project | Active risk tasks | `/compliance-status` | Risk Register | Ongoing |
| RM-03 | Risk reports to leadership | B | `/compliance-report` | Report deliveries | `/compliance-report` | — | Periodic |

## Secure Development (SD-01 through SD-06)

| Control | Statement | Cat | Enforcement | Evidence Source | Command/Hook | Asana Project | Cadence |
|---------|-----------|-----|-------------|----------------|-------------|---------------|---------|
| SD-01 | Code review before merge | A | PR review requirement | PR merge records | `/evidence-collect` | Change Log | Per change |
| SD-02 | Mandatory version control | A | Git usage | Git log | `/evidence-collect` | — | Ongoing |
| SD-03 | Secure-by-design principles | C | Training + code review | Review records | `/security-check` | — | Ongoing |
| SD-04 | Code scanned before deploy, patches in 90 days | A | `/security-check` + pre-deploy hook | Scan reports | `/security-check` | Change Log | Per deploy |
| SD-05 | No customer data for testing | C | PII hook + policy | PII scan logs | PII hook | — | Ongoing |
| SD-06 | Annual developer security training | C | Scheduled task | Training records | — | — | Annual |

## Third-Party Management (TP-01 through TP-05)

| Control | Statement | Cat | Enforcement | Evidence Source | Command/Hook | Asana Project | Cadence |
|---------|-----------|-----|-------------|----------------|-------------|---------------|---------|
| TP-01 | Due diligence before engaging | C | Vanta vendor assessment | Assessment records | — | — | Per engagement |
| TP-02 | Written agreements with security terms | C | Contract process | Signed agreements | — | — | Per agreement |
| TP-03 | Annual monitoring/review | B | Scheduled review + Vanta | Review records | `/compliance-status` | — | Annual |
| TP-04 | Cloud provider annual review | B | Scheduled review + Vanta | Review records | `/compliance-status` | — | Annual |
| TP-05 | 8-domain third-party assessment | C | Vanta vendor assessment | Assessment records | — | — | Per engagement |

## AI Governance (AI-01 through AI-08)

| Control | Statement | Cat | Enforcement | Evidence Source | Command/Hook | Asana Project | Cadence |
|---------|-----------|-----|-------------|----------------|-------------|---------------|---------|
| AI-01 | Work AI in managed accounts only | C | Policy + training | Account audit | — | — | Ongoing |
| AI-02 | Personal AI accounts prohibited | C | Policy + training | Policy acknowledgment | — | — | Ongoing |
| AI-03 | AI access via Asana workflow | B | Asana provisioning workflow | Access request tasks | — | — | Per request |
| AI-04 | PII anonymized before AI processing | A | PII scan hook | Hook scan logs | PII hook | — | Per use |
| AI-05 | Human-in-the-loop verification | C | Policy + training | Delivery review records | — | — | Per deliverable |
| AI-06 | AI credential sharing prohibited | C | Policy | Policy acknowledgment | — | — | Ongoing |
| AI-07 | Quarterly policy review | B | Scheduled review | Review minutes | `/compliance-status` | — | Quarterly |
| AI-08 | Employee AI policy acknowledgment | C | Vanta acknowledgment | Signed acknowledgments | — | — | Annual |

---

## Summary by Category

| Category | Count | Controls |
|----------|-------|----------|
| **A — Automatable** | 28 | AC-02, AC-03, AC-08, AC-11, AC-13, AM-03, CR-01, CR-02, CR-03, DM-02, IR-02, IR-03, IS-02, IS-04, OS-01, OS-03, OS-04, OS-05, OS-07, OS-11, OS-12, OS-15, RM-02, SD-01, SD-02, SD-04, AI-04 (27 + OS-13 partial = 28) |
| **B — Semi-Automated** | 31 | AC-01, AC-05, AC-06, AC-09, AC-10, AC-12, AC-14, AM-01, AM-02, BC-01, DM-03, DM-04, DM-05, HR-03, HR-05, IR-01, IR-04, OS-02, OS-06, OS-08, OS-09, OS-10, OS-14, RM-01, RM-03, TP-03, TP-04, AI-03, AI-07 (29 + 2 partial = 31) |
| **C — Policy-Only** | 32 | AC-04, AC-07, AM-04, AM-05, AM-06, BC-02, BC-03, CC-01, CC-02, CR-04, DM-01, HR-01, HR-02, HR-04, IR-05, IS-01, IS-03, IS-05, IS-06, PS-01, PS-02, PS-03, PS-04, SD-03, SD-05, SD-06, TP-01, TP-02, TP-05, AI-01, AI-02, AI-05, AI-06, AI-08 (32) |
