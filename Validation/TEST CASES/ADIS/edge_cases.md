# ADIS — Edge Cases

## Purpose
Documents boundary conditions and edge cases that must be handled gracefully. Each edge case describes the scenario, expected behavior, and validation approach.

---

## Edge Case 1: Zero Script-Bearing Tables Populated
**Scenario**: Instance has no records in any of the 8 script-bearing tables (fresh install, empty sub-production).  
**Expected Behavior**: Scan completes with `records_scanned=0`, `findings_count=0`, Risk Score=0. No errors, no crashes.  
**Validation**: Run scan on clean sub-production instance. Assert graceful zero-result handling.

---

## Edge Case 2: Single Script Include with 1,000,000 Characters
**Scenario**: A single Script Include contains a 1MB script body (edge case from code generation tools dumping large JSON/JS).  
**Expected Behavior**: Regex engine processes the body without timeout. Finding insertion works (ServiceNow field max is 4MB for string fields).  
**Validation**: Create 1MB test Script Include. Run scan. Verify no timeout and no truncation.

---

## Edge Case 3: Rule with Empty Regex Pattern
**Scenario**: An active rule has `regex_pattern` = `""` (empty string).  
**Expected Behavior**: Empty regex matches everything in JavaScript — this would create thousands of false positives. Scanner should skip rules with empty patterns and log a warning.  
**Validation**: TSR-002 variant — insert rule with empty pattern, verify it's skipped.

---

## Edge Case 4: Rule with Invalid Regex (Uncompilable)
**Scenario**: A rule contains `regex_pattern` = `"[invalid"` (unclosed character class).  
**Expected Behavior**: `new RegExp(pattern)` throws `SyntaxError`. Scanner must catch this, log the error, and skip the rule without crashing the entire scan.  
**Validation**: TSR-002 variant — insert uncompileable regex, verify scanner continues scanning other rules.

---

## Edge Case 5: 10,000+ Deprecation Rules Active
**Scenario**: Enterprise customer adds 10,000 custom rules covering internal APIs.  
**Expected Behavior**: Rule loading completes (batch query), regex matching overhead is linear O(rules × scripts). Performance may degrade but should not crash.  
**Validation**: Load test harness: seed 10K rules, run scan against 1K scripts, measure time.

---

## Edge Case 6: Scan Run with `status` = `deleted`
**Scenario**: A mid-scan crash leaves a scan run in `running` status. Admin deletes the record manually. Delta scan references the deleted scan.  
**Expected Behavior**: Delta scan falls back to full scan mode with a warning log.  
**Validation**: REG-004 covers this behavior.

---

## Edge Case 7: REST API Called with Invalid `run_id`
**Scenario**: GET `/api/x_adis/scan/report?run_id=nonexistent`.  
**Expected Behavior**: HTTP 404 with descriptive JSON error body: `{"error": "Scan run not found", "run_id": "nonexistent"}`.  
**Validation**: TSR-009 extended — test with invalid sys_id.

---

## Edge Case 8: CSV Export with 0 Findings
**Scenario**: Scan completed successfully but found zero deprecated APIs.  
**Expected Behavior**: CSV contains header row only. No empty data rows.  
**Validation**: TSR-010 extended — run scan on clean instance, export CSV, verify header-only output.

---

## Edge Case 9: Multiple Scans in Same Second
**Scenario**: Two scans triggered in the same second by different users or via API.  
**Expected Behavior**: Concurrent scan prevention rejects the second scan. Scan run IDs are unique (GlideRecord sys_ids are UUID-based and won't collide).  
**Validation**: TSR-008 covers this.

---

## Edge Case 10: Finding Code Snippet Contains `NULL` Byte
**Scenario**: Script body contains a null character (`\x00`) from corrupted import.  
**Expected Behavior**: ServiceNow GlideRecord may reject null bytes in string fields. Scanner should sanitize code snippets by stripping null bytes before insert.  
**Validation**: Create Script Include with null byte. Run scan. Verify finding stored without `\x00`.
