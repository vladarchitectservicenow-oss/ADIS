# ADIS — Test Suite SOP (Standard Operating Procedure)

## Purpose
This document defines the complete test suite for ADIS (Australia Deprecation Impact Scanner). All scenarios must pass before any release is tagged or deployed to production.

## Test Environment Setup
```bash
# Clone the repository
git clone https://github.com/vladarchitectservicenow-oss/ADIS.git
cd ADIS

# Install Python test dependencies
pip install pytest requests

# Set environment variables for ServiceNow connection
export SN_URL="https://dev362840.service-now.com"
export SN_USER="admin"
export SN_PASS="your_password"
```

---

## Scenario 1: Rule Registry — Load All Active Rules
**ID**: TSR-001  
**Priority**: Critical  
**Type**: Unit Test  

**Description**: Verify that `ADISRuleRegistry.js` loads all active rules from `x_adis_deprecation_rule` table where `active=true`.

**Preconditions**:
- At least 5 active rules exist in `x_adis_deprecation_rule`
- At least 1 inactive rule exists (for negative test)

**Steps**:
1. Instantiate `new ADISRuleRegistry()`
2. Call `registry.getActiveRules("Zurich→Australia")`
3. Assert result is an array with >= 5 elements
4. Assert every returned rule has `regex_pattern`, `severity`, `replacement_hint`, `documentation_url`
5. Assert no inactive rules are in the result

**Expected Result**: All active rules returned; inactive rules excluded; all fields populated.

---

## Scenario 2: Rule Registry — Regex Pattern Validation
**ID**: TSR-002  
**Priority**: Critical  
**Type**: Unit Test  

**Description**: Verify that all regex patterns in active rules compile and match known deprecated code samples.

**Preconditions**:
- Rule for `GlideElementDynamicAttribute` exists with regex pattern

**Steps**:
1. Load rule for "GlideElementDynamicAttribute"
2. Test pattern against `"new GlideElementDynamicAttribute()"` → must match
3. Test pattern against `"new GlideRecord('incident')"` → must NOT match (no false positive)
4. Test pattern against `"// new GlideElementDynamicAttribute()"` → must NOT match (commented code)
5. Assert `new RegExp(pattern).test(sample)` returns expected boolean

**Expected Result**: Pattern matches deprecated usage, excludes valid code and comments.

---

## Scenario 3: Scanner — Full Scan Execution
**ID**: TSR-003  
**Priority**: Critical  
**Type**: Integration Test  

**Description**: Execute a full scan and verify a `x_adis_scan_run` record is created with correct metadata.

**Steps**:
1. Call `new ADISScanner(new ADISRuleRegistry()).runScan("full")`
2. Query `x_adis_scan_run` for the returned `runId`
3. Assert `status` == `"running"` or `"completed"` (depends on sync/async)
4. Assert `scan_type` == `"full"`
5. Assert `records_scanned` > 0
6. Assert `start_time` is not null

**Expected Result**: Scan run record exists with valid metadata.

---

## Scenario 4: Scanner — Finding Creation for Deprecated API
**ID**: TSR-004  
**Priority**: Critical  
**Type**: Integration Test  

**Description**: Create a Script Include with a known deprecated API, run scan, verify finding is created.

**Preconditions**:
- Test Script Include created with body containing `new GlideElementDynamicAttribute()` or other deprecated API

**Steps**:
1. Insert test Script Include into `sys_script_include`
2. Run full scan
3. Query `x_adis_finding` where `table_name` == `"sys_script_include"` and record matches test record
4. Assert finding exists with severity >= 3 (Critical or High)
5. Assert `code_snippet` contains the deprecated text
6. Assert `remediation_hint` is not null

**Expected Result**: One finding created for the test Script Include, correctly attributed.

---

## Scenario 5: Risk Score Calculation
**ID**: TSR-005  
**Priority**: High  
**Type**: Unit Test  

**Description**: Verify Risk Score formula produces correct values for known finding combinations.

**Steps**:
1. Create test findings: 2 Critical, 3 High, 5 Medium, 10 Low
2. Calculate expected score: `(2×4 + 3×3 + 5×2 + 10×1) / 20 × 25 = (8+9+10+10)/20 × 25 = 37/20 × 25 = 46.25`
3. Run scanner risk score calculation method
4. Assert `Math.abs(actual - 46.25) < 0.1`

**Expected Result**: Risk Score = 46.25 ± 0.1.

---

## Scenario 6: Delta Scan — Only Reports Changes
**ID**: TSR-006  
**Priority**: High  
**Type**: Integration Test  

**Description**: Run full scan, then delta scan with no changes → delta should report 0 new findings.

**Steps**:
1. Run full scan → note finding count (N)
2. Run delta scan (no code changes between scans)
3. Query `x_adis_finding` created by delta scan → count = 0
4. Assert delta scan `records_scanned` > 0 (it did scan)
5. Assert delta scan `findings_count` == 0 (no new findings)

