# ADIS — Risk Report

## Risk Assessment Overview
**Assessment Date**: 2026-05-27  
**Assessor**: Automated pipeline (repo-validation-engineer)  
**Risk Methodology**: Severity × Likelihood × Impact (1–5 scale each)

---

## Technical Risks

| ID | Risk | Severity | Likelihood | Impact | Score | Mitigation |
|---|---|---|---|---|---|---|
| T1 | Regex false positives matching valid code | 3 | 4 | 3 | 36 | Pre-validate all regex patterns against Zurich codebase before publishing rules |
| T2 | Regex false negatives missing deprecated APIs | 4 | 3 | 4 | 48 | Maintain rule coverage tracker; cross-reference against ServiceNow deprecation announcements |
| T3 | Background Script timeout on large instances (>200K scripts) | 3 | 3 | 4 | 36 | Chunked scanning by table; configurable timeout; delta mode reduces full scan frequency |
| T4 | GlideRecord query limits truncating scan results | 2 | 2 | 5 | 20 | Use `setLimit` and pagination; enforce maximum batch size |
| T5 | Rule engine performance degradation with many rules (>500) | 2 | 2 | 3 | 12 | Index rules by release scope; pre-compile regex patterns on load |
| T6 | Rhino RegExp differences causing rule mismatch after Australia upgrade | 4 | 2 | 4 | 32 | Test all rules against both Zurich and Australia Rhino environments |
| T7 | Concurrent scan runs causing data corruption | 3 | 2 | 5 | 30 | Mutex via `x_adis_scan_run` status check; reject new scan if any `running` scan exists |

## Operational Risks

| ID | Risk | Severity | Likelihood | Impact | Score | Mitigation |
|---|---|---|---|---|---|---|
| O1 | No scan completes before Australia upgrade deadline | 5 | 4 | 5 | 100 | Schedule scans 4 weeks before upgrade window; automated reminder via Scheduled Job |
| O2 | Admin runs scan in production without testing on sub-prod first | 3 | 4 | 4 | 48 | README and install guide mandate sub-prod testing; add confirmation prompt before first production scan |
| O3 | Findings overwhelm remediation team (1,000+ Critical findings) | 3 | 3 | 4 | 36 | Severity-based filtering; remediation tracking dashboard; delta mode shows progress |
| O4 | App uninstalled accidentally removing all scan history | 4 | 1 | 5 | 20 | Backup scan results to CSV before major operations; document export procedure |
| O5 | No one assigned to review scan results | 3 | 3 | 5 | 45 | Auto-assign remediation tasks to platform team group; email notification on scan completion |

## Security Risks

| ID | Risk | Severity | Likelihood | Impact | Score | Mitigation |
|---|---|---|---|---|---|---|
| S1 | Cross-scope privilege escalation via ACL misconfiguration | 4 | 2 | 5 | 40 | Default-deny ACLs; audit all cross-scope access; restrict to `x_adis` scope |
| S2 | Script code snippets in findings expose sensitive logic | 2 | 4 | 3 | 24 | RBAC on `x_adis_finding` table; restrict read access to `x_adis.admin` and `x_adis.scanner` |
| S3 | Scan results exfiltrated via report export | 2 | 2 | 3 | 12 | RBAC on report endpoints; audit log for all exports |
| S4 | Hardcoded credentials in source code | 4 | 1 | 5 | 20 | Code review gate; config table for credentials; env var injection |

## Business Risks

| ID | Risk | Severity | Likelihood | Impact | Score | Mitigation |
|---|---|---|---|---|---|---|
| B1 | Australia upgrade proceeds without ADIS scan → production outage | 5 | 2 | 5 | 50 | Mandatory scan gate in upgrade runbook; executive sponsor sign-off required to skip |
| B2 | False positive flood erodes trust in tool | 3 | 3 | 4 | 36 | Tune regex patterns to <5% false positive rate; provide "mark as false positive" workflow |
| B3 | Commercial customers demand features beyond AGPL scope | 2 | 3 | 2 | 12 | Commercial licensing option available; clear feature roadmap published |
| B4 | ServiceNow releases Australia patches that change deprecation status mid-cycle | 3 | 3 | 4 | 36 | Rules as data (not code); update `x_adis_deprecation_rule` via update set without app upgrade |

## Risk Score Summary

| Category | Average Score | Risk Level |
|---|---|---|
| Technical (7 risks) | 30.6 | MEDIUM |
| Operational (5 risks) | 49.8 | HIGH |
| Security (4 risks) | 24.0 | LOW |
| Business (4 risks) | 33.5 | MEDIUM |
| **Overall (20 risks)** | **34.5** | **MEDIUM** |

## Top 5 Risks by Score

1. **O1** — No scan completes before upgrade deadline (100) — **HIGH**
2. **T2** — Regex false negatives missing deprecated APIs (48) — **HIGH**
3. **O2** — Admin runs scan in production without testing (48) — **HIGH**
4. **O5** — No one assigned to review scan results (45) — **HIGH**
5. **S1** — Cross-scope privilege escalation (40) — **MEDIUM**

## Risk Burndown Plan

| Phase | Actions | Target Score Reduction |
|---|---|---|
| Pre-release (v1.0) | T1, T2, T6 (regex validation suite) | -20 pts |
| v1.1 | O1 (scheduled scan reminders), O2 (confirmation prompt) | -30 pts |
| v1.2 | O5 (auto-assignment), S1 (ACL audit) | -25 pts |
| Continuous | B4 (rule update pipeline), T4 (performance tuning) | -10 pts |

## Risk Acceptance Criteria
- All HIGH risks (O1, T2, O2, O5) must be mitigated to MEDIUM before Australia upgrade window opens
- All CRITICAL-score risks (>60) are unacceptable and require immediate remediation
- Overall risk score must be <25 (LOW) before production deployment to regulated-industry instances
