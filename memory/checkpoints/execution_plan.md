# ADIS — Execution Plan

## Plan Metadata
| Field | Value |
|---|---|
| Repository | `vladarchitectservicenow-oss/ADIS` |
| Scope | `x_adis` |
| Target Release | v1.0.0 (Zurich → Australia support) |
| Start | 2026-05-27 |
| Methodology | Agile / Kanban |
| Tools | ServiceNow Studio, GitHub, pytest |

---

## Phase 1: Foundation (COMPLETE)

### Deliverables
- [x] Scoped application skeleton (`sys_app.xml`)
- [x] Data model: 4 tables defined in `src/tables_scan_run.xml`
- [x] Core Script Includes: `ADISScanner.js`, `ADISRuleRegistry.js`, `ADISReportGenerator.js`
- [x] AGPL-3.0 LICENSE with copyright
- [x] Architecture documentation
- [x] Architecture plan (`src/architecture_plan.md`)

### Verification
- [x] `sys_app.xml` imports successfully into ServiceNow Studio
- [x] All 4 tables created in `x_adis` scope
- [x] Script Includes compile without syntax errors in Rhino engine

---

## Phase 2: Rule Engine (COMPLETE — needs expansion)

### Deliverables
- [x] `ADISRuleRegistry.js` loads rules from `x_adis_deprecation_rule`
- [x] Regex-based pattern matching
- [x] Severity scoring (Critical=4, High=3, Medium=2, Low=1)
- [ ] **Pre-seeded rule set for Zurich → Australia** (15+ rules)
- [ ] Rule validation harness (test regex against known deprecated signatures)

### Pre-Seeded Rules — High Priority
| # | Deprecated Element | Severity | Release |
|---|---|---|---|
| 1 | `GlideElementDynamicAttribute` | Critical | Australia |
| 2 | `GlideEncrypter` API | High | Australia |
| 3 | Data Generation profiles | High | Australia |
| 4 | Agent Workspace (legacy) | Medium | Australia |
| 5 | Document Intelligence | Medium | Australia |
| 6 | `GlideAggregate.addHaving()` legacy signature | Medium | Australia |
| 7 | `g_form.getReference()` deprecated callback | High | Australia |
| 8 | `GlideSysAttachment.copy()` removed | Critical | Australia |
| 9 | `GlideSchedule` deprecated API | Medium | Australia |
| 10 | Legacy UI16 macros | Low | Australia |
| 11 | `sys_attachment` direct queries | High | Australia |
| 12 | `Packages.com.glide.util.*` imports | Critical | Australia |
| 13 | `GlideRecord.addNullQuery()` removed | High | Australia |
| 14 | `current.update()` without setWorkflow | Low | Australia |
| 15 | `gs.eventQueue()` deprecated event naming | Medium | Australia |

### Verification
- [ ] All 15 rules have valid regex patterns
- [ ] All 15 rules have replacement hints
- [ ] All 15 rules have documentation URLs linking to ServiceNow docs
- [ ] `pytest test_adis_rule_registry.py` passes 10/10

---

## Phase 3: Scanner Engine (IN PROGRESS)

### Deliverables
- [ ] `ADISScanner.js` iterates over 8 script-bearing tables
- [ ] Scan result stored in `x_adis_scan_run` with status tracking
- [ ] Each finding stored in `x_adis_finding` with rule reference
- [ ] Risk Score calculation: `(C×4 + H×3 + M×2 + L×1) / Total × 25`
- [ ] Delta scan mode (compare against last `completed` scan)
- [ ] Configurable timeout and table scope

### Script-Bearing Tables to Scan
| # | Table | Estimated Records |
|---|---|---|
| 1 | `sys_script_include` | 100–500 |
| 2 | `sys_script` (Business Rules) | 200–2,000 |
| 3 | `sys_script_client` | 50–500 |
| 4 | `sysauto_script` (Scheduled Jobs) | 10–100 |
| 5 | `sys_transform_script` | 5–50 |
| 6 | `sys_ui_action` | 50–300 |
| 7 | `sys_hub_action_type_definition` | 10–200 |
| 8 | `sys_ui_macro` | 5–50 |

