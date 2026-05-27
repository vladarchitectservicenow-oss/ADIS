# ADIS — Validation Checklist

## Purpose
Pre-release checklist for ADIS validation. All items must be checked before tagging a release.

---

## Code Quality

| # | Check | Method | Status |
|---|---|---|---|
| C1 | No hardcoded credentials in source | `grep -rPn "password=|DEFAULT_PASS|api_key" src/` | ☐ |
| C2 | All Script Includes compile in Rhino | Import to ServiceNow Studio → Check for syntax errors | ☐ |
| C3 | `sys_app.xml` is valid XML | `xmllint --noout src/sys_app.xml` | ☐ |
| C4 | No TODO/FIXME without ticket reference | `grep -r "TODO\|FIXME" src/ --include="*.js"` | ☐ |
| C5 | JavaScript consistent with Rhino (ES5) | No `const`, `let`, arrow functions, template literals | ☐ |
| C6 | `.gitignore` exists and excludes `__pycache__/` | `ls .gitignore` | ☐ |

---

## Data Model

| # | Check | Method | Status |
|---|---|---|---|
| D1 | All 4 tables defined in `tables_scan_run.xml` | Review XML for `x_adis_scan_run`, `x_adis_finding`, `x_adis_deprecation_rule`, `x_adis_remediation_task` | ☐ |
| D2 | Field types match spec (string → string, integer → integer) | Compare XML schema against architecture_summary.md | ☐ |
| D3 | No cross-scope references to non-`x_adis` tables | `grep "sys_" src/tables_scan_run.xml` — should be empty | ☐ |
| D4 | Reference fields include reference qualifier | Check `x_adis_finding.rule` references `x_adis_deprecation_rule` | ☐ |

---

## Functional Validation

| # | Check | Method | Status |
|---|---|---|---|
| F1 | Full scan produces `x_adis_scan_run` record | TSR-003 | ☐ |
| F2 | Deprecated API usage creates finding | TSR-004 | ☐ |
| F3 | Risk Score calculation is correct | TSR-005 | ☐ |
| F4 | Delta scan only reports changes | TSR-006, TSR-007 | ☐ |
| F5 | Concurrent scan prevention works | TSR-008 | ☐ |
| F6 | REST JSON API returns valid response | TSR-009 | ☐ |
| F7 | CSV export produces valid CSV | TSR-010 | ☐ |

---

## Security

| # | Check | Method | Status |
|---|---|---|---|
| S1 | Scanner cannot modify rules | TSR-011 | ☐ |
| S2 | Report reader cannot run scan | TSR-012 | ☐ |
| S3 | Unauthenticated REST requests return 401 | Manual: curl without auth | ☐ |
| S4 | No outbound network calls from scanner | Network audit or code review | ☐ |
| S5 | ACLs scoped to `x_adis` application context | Review ACL definitions in app | ☐ |
| S6 | Audit logging enabled for all operations | Check `sys_log` after scan | ☐ |

---

## Documentation

| # | Check | Method | Status |
|---|---|---|---|
| M1 | README.md has >= 2,000 words | `wc -w README.md` | ☐ |
| M2 | README.md has exactly 1 `## Overview` section | `grep -c '^## Overview$' README.md` | ☐ |
| M3 | README.md has exactly 1 `## License` section | `grep -c '^## License$' README.md` | ☐ |
| M4 | README.md contains Mermaid diagram | `grep -c '```mermaid' README.md` >= 1 | ☐ |
| M5 | README.md contains ROI analysis | `grep -c 'Savings' README.md` >= 1 | ☐ |
| M6 | README.md contains Troubleshooting table with 12+ rows | Count `\|.*\|.*\|` in Troubleshooting section | ☐ |
| M7 | LICENSE contains "Copyright (C) 2026 Vladimir Kapustin" | `grep -c "Vladimir Kapustin" LICENSE` >= 2 | ☐ |
| M8 | architecture_summary.md >= 40 lines | `wc -l memory/checkpoints/architecture_summary.md` >= 40 | ☐ |
| M9 | dependency_report.md >= 40 lines | `wc -l memory/checkpoints/dependency_report.md` >= 40 | ☐ |
| M10 | risk_report.md >= 40 lines | `wc -l memory/checkpoints/risk_report.md` >= 40 | ☐ |
| M11 | execution_plan.md >= 40 lines | `wc -l memory/checkpoints/execution_plan.md` >= 40 | ☐ |
| M12 | test_suite_SOP.md has 10+ scenarios | Count `## Scenario` headers >= 10 | ☐ |
| M13 | regression_cases.md has 5+ cases | Count `## Regression Case` headers >= 5 | ☐ |
| M14 | edge_cases.md has 10+ cases | Count `## Edge Case` headers >= 10 | ☐ |

---

## Release Artifacts

| # | Check | Method | Status |
|---|---|---|---|
| R1 | Update set XML exported | `ls releases/ADIS_update_set_v1.0.xml` (or similar) | ☐ |
| R2 | Version tag created | `git tag -l "v*"` | ☐ |
| R3 | GitHub Release published | Check GitHub Releases page | ☐ |
| R4 | DONE.marker committed | `ls DONE.marker` | ☐ |

---

## Environment Compatibility

| # | Check | Method | Status |
|---|---|---|---|
| E1 | App installs on Zurich PDI | Import update set on Zurich instance | ☐ |
| E2 | App installs on Australia PDI | Import update set on Australia instance | ☐ |
| E3 | Scan completes on instance with 50K+ scripts | Load test on populated sub-production | ☐ |
| E4 | No browser console errors in dashboard | Open dashboard in Chrome, check F12 console | ☐ |

---

## Sign-Off

| Role | Name | Date | Signature |
|---|---|---|---|
| Developer | | | |
| QA | | | |
| Security Review | | | |
| Release Manager | | | |
