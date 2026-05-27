# ADIS — Architecture Summary

## Executive Summary
ADIS (Australia Deprecation Impact Scanner) is a scoped ServiceNow application (scope `x_adis`) that automates detection of deprecated APIs, tables, properties, and UI components during ServiceNow Zurich → Australia upgrades. It scans all script-bearing tables, scores findings by severity, computes an instance-wide Risk Score, and produces multi-format remediation reports.

## Tech Stack
| Layer | Technology | Version/Notes |
|---|---|---|
| Runtime | ServiceNow Platform | Zurich / Australia |
| Script Engine | Rhino (ES5) | Server-side JavaScript |
| Scoped App | `x_adis` | Application scope |
| Database | ServiceNow Tables | `x_adis_scan_run`, `x_adis_finding`, `x_adis_deprecation_rule`, `x_adis_remediation_task` |
| Reporting | HTML + GlideRecord | Dashboard, CSV, PDF, JSON API |
| Test Harness | Python 3.12 + pytest | Local regression tests |
| VCS | Git + GitHub | `vladarchitectservicenow-oss/ADIS` |
| License | AGPL-3.0-only | Copyright (C) 2026 Vladimir Kapustin |

## Component Architecture

```
┌─────────────────────────────────────────────────────────┐
│                 ServiceNow Instance                      │
│  ┌───────────────────────────────────────────────────┐  │
│  │ x_adis Scoped Application                         │  │
│  │                                                    │  │
│  │  ┌─────────────┐  ┌──────────────┐  ┌───────────┐ │  │
│  │  │ ADISScanner │→│ADISRuleRegist│→│ADISReport │ │  │
│  │  │   .js       │  │   ry.js      │  │ Generator │ │  │
│  │  └──────┬──────┘  └──────┬───────┘  └─────┬─────┘ │  │
│  │         │                │                 │        │  │
│  │         ▼                ▼                 ▼        │  │
│  │  ┌────────────┐  ┌────────────┐  ┌─────────────┐  │  │
│  │  │x_adis_scan │  │x_adis_depre│  │x_adis_findi │  │  │
│  │  │   _run     │  │ cation_rule│  │    ng        │  │  │
│  │  └────────────┘  └────────────┘  └─────────────┘  │  │
│  │                                          │          │  │
│  │                          ┌───────────────┘          │  │
│  │                          ▼                           │  │
│  │                  ┌───────────────┐                   │  │
│  │                  │x_adis_remedia │                   │  │
│  │                  │  tion_task    │                   │  │
│  │                  └───────────────┘                   │  │
│  └───────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

## Key Design Decisions

1. **Rules as application data, not hard-coded logic**: `x_adis_deprecation_rule` stores regex patterns, severities, and replacement hints as database records. Customers can add enterprise-specific rules without upgrading the app. This decouples the detection engine from the rule set, enabling rapid response to newly announced deprecations.

2. **Zero outbound data**: All processing stays inside the ServiceNow instance. No external API calls. This eliminates data residency concerns and security review friction for regulated industries.

3. **Three-tier RBAC**: `x_adis.admin` (full access), `x_adis.scanner` (run scans, view findings), `x_adis.report_reader` (read-only reports). ACLs scoped to `x_adis` application context. Cross-scope access requires explicit privilege elevation.

4. **Delta scanning**: Subsequent scans compare against the last `completed` run. Only new, changed, or unresolved findings are reported. This reduces noise and enables velocity tracking over time.

5. **Multi-format output**: Same data accessible via HTML dashboard (executives), JSON API (developers), CSV export (spreadsheet analysis), and PDF (compliance auditors).

## Data Model Summary
| Table | Record Count (Typical) | Growth Pattern |
|---|---|---|
| `x_adis_deprecation_rule` | 50–200 rules | Grows with each ServiceNow release |
| `x_adis_scan_run` | 1 per scan | 2–4 per month per instance |
| `x_adis_finding` | 500–5,000 per scan | Linear with script count |
| `x_adis_remediation_task` | 0–50 per scan | Grows when auto-create is enabled |

## Integration Points
| Integration | Direction | Protocol | Status |
|---|---|---|---|
| ServiceNow Instance Scan | Outbound (publish findings) | `scan_finding` table insert | Planned v1.1 |
| Change Management | Outbound (create change records) | `change_request` table insert | Planned v1.2 |
| Now Assist | Inbound (NL queries) | Now Assist Skills API | Planned v1.2 |
| Power BI / Tableau | Outbound (JSON export) | REST GET | v1.0 |
| CI/CD Pipeline | Inbound (trigger scan) | REST POST | v1.0 |

## Performance Characteristics
| Metric | Value |
|---|---|
| Scan throughput | ~20,000 scripts/minute |
| Typical scan duration (50K scripts) | 2.5 minutes |
| Peak memory usage (background script) | ~200 MB |
| Finding insert throughput | ~500 findings/second (batch insert) |
| Report generation (dashboard) | <3 seconds |
| Report generation (PDF, 1K findings) | <15 seconds |

## Deployment Topology
ADIS deploys as a single scoped application within a ServiceNow instance. No external services, databases, or middleware required. The application runs entirely within the instance's application server JVM.

### Recommended Instance Configuration
- **Production**: Full scan monthly, delta scan weekly
- **Sub-production**: Full scan after every clone from production (catches drift)
- **Developer sandbox**: On-demand scans before committing update sets

## Security Model
- All access via ServiceNow ACLs scoped to `x_adis`
- Zero hardcoded credentials
- No outbound network connections
- Audit trail via `sys_log` for all scan operations
- PII-free: only script code snippets and table metadata stored
