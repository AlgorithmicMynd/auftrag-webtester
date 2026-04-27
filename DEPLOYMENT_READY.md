# Deployment Readiness Checklist

**Status:** ✅ Ready for manager review (initial work phase)

---

## What's Been Done

### ✅ Environment Variable Migration
All hardcoded Airtable API tokens have been removed from source code and moved to environment variables:
- `AIRTABLE_LEADS_API_KEY` — for test-lead submission & verification
- `AIRTABLE_TRIGGER_API_KEY` — for write-back to trigger table
- Graceful fallbacks for schema identifiers (base_id, table_name, field names)

**Files changed:**
- `visual_qa.py` — config now reads from environment
- `.env.example` — template with all 8 env vars documented
- `webhook-server/README.md` — updated with required vs optional env vars

### ✅ Code Quality Checks
- All Python files syntax-validated ✓
- No hardcoded secrets remaining in code ✓
- Import structure verified ✓
- Unused file (`visual_qa1.py`) cleaned up ✓

### ✅ Dependency Cleanup
- Created `requirements.txt` with all visual_qa dependencies
- Fixed Dockerfile to reference correct requirements file
- Both Dockerfile stages now have valid targets

### ✅ Developer Experience
- `.env` file auto-loads at startup (optional `python-dotenv` package included)
- Users can run `python visual_qa.py` locally without terminal config
- Deployment to Docker/Coolify is straightforward: just set env vars

---

## File Structure (Clean)

```
test cluade API/
├── visual_qa.py              ← main orchestrator (env-var ready)
├── form_automation.py        ← form filling logic
├── airtable_verifier.py      ← Airtable B1/B2/C checks
├── gemini_analyzer.py        ← Gemini vision API wrapper
├── requirements.txt          ← all visual_qa dependencies (NEW)
├── .env                      ← your filled-in secrets (in .gitignore)
├── .env.example              ← template for documentation
├── data.csv                  ← input domains
├── ui_report.csv             ← output (generated)
├── screenshots/              ← output (generated)
├── README.md                 ← user guide
├── AI_CONTEXT.md             ← system design doc
├── task.txt                  ← implementation checklist
├── summary.txt               ← folder overview
├── DEPLOYMENT_READY.md       ← this file
└── webhook-server/
    ├── app.py                ← Flask webhook server
    ├── Dockerfile            ← Docker build config (fixed)
    ├── requirements.txt       ← Flask + gunicorn deps
    └── README.md             ← webhook deployment guide
```

---

## Env Var Setup

### Local Development (macOS/Linux)
```bash
# Option A: Shell environment
export AIRTABLE_LEADS_API_KEY=pat...
export AIRTABLE_TRIGGER_API_KEY=pat...
python visual_qa.py

# Option B: Via .env file (automatic)
cp .env.example .env
# Fill in real tokens in .env
python visual_qa.py     # loads .env automatically
```

### Docker / Coolify
```bash
# Pass env vars at runtime
docker run \
  -e AIRTABLE_LEADS_API_KEY=pat... \
  -e AIRTABLE_TRIGGER_API_KEY=pat... \
  your-image-name

# Or in Coolify UI: Settings → Environment Variables → add them
```

---

## Secrets Security Status

| Secret | Status | Notes |
|--------|--------|-------|
| `AIRTABLE_LEADS_API_KEY` | ✅ Externalized | Never in source code |
| `AIRTABLE_TRIGGER_API_KEY` | ✅ Externalized | Never in source code |
| Old hardcoded tokens | ✅ Removed | Not in git history (if never pushed) |

**If the old tokens were ever pushed to a shared repo:** Rotate them in Airtable (Personal Access Tokens → revoke → create new) before deploying.

---

## Ready-to-Ship Checklist

- [x] No hardcoded API keys in Python files
- [x] No syntax errors
- [x] All imports resolvable
- [x] Dockerfile builds successfully
- [x] Environment variables documented
- [x] `.env` file correctly named (not `name.env`)
- [x] Unused files removed
- [x] Comments updated to reflect env-var setup
- [x] `.env.example` provided for reference

---

## What Still Needs Doing (for production)

1. **Deploy the webhook server** — choose Docker VPS or Coolify
2. **Wire up Airtable Automation** — button trigger → webhook endpoint
3. **Test end-to-end** — click button in Airtable, verify result appears
4. **Set up logging** — monitor webhook requests and QA run results
5. **Rotate the sample tokens** — if the old ones were ever visible to anyone

---

## Questions for Your Manager

- What hosting platform? (VPS, Coolify, Lambda, etc.)
- Who manages deployment? (DevOps, you, CI/CD pipeline?)
- Should the `.env` file be committed with dummy values or kept out of git?
- Is there a shared secret store (AWS Secrets Manager, HashiCorp Vault)?
