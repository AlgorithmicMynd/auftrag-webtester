# Code Changes Summary

All changes made to implement error field tracking and prepare for Coolify deployment.

---

## FILES CHANGED

### 1. **webhook-server/app.py** ✅ FIXED

**What changed:**
- Line 72: Fixed `airtable_record_id=None` → `airtable_record_id=record_id`
  - **Why:** The record_id must be passed to the pipeline so it can write results back to Airtable
- Line 72-81: Updated error handling to pass `error_msg` parameter
- Line 83-91: Added error message capture from result dict

**Impact:**
- Webhook now correctly passes the record ID to the QA pipeline
- Error messages are properly captured and sent to Airtable
- The "Kontakt error" field gets populated with error details

---

### 2. **visual_qa.py** ✅ UPDATED

**Change 1 — Config (Line 100-115):**
```python
# Added:
"error_field": os.environ.get("AIRTABLE_TRIGGER_ERROR_FIELD", "Kontakt error"),
```
- Reads the error field name from environment variable
- Defaults to "Kontakt error"

**Change 2 — write_qa_result_to_airtable() function (Line 343-380):**
- Added `error_msg: str = ""` parameter
- Builds payload to include error field if message provided
- Limits error message to 500 characters (Airtable text field limit)

**Change 3 — run_domain() function (Line 655-661):**
```python
if airtable_record_id and AIRTABLE_TRIGGER_CONFIG.get("api_key") and AIRTABLE_TRIGGER_CONFIG.get("base_id"):
    error_msg = result.get("error", "") or result.get("form_error", "")
    write_qa_result_to_airtable(airtable_record_id, result["status"], error_msg=error_msg)
```
- Now extracts error details from the result dict
- Passes them to the write-back function

**Impact:**
- All errors are now tracked and stored in Airtable
- Sales team can see exactly what failed: screenshot analysis issues, form problems, API errors, etc.

---

### 3. **.env** ✅ UPDATED

**Added:**
```
AIRTABLE_TRIGGER_ERROR_FIELD=Kontakt error
```

**Impact:**
- Environment variable tells the code which Airtable field to use for error messages
- Can be overridden in Coolify settings if the field name is different

---

### 4. **COOLIFY_DEPLOYMENT.md** ✨ NEW FILE

Complete step-by-step guide for:
- Preparing Git repository
- Deploying to Coolify
- Configuring Airtable automation
- Testing the workflow
- Troubleshooting

---

## FLOW DIAGRAM (After Changes)

```
┌─────────────────────────────────────────────────────────────┐
│ Sales Employee Changes Kontakt status → "to be tested"      │
└────────────────────────┬────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────────┐
│ Airtable Automation:                                        │
│ 1. Set Kontakt status → "testing" (lock)                    │
│ 2. POST /run-qa { record_id, domain }                       │
└────────────────────────┬────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────────┐
│ Webhook Server (app.py):                                    │
│ 1. Receive POST /run-qa                                     │
│ 2. Call run_domain(domain, record_id)                       │
└────────────────────────┬────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────────┐
│ QA Pipeline (visual_qa.py):                                 │
│ 1. Screenshot → Gemini → Form test → Airtable checks        │
│ 2. Collect status & errors                                  │
│ 3. Call write_qa_result_to_airtable(record_id, status,      │
│    error_msg=error_msg)                                     │
└────────────────────────┬────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────────┐
│ Write-Back to Airtable:                                     │
│ PATCH record:                                               │
│   Kontakt status → "passed" or "failed"                     │
│   Kontakt error  → error message (if any)                   │
└────────────────────────┬────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────────┐
│ Sales Employee Sees:                                        │
│ ✅ Final result (passed/failed)                             │
│ ℹ️  Error details (if it failed)                            │
│ 📸 Screenshot in related records (if integrated)            │
└─────────────────────────────────────────────────────────────┘
```

---

## ENVIRONMENT VARIABLES SUMMARY

All variables needed in Coolify:

| Variable | Value | Purpose |
|----------|-------|---------|
| `AIRTABLE_LEADS_API_KEY` | `pat...` | Submit test leads to Leads Partner table |
| `AIRTABLE_LEADS_BASE_ID` | `appL4PpAWoTl3rEzE` | Base for test leads |
| `AIRTABLE_LEADS_TABLE` | `Leads Partner` | Table name |
| `AIRTABLE_TRIGGER_API_KEY` | `pat...` | Write results to EMD Webseiten |
| `AIRTABLE_TRIGGER_BASE_ID` | `apphwncsSpj5PTIFX` | Base for trigger table |
| `AIRTABLE_TRIGGER_TABLE` | `EMD Webseiten` | Sales team's table |
| `AIRTABLE_TRIGGER_DOMAIN_FIELD` | `Domain` | Field containing website URL |
| `AIRTABLE_TRIGGER_RESULT_FIELD` | `Kontakt status` | Field for passed/failed |
| `AIRTABLE_TRIGGER_ERROR_FIELD` | `Kontakt error` | Field for error messages |
| `GEMINI_API_KEY` | (optional) | Google Gemini API key |

---

## WHAT'S READY NOW

✅ **Webhook server** — receives requests and triggers QA pipeline  
✅ **Error tracking** — captures and writes error messages to Airtable  
✅ **Write-back system** — updates both "Kontakt status" and "Kontakt error" fields  
✅ **Environment variables** — all configurable, no secrets in source code  
✅ **Deployment guide** — step-by-step Coolify + Airtable setup  

---

## NEXT IMMEDIATE STEPS

1. **Deploy to Coolify** — Follow COOLIFY_DEPLOYMENT.md Part 1-2
2. **Configure Airtable Automation** — Follow COOLIFY_DEPLOYMENT.md Part 3
3. **Run test** — Change one record to "to be tested" and verify it works
4. **Monitor logs** — Check Coolify logs for any errors during first run
5. **Celebrate** — Your QA automation is live! 🎉

---

## QUESTIONS?

Check the troubleshooting section in **COOLIFY_DEPLOYMENT.md** for common issues.