**Expected Result**: Delta scan reports zero new findings when no code changed.

---

## Scenario 7: Delta Scan — Reports New and Changed Findings
**ID**: TSR-007  
**Priority**: High  
**Type**: Integration Test  

**Description**: After modifying a Script Include to add a deprecated API, delta scan detects only the new finding.

**Steps**:
1. Run full scan → baseline
2. Modify test Script Include to add `GlideEncrypter.encrypt()` call
3. Run delta scan
4. Assert delta scan `findings_count` >= 1 (at least the new finding)
5. Query new findings → assert at least one references the modified Script Include

**Expected Result**: Delta scan detects the new deprecated usage.

---

## Scenario 8: Concurrent Scan Prevention
**ID**: TSR-008  
**Priority**: Medium  
**Type**: Integration Test  

**Description**: Attempt to start a second scan while one is running → second scan should fail gracefully.

**Steps**:
1. Start scan 1 (synchronous or long-running)
2. Attempt to start scan 2 while scan 1 `status` == `"running"`
3. Assert scan 2 returns error or rejection message
4. Assert no second `x_adis_scan_run` record created with `status=running`

**Expected Result**: Concurrent scans prevented; only one active scan at a time.

---

## Scenario 9: REST API — Scan Report JSON
**ID**: TSR-009  
**Priority**: High  
**Type**: API Test  

**Description**: GET the scan report via REST API and verify JSON response structure.

**Preconditions**:
- At least one completed scan exists with findings

**Steps**:
1. GET `/api/x_adis/scan/report?run_id=<sys_id>` with auth
2. Assert HTTP 200
3. Assert `Content-Type: application/json`
4. Assert response contains `risk_score` (number), `findings_count` (number), `severity_breakdown` (object)
5. Assert `severity_breakdown` has keys: `critical`, `high`, `medium`, `low`

**Expected Result**: Valid JSON response with all expected fields.

---

## Scenario 10: REST API — CSV Export
**ID**: TSR-010  
**Priority**: Medium  
**Type**: API Test  

**Description**: Export findings as CSV via REST API.

**Steps**:
1. GET `/api/x_adis/scan/report?run_id=<sys_id>&format=csv` with auth
2. Assert HTTP 200
3. Assert `Content-Type` includes `text/csv`
4. Assert first line contains headers: `sys_id`, `rule_name`, `severity`, `table_name`
5. Assert at least 1 data row exists

**Expected Result**: Valid CSV with headers and data rows.

---

## Scenario 11: RBAC — Scanner Cannot Modify Rules
**ID**: TSR-011  
**Priority**: High  
**Type**: Security Test  

**Description**: User with `x_adis.scanner` role cannot create or modify deprecation rules.

**Steps**:
1. Authenticate as `x_adis.scanner` user
2. Attempt POST to create new `x_adis_deprecation_rule` record
3. Assert HTTP 403 or error
4. Attempt PUT to modify existing rule → assert HTTP 403

**Expected Result**: Scanner role denied write access to rule table.

---

## Scenario 12: RBAC — Report Reader Cannot Run Scan
**ID**: TSR-012  
**Priority**: High  
**Type**: Security Test  

**Description**: User with `x_adis.report_reader` role can read reports but cannot trigger scans.

**Steps**:
1. Authenticate as `x_adis.report_reader` user
2. Attempt POST to `/api/x_adis/scan` → assert HTTP 403
3. GET `/api/x_adis/scan/report?run_id=<sys_id>` → assert HTTP 200

**Expected Result**: Read access to reports; write access denied.

---

## Pass/Fail Criteria

| Scenario | ID | Status | Notes |
|---|---|---|---|
| Rule Registry — Load All Active Rules | TSR-001 | PENDING | |
| Rule Registry — Regex Pattern Validation | TSR-002 | PENDING | |
| Scanner — Full Scan Execution | TSR-003 | PENDING | |
| Scanner — Finding Creation for Deprecated API | TSR-004 | PENDING | |
| Risk Score Calculation | TSR-005 | PENDING | |
| Delta Scan — Only Reports Changes | TSR-006 | PENDING | |
| Delta Scan — Reports New and Changed Findings | TSR-007 | PENDING | |
| Concurrent Scan Prevention | TSR-008 | PENDING | |
| REST API — Scan Report JSON | TSR-009 | PENDING | |
| REST API — CSV Export | TSR-010 | PENDING | |
| RBAC — Scanner Cannot Modify Rules | TSR-011 | PENDING | |
| RBAC — Report Reader Cannot Run Scan | TSR-012 | PENDING | |

**Minimum pass threshold**: 10/12 scenarios must pass for release approval.

---

## Running the Test Suite
```bash
# Python test harness (local)
cd tests
pytest test_adis_rule_registry.py -v
pytest test_adis_scan_e2e.py -v

# ServiceNow ATF (planned)
# Navigate to Automated Test Framework → Suites → ADIS Test Suite → Run
```
