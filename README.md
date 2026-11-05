# Swytchcode Developer Exploration & Customer Onboarding Prototype

**Author:** Prince Kumar  
**Submission Date:** November 5, 2026  
**Assignment:** Swytchcode PPI Next Steps  
**Live Reddit Post:** [Reddit Discussion](https://www.reddit.com/user/No-Cranberry9977/comments/1xojlt2/why_i_stopped_giving_ai_agents_raw_api_keys/)  

---

## 1. Problem and Intended User
- **Target User:** Backend developers and AI engineers building agentic workflows (e.g., using LangGraph or AutoGen) that need to call external SaaS APIs.
- **Problem:** When an AI agent needs to call third-party services like Stripe, HubSpot, or Resend, developers usually have to install heavy vendor SDKs, manage raw API keys in application memory, and deal with hallucinated parameters or runtime type errors from the model.
- **How Swytchcode Solves This:** Swytchcode provides a contract-driven execution layer. Instead of bundling dozens of disparate SDKs and exposing raw API keys, integrations are declared in local contract files called **Wrekenfiles** (`wrekenfile.yaml`). The agent invokes standardized operations through a lightweight local runtime (`swytchcode-runtime`), which validates parameters locally and handles authentication securely.

---

## 2. API & Integrations Explored
- **Primary Integration:** **Stripe** (`stripe.stripe@2026-02-25.clover`)
  - Operation: `stripe.create_payment` (tested via `swytchcode demo stripe`)
- **Secondary Integrations (in Wrekenfile):**
  - **HubSpot CRM:** `hubspot.crm.contacts.create` (for contact creation)
  - **Resend:** `resend.email.create` (for onboarding confirmation emails)
- **Prototype Script:** A multi-step onboarding flow (`main.py`) coordinating customer creation, CRM indexing, and onboarding notifications.

---

## 3. Setup and Commands

### Prerequisites
- Node.js (v18+; tested on Node.js v22.18.0)
- Python 3.10+
- OS: Windows 11 (PowerShell)

### Installation & Reproduction
```bash
# 1. Install the CLI globally
npm install -g swytchcode

# 2. Verify installation
swytchcode --version

# 3. Initialize workspace
swytchcode init --editor cursor --mode sandbox --non-interactive

# 4. Run the live demo operation
swytchcode demo stripe

# 5. Install Python dependencies and run the example script
pip install -r requirements.txt
python main.py
```

---

## 4. What Worked
1. **Global CLI Installation:** Installing via `npm install -g swytchcode` pulled version 2.25.1 and placed the Windows binary (`swytchcode-cli-win32-x64\bin\swytchcode.exe`) on my PATH.
2. **Project Initialization:** `swytchcode init` generated the `.swytchcode/` directory, configuration rules, and environment scaffolding.
3. **Sandbox API Execution:** Executing `swytchcode demo stripe` worked out-of-the-box without needing my own Stripe API key, returning a valid payment intent:
   ```text
   description: Demo payment
   created: 2026-11-03T02:04:38.000Z
   payment_id: pi_3UXX5WRsPvFZeq6G0w1kcgHb
   Ran on the demo account. Use your own Stripe key: swytchcode login stripe
   ```
4. **Wrekenfile Contract Model:** Inspecting the downloaded Wrekenfiles in `.swytchcode/integrations/` showed clear, typed specifications for methods, endpoints, required headers, and expected responses.
5. **Python Runtime:** Verified `swytchcode-runtime` in Python to structure deterministic API calls without installing the official Stripe SDK.

---

## 5. What Did Not Work & Friction Points

During my testing on Windows 11, I encountered several practical friction points:

1. **CLI Process Hang on Non-TTY Execution (Windows):**
   - *Issue:* When running commands like `swy --version` in non-interactive terminal contexts or subprocess runners, the command hung indefinitely.
   - *Cause:* In `bin/swytchcode.js` (line 125), `readAllStdin()` uses an `Atomics.wait()` loop waiting for EOF on non-TTY stdin.
   - *Workaround:* Ran commands in an interactive PowerShell prompt or invoked the native binary directly:
     `%APPDATA%\npm\node_modules\swytchcode\node_modules\swytchcode-cli-win32-x64\bin\swytchcode.exe`.
2. **PostHog Telemetry Exit Delay:**
   - *Issue:* Commands often took 15–20 seconds to exit after printing their output, throwing a timeout error:
     `posthog WARN: shutdown timeout exceeded... context deadline exceeded`.
   - *Feedback:* The telemetry flush should be detached or have a much shorter timeout (e.g. 1 second) so it doesn't freeze the developer's terminal.
3. **Non-Interactive Flag Requirements in `init`:**
   - *Issue:* Running `swytchcode init` in CI or scripted environments fails unless both `--editor` and `--mode` are explicitly supplied.
   - *Feedback:* It would be smoother if `--editor none` and `--mode sandbox` were applied as sensible defaults when `--non-interactive` is passed.
4. **Login Requirement on `swytchcode get`:**
   - *Issue:* Running `swytchcode get Stripe` blocked with `error: authentication required for get: not authenticated`.
   - *Workaround:* Used `swytchcode examples` which bundled working Wrekenfiles without needing account authentication.

---

## 6. Notes on the Generated Wrekenfile (`wrekenfile.yaml`)
The Wrekenfile is the core schema format that replaces third-party client libraries:
- **Location:** `.swytchcode/integrations/{provider}/{library}/{version}/wrekenfile.yaml`
- **Key Sections:**
  - `VERSION`: Specifies schema syntax version (e.g. `2.0.2`).
  - `DEFAULTS`: Base URLs, content types, and auth mechanisms (`bearer_token`, `basic_auth`).
  - `METHODS`: Explicit definitions for endpoints (e.g., `customers.customer.create`).
  - `INPUTS` & `RETURNS`: Strict typing (`STRUCT`, `STRING`, `INT`, `OBJECT`) specifying parameter placement (`body`, `query`, `header`).
- **Why It Matters:** Instead of forcing an LLM to guess API payloads from loose natural language or 500-page OpenAPI specs, the Wrekenfile acts as a clear local contract that the runtime validates before making any network call.
