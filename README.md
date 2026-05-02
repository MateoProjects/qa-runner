# qa-runner

A zero-dependency manual QA tool — one HTML file, no build step, no server. Load a JSON suite, run through tests step-by-step, save a JSON + HTML report.

**[→ Open `qa-tool.html` in Chrome or Edge to start](qa-tool.html)**

---

## How it works

1. Open `qa-runner.html` in Chrome or Edge (Firefox works but File System Access API is unavailable — files download instead)
2. Drop one or more suite JSON files onto the load screen (or click **Select files**)
3. Choose which tests to run
4. Mark each step **Pass / Fail / Skip** — keyboard shortcuts P / F / S
5. Click **Save results** → writes `YYYY-MM-DD_HH-mm_<suite>.json` + `_report.html` to `qa/results/`

Supports Catalan, Spanish, and English (language persists across sessions).

---

## Suite JSON format

A suite file describes a product area. It has sections, each section has tests, each test has steps.

```json
{
  "suite": "Hub",
  "version": "2.0",
  "sections": [
    {
      "id": "auth",
      "title": "Authentication",
      "tests": [
        {
          "id": "auth-login",
          "title": "Login flow",
          "steps": [
            { "id": "s1", "description": "Login page loads correctly" },
            { "id": "s2", "description": "Valid credentials → redirects to dashboard" },
            { "id": "s3", "description": "Wrong password → shows error" },
            { "id": "s4", "description": "Unknown user → shows error" },
            { "id": "s5", "description": "Empty form → shows validation" }
          ]
        },
        {
          "id": "auth-session",
          "title": "Logout and session",
          "steps": [
            { "id": "s1", "description": "Logout redirects outside the dashboard" },
            { "id": "s2", "description": "After logout the panel requires login" }
          ]
        }
      ]
    },
    {
      "id": "dashboard",
      "title": "Dashboard",
      "tests": [
        {
          "id": "dashboard-layout",
          "title": "Layout and components",
          "steps": [
            { "id": "s1", "description": "Dashboard title visible" },
            { "id": "s2", "description": "4 stat cards visible" },
            { "id": "s3", "description": "Sidebar visible and functional" }
          ]
        }
      ]
    }
  ]
}
```

### Schema rules

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `suite` | string | yes | Display name shown in the UI and report |
| `version` | string | no | e.g. `"2.0"` — shown as a badge |
| `sections[].id` | string | yes | Unique within the file, no spaces |
| `sections[].title` | string | yes | Shown as section header |
| `tests[].id` | string | yes | Unique within the file |
| `tests[].title` | string | yes | Shown as card header |
| `steps[].id` | string | yes | Unique within the test (`s1`, `s2`, …) |
| `steps[].description` | string | yes | One sentence — what the tester must verify |

---

## Generating suites with AI

You can ask an LLM to produce a suite JSON automatically. Paste the schema above and a description of your product, then use a prompt like:

> **Prompt template:**
>
> Generate a QA suite JSON for [product name / feature].
> Use the following schema exactly:
>
> - Top-level keys: `suite` (string), `version` (string), `sections` (array)
> - Each section: `id` (slug), `title`, `tests` (array)
> - Each test: `id` (slug), `title`, `steps` (array)
> - Each step: `id` (`s1`, `s2`, …), `description` (one sentence, tester point of view)
>
> Product context: [paste your spec, user stories, or feature description here]
>
> Group tests by functional area (e.g. Authentication, Dashboard, API Endpoints).
> Each step must describe a single verifiable action — what the tester does and what they expect to see.
> Return only valid JSON, no prose.

### Example AI output for a user authentication module

```json
{
  "suite": "Auth Module",
  "version": "1.0",
  "sections": [
    {
      "id": "registration",
      "title": "Registration",
      "tests": [
        {
          "id": "reg-happy-path",
          "title": "Successful registration",
          "steps": [
            { "id": "s1", "description": "Navigate to /register — page loads with form" },
            { "id": "s2", "description": "Fill valid name, email, password and submit → redirected to dashboard" },
            { "id": "s3", "description": "Welcome email appears in the test inbox within 30 s" }
          ]
        },
        {
          "id": "reg-validation",
          "title": "Form validation",
          "steps": [
            { "id": "s1", "description": "Submit empty form → all required fields highlighted" },
            { "id": "s2", "description": "Enter mismatched passwords → inline error shown" },
            { "id": "s3", "description": "Enter already-registered email → duplicate error shown" }
          ]
        }
      ]
    },
    {
      "id": "login",
      "title": "Login",
      "tests": [
        {
          "id": "login-happy-path",
          "title": "Successful login",
          "steps": [
            { "id": "s1", "description": "Navigate to /login — page loads" },
            { "id": "s2", "description": "Enter valid credentials → redirected to /dashboard" },
            { "id": "s3", "description": "User name visible in the top navigation bar" }
          ]
        },
        {
          "id": "login-errors",
          "title": "Login error states",
          "steps": [
            { "id": "s1", "description": "Wrong password → error message shown, no redirect" },
            { "id": "s2", "description": "Unknown email → error message shown" },
            { "id": "s3", "description": "Rate limit: 5 failed attempts → account temporarily locked" }
          ]
        }
      ]
    }
  ]
}
```

---

## Files
Example of folder for a qa_runner tool
```
qa/
├── qa-tool.html          ← the tool (open this)
├── suites/
│   ├── software.json         ← software suite
│   └── e2e.json          ← End-to-end workflow
└── results/              ← generated reports land here
```

---

## Tips

- **Run multiple suites together** — drop all three JSON files at once; the tool merges them into one run.
- **Keyboard-first workflow** — use P / F / S + Arrow keys to fly through steps without touching the mouse.
- **Failed steps** — entering a reason is optional but shows up highlighted in the HTML report.
- **The tool is offline-first** — no internet required after the file is open.
