# Website QA Tester

An automated QA tool that audits websites for **UI issues** (overlapping elements, text cutoff, layout problems, misalignment) and verifies **Airtable integration** with the website's contact form. Accepts a single website URL or a CSV file containing multiple website domains for batch processing.

Built primarily for auditing German cleaning service landing pages, but the architecture is generalizable to any website.

---

## What It Does

The tool runs a **4-stage automated pipeline** on each website:

| Stage | What It Checks |
|-------|----------------|
| **1. Screenshot Capture** | Opens the website in a headless Chromium browser (1440x900 viewport), bypasses bot detection, and takes a full-page screenshot. |
| **2. Visual Analysis (AI)** | Sends the screenshot to Google Gemini 1.5 Flash to detect overlapping elements, text cutoff, layout breaks, and misalignment. Returns PASS or FAIL. *(Optional -- requires API key.)* |
| **3. Contact Form Test** | Navigates to the contact/Kontakt section, fills out all form fields with test data, and attempts submission. Detects success via URL redirect or on-page confirmation message. |
| **4. Airtable Verification** | Three-part check: **(B1)** Confirms the frontend JavaScript handler (`airtable-form-handler.js`) exists and is correctly configured for the domain. **(B2)** Bypasses the website form entirely and submits a test record directly to Airtable via REST API. **(C)** Polls Airtable to verify the record actually landed in the table. |

### Why the Airtable check uses the API directly

Website contact forms are often protected by CAPTCHA/Turnstile, which prevents automated clicking of the submit button. Instead of trying to solve the CAPTCHA, the tool reads the Airtable configuration from the website's JavaScript and then calls the Airtable API directly with the same credentials. This confirms the integration is wired up correctly end-to-end without needing to bypass anti-bot protections.

### Final Status

Each website receives a final verdict:

- **PASS** -- All enabled checks passed.
- **PARTIAL** -- Some checks passed, others failed or were skipped.
- **FAIL** -- Critical checks failed.

---

## Project Structure

```
.
├── gui.py                  # GUI application (PySimpleGUI) -- recommended entry point
├── visual_qa.py            # Main QA orchestrator (4-stage pipeline)
├── form_automation.py      # Contact form navigation and submission logic
├── airtable_verifier.py    # Airtable frontend check, API submission, and verification
├── gemini_analyzer.py      # Google Gemini vision API wrapper
├── requirements_gui.txt    # Python dependencies
├── run_gui.bat             # Windows one-click launcher (installs deps automatically)
├── QUICK_START.txt         # Short setup guide
├── data.csv                # Input: list of domains to test
├── ui_report.csv           # Output: test results per domain
└── screenshots/            # Output: full-page screenshots per domain
```

---

## How to Start

### Option A: GUI (Recommended)

Double-click **`run_gui.bat`** -- it automatically installs dependencies and launches the GUI.

Or run manually:

```bash
python gui.py
```

The GUI provides:
- A text field to enter a single domain (e.g. `fensterreinigung-ulm.de`)
- A file browser to select a CSV file with multiple domains
- A "RUN TEST" button that runs the pipeline in the background
- A progress bar and live output console
- Buttons to open the generated report (`ui_report.csv`) and screenshots folder

### Option B: Command Line

```bash
python visual_qa.py
```

This reads domains from `data.csv` (must have a `domain` column) in the same directory and writes results to `ui_report.csv`.

---

## Input Format

**Single domain (GUI):** Enter the bare domain, e.g. `fensterreinigung-ulm.de`. The tool adds `https://` automatically.

**CSV file (GUI or CLI):** A CSV with a `domain` column:

```csv
domain
fensterreinigungbruchsal.de
fensterreinigungstuttgart.de
fensterreinigungulm.de
```

---

## Output

### Report (`ui_report.csv`)

| Column | Description |
|--------|-------------|
| `domain` | Website tested |
| `status` | Final verdict: PASS, PARTIAL, or FAIL |
| `screenshot` | Path to the captured screenshot |
| `gemini_status` | PASS, FAIL, SKIPPED, or ERROR |
| `gemini_issues` | Pipe-separated list of detected visual issues |
| `form_status` | PASS or FAIL |
| `form_error` | Error details if form test failed |
| `airtable_linked` | Whether the frontend JS handler is correctly configured |
| `api_submission` | Whether a test record was successfully submitted to Airtable |
| `airtable_verified` | Whether the submitted record was found in Airtable |
| `error` | Any exception that occurred during testing |

### Screenshots (`screenshots/`)

Full-page PNG screenshots of each tested website, named by domain.

---

## Setup Requirements

### For Developers (Technical Users)

