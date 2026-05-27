# ADIS — Regression Cases

## Purpose
This document catalogs known failure modes and regression risks. Every item listed here must have a corresponding test in the test suite or be validated manually before each release.

---

## Regression Case 1: Rule Engine Doesn't Load After Upgrade
**ID**: REG-001  
**Severity**: Critical  
**Root Cause**: Schema change in `x_adis_deprecation_rule` table during app update. If a new field is added to the rule table and `ADISRuleRegistry.js` doesn't handle `null` gracefully, `registry.getActiveRules()` may throw a `TypeError` and crash the scanner.  
**Reproduction**: Update app from v0.9 to v1.0 without running the table alter script first.  
**Fix**: Wrap all `GlideRecord.getValue()` calls with null-coalescing: `gr.getValue("field") || ""`.  
**Test**: TSR-001 covers this — if any active rule has a null field, the test catches it.

---

## Regression Case 2: Risk Score Division by Zero
**ID**: REG-002  
**Severity**: High  
**Root Cause**: If a scan produces 0 findings (all rules match nothing), the Risk Score formula divides by zero and returns `NaN` or `Infinity`.  
**Reproduction**: Run scan on instance with no script-bearing tables populated.  
**Fix**: Guard clause: `if (totalFindings === 0) return 0;` before division.  
**Test**: TSR-005 variant — feed 0 findings and assert Risk Score = 0.

---

## Regression Case 3: CSV Export Corrupts Special Characters
**ID**: REG-003  
**Severity**: Medium  
**Root Cause**: Code snippets in `x_adis_finding.code_snippet` may contain commas, quotes, or newlines that break CSV formatting.  
**Reproduction**: Create a finding with code snippet containing `"` and `,` characters.  
**Fix**: Proper CSV escaping: wrap all fields in double quotes, double-escape internal quotes.  
**Test**: TSR-010 extended — verify CSV round-trips correctly through Excel/Google Sheets import.

---

## Regression Case 4: Delta Scan References Deleted Baseline
**ID**: REG-004  
**Severity**: High  
**Root Cause**: If the reference scan (last `completed` scan) is deleted, delta scan crashes with a null reference error.  
**Reproduction**: Run full scan → delete the `x_adis_scan_run` record → run delta scan.  
**Fix**: Fallback: if baseline scan not found, treat delta as full scan and log warning.  
**Test**: TSR-006/007 extended — test delta behavior when baseline is deleted.

---

## Regression Case 5: Large Script Body Causes Regex Catastrophic Backtracking
**ID**: REG-005  
**Severity**: Medium  
**Root Cause**: A poorly written regex (e.g., `(a+)+b` pattern) against a 100KB script body causes exponential backtracking and hangs the scanner.  
**Reproduction**: Create a 100KB Script Include and match against a vulnerable regex with nested quantifiers.  
**Fix**: All regex patterns pre-tested against 100KB input; timeout wrapper around regex execution.  
**Test**: TSR-003 extended — include a 100KB dummy script body in scan.

---

## Regression Case 6: Report Generator Fails on UTF-8 Characters
**ID**: REG-006  
**Severity**: Low  
**Root Cause**: ServiceNow PDF/CSV generation may not handle UTF-8 correctly if code snippets contain emoji or non-Latin characters.  
**Reproduction**: Create Script Include with emoji in comments. Run scan. Export PDF.  
**Fix**: Ensure `Content-Type` headers specify charset; test with Unicode edge cases.  
**Test**: Manual validation — include "🚀" and "日本語" in test code.
