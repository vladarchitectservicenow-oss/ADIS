# ADIS — Dependency Report

## Core Dependencies

### ServiceNow Platform Dependencies
| Dependency | Type | Version Required | Risk if Unavailable |
|---|---|---|---|
| GlideRecord | API — Data query | Zurich+ | Scan engine cannot query any table; complete failure |
| GlideSystem (`gs`) | API — System calls | Zurich+ | Logging (`gs.info`), session management broken; scan runs silently |
| GlideDateTime | API — Timestamps | Zurich+ | Scan run duration calculation fails; reports show null timestamps |
| Scoped Application Framework | Runtime | Zurich+ | Application cannot be installed or published |
| Update Set / Source Control | Deployment | Any | App cannot be imported or tracked in version control |

### Scoped Table Dependencies
| Table | Relationship | Cardinality | Description |
|---|---|---|---|
| `x_adis_scan_run` | Parent → `x_adis_finding.scan_run` | 1:N | One scan produces many findings |
| `x_adis_scan_run` | Parent → `x_adis_remediation_task.scan_run` | 1:N | One scan may auto-create multiple tasks |
| `x_adis_deprecation_rule` | Reference → `x_adis_finding.rule` | 1:N | One rule may match many findings |
| `x_adis_finding` | Parent → `x_adis_remediation_task.finding` | 1:1 | Each finding may have one remediation task |

### Script Include Dependencies
| Module | Depends On | Nature | Critical |
|---|---|---|---|
| `ADISScanner` | `ADISRuleRegistry` | Constructor injection | Yes — scanner cannot function without rules |
| `ADISScanner` | `GlideRecord` | Runtime | Yes — all table queries depend on GR |
| `ADISReportGenerator` | `GlideRecord` | Runtime | Yes — report queries run against findings/scan tables |
| `ADISReportGenerator` | `GlideExcelParser` (optional) | Runtime | No — CSV export degrades gracefully |
| `ADISReportGenerator` | `PDFGenerationAPI` (optional) | Runtime | No — PDF export disabled if unavailable |

### External Dependencies (Zero)
| Category | Count | Notes |
|---|---|---|
| NPM packages | 0 | All JavaScript runs server-side in Rhino engine |
| Python packages (test harness) | 2 | `pytest`, `requests` — dev/test only, not runtime |
| External APIs | 0 | Zero outbound calls by design |
| Database drivers | 0 | All data in ServiceNow tables |
| Third-party services | 0 | No SaaS integrations |

### Python Test Harness Dependencies (dev-only)
| Package | Version | Purpose |
|---|---|---|
| `pytest` | >= 7.0 | Test runner for `test_adis_rule_registry.py`, `test_adis_scan_e2e.py` |
| `requests` | >= 2.28 | HTTP client for REST API tests |

## Indirect / Transitive Dependencies

### ServiceNow Platform Internals
ADIS relies on the standard ServiceNow server-side JavaScript runtime (Rhino engine). Any breaking change to Rhino's `RegExp` implementation would affect rule matching. However, Rhino changes are documented in ServiceNow release notes and are extremely rare.

### Instance Configuration
| Setting | Required Value | Impact if Wrong |
|---|---|---|
| `glide.script.use.sandbox` | `true` (default) | Scoped app isolation may break |
| `glide.rest.api.enabled` | `true` | REST API endpoints unavailable |
| `glide.ui.export.pdf.enabled` | `true` | PDF export unavailable |
| `com.glide.sys.info` logging level | `info` or lower | Debug logging unavailable |

## Dependency Risk Matrix

| Risk Level | Dependency | Mitigation |
|---|---|---|
| CRITICAL | GlideRecord API | ServiceNow core API — only breaks with major platform release |
| CRITICAL | Scoped App Framework | ServiceNow core feature — deprecation would be announced 2+ releases prior |
| HIGH | ADISRuleRegistry | Internal dependency — version-locked within the app |
| MEDIUM | Rhino RegExp engine | ServiceNow core — test regex patterns against target release before upgrade |
| LOW | PDFGenerationAPI | Optional feature — graceful degradation if unavailable |
| LOW | pytest/requests (test harness) | Dev-only — no impact on production runtime |

## Version Compatibility Matrix
| ServiceNow Release | ADIS Version | Status |
|---|---|---|
| Zurich | 1.0.0 | Fully supported |
| Australia | 1.0.0 | Target release; rule set optimized for this upgrade path |
| Xanadu | 1.0.0 (untested) | May work; no Australia-specific rules will match |
| Washington DC | 1.0.0 (untested) | Unsupported; `GlideElementDynamicAttribute` already removed |