**Prerequisites:**
- **Python 3.8+** -- [Download from python.org](https://www.python.org/downloads/)
- **pip** -- Comes bundled with Python
- **Git** (optional) -- Only if cloning from a repository

**Installation:**

```bash
# 1. Install Python dependencies
pip install -r requirements_gui.txt

# 2. Install the Playwright browser engine (one-time)
playwright install chromium
```

**Dependencies installed by `requirements_gui.txt`:**

| Package | Version | Purpose |
|---------|---------|---------|
| `PySimpleGUI` | >= 4.60.5 | Desktop GUI framework |
| `pandas` | >= 1.3.0 | CSV reading and writing |
| `playwright` | >= 1.40.0 | Headless browser automation |
| `requests` | >= 2.28.0 | HTTP requests (Airtable API calls) |
| `playwright-stealth` | >= 1.0.1 | Anti-bot detection bypass |
| `google-generativeai` | >= 0.3.0 | Google Gemini vision API client |
| `Pillow` | >= 9.0.0 | Image processing |

**Optional environment variable:**

```bash
# Enables AI-powered visual analysis of screenshots
# If not set, this stage is simply skipped
set GEMINI_API_KEY=your_google_gemini_api_key
```

You can get a Gemini API key from [Google AI Studio](https://aistudio.google.com/apikey).

---

### For Non-Technical Users (Marketing, QA Teams)

You do **not** need to install anything manually if a developer has already set up the machine. Just:

1. **Double-click `run_gui.bat`** -- this handles everything automatically.
2. Enter a domain or select a CSV file.
3. Click **RUN TEST**.
4. Click **Open Report** when done.

**If this is a fresh machine with nothing installed:**

1. **Install Python:**
   - Go to [python.org/downloads](https://www.python.org/downloads/)
   - Download the latest Python installer for Windows
   - **Important:** Check the box that says "Add Python to PATH" during installation
   - Click "Install Now"

2. **Run the tool:**
   - Double-click **`run_gui.bat`** in the project folder
   - It will automatically install all required packages on first run
   - If you see an error about Playwright browsers, open Command Prompt in the project folder and type:
     ```
     playwright install chromium
     ```
     Then double-click `run_gui.bat` again.

That's it. No code editor, terminal knowledge, or Git required.

---

## Configuration

All configuration is hardcoded in the source files. There are no `.env` files to manage.

| Setting | Location | Default |
|---------|----------|---------|
| Airtable API Key | `visual_qa.py` line ~80 | Hardcoded |
| Airtable Base ID | `visual_qa.py` line ~80 | `appL4PpAWoTl3rEzE` |
| Airtable Table Name | `visual_qa.py` line ~80 | `Leads Partner` |
| Input CSV path | `visual_qa.py` line ~70 | `data.csv` |
| Output CSV path | `visual_qa.py` line ~71 | `ui_report.csv` |
| Browser viewport | `visual_qa.py` | 1440 x 900 |
| Navigation timeout | `visual_qa.py` | 30 seconds |
| Gemini API Key | Environment variable | *(none -- stage skipped if unset)* |
| Test form data | `form_automation.py` line ~28 | Company: "QA Automation", Name: "Test User", etc. |

To change the Airtable configuration or test data, edit the relevant Python file directly.

---

## How It Works Under the Hood

### Form Navigation Strategy

The form automation module uses a three-strategy fallback to find the contact form:

1. **Click navigation link** -- Looks for nav links matching "kontakt", "contact", or "anfrage"
2. **Try direct URLs** -- Navigates to `/kontakt`, `/contact`, etc.
3. **Scroll to anchor** -- Looks for `#kontakt`, `#contact` anchors on the page

### Airtable Verification Flow

```
B1: Check frontend JS handler exists
    └── Fetch /js/airtable-form-handler.js
    └── Verify collectFormData function present
    └── Verify domain matches

B2: Submit test record via API
    └── POST to Airtable REST API
    └── Uses unique email: qa+{run_id}@test.com

C:  Verify record landed
    └── Query Airtable with FIND() formula
    └── Retry 3 times with 4-second delays
```

### Bot Detection Bypass

The tool uses `playwright-stealth` to avoid being blocked by anti-bot systems. It spoofs the Chrome runtime, hides the WebDriver flag, patches the Permissions API, and sets consistent User-Agent headers. Each domain gets an isolated browser context with separate cookies and cache.

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| "Python is not installed" error | Install Python 3.8+ from python.org. Check "Add to PATH" during install. |
| Playwright browser not found | Run `playwright install chromium` in Command Prompt. |
| GUI doesn't open | Run `python gui.py` from Command Prompt to see error details. |
| Website times out | The default timeout is 30 seconds. Slow websites may need a higher timeout (edit `NAVIGATION_TIMEOUT` in `visual_qa.py`). |
| Airtable check fails | Verify the API key and base ID in `visual_qa.py` are correct and the table exists. |
| Gemini analysis shows "SKIPPED" | This is normal if `GEMINI_API_KEY` environment variable is not set. The tool works without it. |
| Form test fails with "Submit button not found" | The website's form structure may not match expected selectors. Check form field names in `form_automation.py`. |