### Verification
- [ ] Full scan completes in <5 minutes on 100K records
- [ ] Each finding has non-null `rule`, `severity`, `code_snippet`, `table_name`, `record_sys_id`
- [ ] `x_adis_scan_run.risk_score` is non-zero when Critical findings exist
- [ ] Delta scan only reports new/changed findings
- [ ] Concurrent scan attempt while `status=running` returns error

---

## Phase 4: Report Generator (IN PROGRESS)

### Deliverables
- [ ] `ADISReportGenerator.js` produces HTML dashboard
- [ ] CSV export with findings table
- [ ] PDF executive summary
- [ ] JSON REST API endpoint (`/api/x_adis/scan/report`)

### Verification
- [ ] HTML dashboard loads in <3 seconds
- [ ] CSV export contains `sys_id, rule_name, severity, table_name, code_snippet, remediation_hint`
- [ ] PDF export <15 seconds for 1K findings
- [ ] JSON API returns valid JSON with `risk_score`, `findings_count`, `severity_breakdown`
- [ ] `x_adis.report_reader` role can GET but not POST

---

## Phase 5: RBAC & Security (TODO)

### Deliverables
- [ ] 3 roles: `x_adis.admin`, `x_adis.scanner`, `x_adis.report_reader`
- [ ] ACLs on all 4 tables
- [ ] ACLs on REST endpoints
- [ ] Audit logging for all scan operations via `sys_log`

### Verification
- [ ] `x_adis.scanner` can run scan but cannot modify rules
- [ ] `x_adis.report_reader` can read findings but cannot run scan
- [ ] Unauthenticated REST requests return 401
- [ ] All operations logged to `sys_log`

---

## Phase 6: Testing & Validation (TODO)

### Deliverables
- [ ] ATF test suite (7 scenarios)
- [ ] Python test harness: 10+ scenarios
- [ ] Stress test: 200K scripts, 500 rules
- [ ] Security review: ACL audit, credential scan, data residency check

### Verification
- [ ] All ATF tests pass
- [ ] `pytest tests/ -v` passes 10/10
- [ ] Stress test completes without timeout or OOM
- [ ] No hardcoded credentials in source

---

## Phase 7: Documentation (COMPLETE — verified 2026-05-27)

### Deliverables
- [x] README.md (2,000+ words, Mermaid diagrams, ROI analysis, troubleshooting)
- [x] LICENSE (AGPL-3.0-only + Copyright Vladimir Kapustin)
- [x] architecture_summary.md
- [x] dependency_report.md
- [x] risk_report.md
- [x] execution_plan.md (this document)
- [x] test_suite_SOP.md (10+ scenarios)
- [x] regression_cases.md
- [x] edge_cases.md
- [x] validation_checklist.md
- [ ] WHITEPAPER.md (marketing collateral — in `/marketing/`)
- [ ] LINKEDIN_POST.md (social media announcement — in `/marketing/`)

---

## Phase 8: Release (TODO)

### Deliverables
- [ ] Version tag: `v1.0.0`
- [ ] GitHub Release with release notes
- [ ] Update set XML export (`releases/ADIS_update_set_v1.0.xml`)
- [ ] ServiceNow Share submission (optional)

---

## Risk & Blockers

| Blocker | Status | Owner | Resolution |
|---|---|---|---|
| Rule set not pre-seeded | TODO | Dev | Create 15 Zurich→Australia rules as table records |
| ATF test suite not implemented | TODO | QA | 7 scenarios per test_suite_SOP.md |
| No production deployment tested | TODO | Ops | Deploy to sub-prod, run full scan, verify findings |
| PDF generation API availability uncertain | INVESTIGATE | Dev | Test PDFGenerationAPI on target instance; fallback to CSV |

## Timeline

| Phase | Estimated Effort | Target |
|---|---|---|
| Phase 1: Foundation | Done | Done |
| Phase 2: Rule Engine | 3 hours | Week 1 |
| Phase 3: Scanner Engine | 5 hours | Week 1–2 |
| Phase 4: Report Generator | 4 hours | Week 2 |
| Phase 5: RBAC & Security | 3 hours | Week 2–3 |
| Phase 6: Testing | 5 hours | Week 3 |
| Phase 7: Documentation | Done | Done |
| Phase 8: Release | 2 hours | Week 3 |

**Total remaining effort**: ~22 hours across 3 weeks.
