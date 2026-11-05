# First-Time Onboarding Friction Diary

**Author:** Prince Kumar  
**Environment:** Windows 11 Home, PowerShell 7 / Windows Terminal, Node.js v22.18.0, Python 3.11  

This diary records the actual issues, errors, and workarounds I encountered while exploring Swytchcode for the first time.

---

### Friction Point 1: CLI Process Hang on Windows (Non-TTY / Subprocess Execution)

- **Step Attempting:** Verifying CLI installation and checking version/help output.
- **Command / Action:** `swytchcode --version` / `swy --help`
- **What I Expected:** The CLI should print the installed version (`swytchcode version 2.25.1`) or help text immediately within 1-2 seconds.
- **What Actually Happened:** The process hung completely in my terminal with no output and zero CPU activity.
- **What I Tried Next:** 
  - Consulted the docs at `https://docs.swytchcode.com/getting-started/installation`.
  - Inspected the wrapper script at `C:\Users\Prince kumar\AppData\Roaming\npm\node_modules\swytchcode\bin\swytchcode.js`.
  - Found that line 125 checks `if (!process.stdin.isTTY)` and calls `readAllStdin()`, which uses an `Atomics.wait()` loop waiting for an EOF. In certain Windows terminal/subprocess environments, this never terminates.
  - Used an AI prompt: *"Why does npm CLI package hang on Atomics.wait in bin/swytchcode.js on Windows?"* The answer confirmed that non-interactive stdio doesn't trigger EOF cleanly.
- **Resolution:** I located the underlying Windows x64 binary directly at:
  `%APPDATA%\npm\node_modules\swytchcode\node_modules\swytchcode-cli-win32-x64\bin\swytchcode.exe`
  and verified it runs immediately. In regular interactive PowerShell sessions, passing input or running the command directly resolved it.
- **Feedback / Suggestion:** In `bin/swytchcode.js`, bypass `readAllStdin()` when flags like `--version` or `--help` are passed, or add a short read timeout (e.g., 100ms) rather than blocking indefinitely.

---

### Friction Point 2: PostHog Telemetry Timeouts Causing 15–20s Exit Delays

- **Step Attempting:** Running built-in demo commands to test sandbox execution.
- **Command / Action:** `swytchcode demo stripe`
- **What I Expected:** The demo should output the created payment object and immediately return control back to the terminal prompt.
- **What Actually Happened:** The payment was created successfully, but the terminal stayed frozen for ~18 seconds before exiting with:
  ```text
  posthog 2026/11/03 07:34:44 WARN: shutdown timeout exceeded, some messages may be lost
  posthog 2026/11/03 07:34:44 WARN: shutdown timeout: 1 batches still in flight
  ```
- **What I Tried Next:** Searched the CLI help options and docs for telemetry settings. Ran the command again to see if it was a temporary network glitch, but the timeout repeated on subsequent runs.
- **Resolution:** The operation itself succeeded, but the CLI was waiting for a PostHog analytics batch to flush before process exit.
- **Feedback / Suggestion:** The shutdown flush timeout should be capped at 1 second or spawned in a detached process so that slow telemetry does not degrade the local developer experience.

---

### Friction Point 3: Missing Required Flags in Non-Interactive `init`

- **Step Attempting:** Initializing a new project workspace.
- **Command / Action:** `swytchcode init`
- **What I Expected:** A simple interactive wizard or safe defaults for a new project directory.
- **What Actually Happened:** In scripted/non-TTY execution, it failed with:
  ```text
  error: init requires --editor when running non-interactively
  ```
  After adding `--editor cursor`, it failed again with:
  ```text
  error: init requires --mode when running non-interactively
  ```
- **What I Tried Next:** Ran `swytchcode init --help` to check all required options.
- **Resolution:** Ran with full flags:
  `swytchcode init --editor cursor --mode sandbox --non-interactive`
  This initialized the project properly.
- **Feedback / Suggestion:** When `--non-interactive` is passed, default `--editor` to `none` (or auto-detect VS Code/Cursor) and default `--mode` to `sandbox` rather than throwing successive errors.

---

### Friction Point 4: Unauthenticated Users Blocked by `swytchcode get`

- **Step Attempting:** Downloading the public Stripe integration bundle as referenced in setup guides.
- **Command / Action:** `swytchcode get Stripe`
- **What I Expected:** It should download the public Stripe Wrekenfile into `.swytchcode/integrations/`.
- **What Actually Happened:**
  ```text
  error: authentication required for get: not authenticated - run `swytchcode login` for registry features
  ```
- **What I Tried Next:** Checked `https://docs.swytchcode.com` to see if public integrations require an account. Then ran `swytchcode examples` to see if templates contained the schemas.
- **Resolution:** Found that `swytchcode examples` downloads sample projects (like `customer.onboarding`) that include pre-packaged Wrekenfiles, bypassing the login requirement.
- **Feedback / Suggestion:** Allow unauthenticated read-only downloads for popular public integrations (Stripe, GitHub, HubSpot) via `swytchcode get` so first-time evaluators can test without signing up first.

---

### Friction Point 5: Manifest Key Inconsistency in `swytchcode doctor`

- **Step Attempting:** Running the project health check on the downloaded example project.
- **Command / Action:** `swytchcode doctor`
- **What I Expected:** Clean pass with all checks green.
- **What Actually Happened:**
  ```text
  × [manifest_version:stripe.stripe@2026-02-25.clover] installed version is missing from manifest.json - run: swytchcode sync
  ```
- **What I Tried Next:** Opened `.swytchcode/integrations/manifest.json` and `.swytchcode/tooling.json` to inspect the package identifiers.
- **Resolution:** Discovered `manifest.json` listed the provider as `"stripe.stripe_sdk"`, whereas `tooling.json` referenced `"stripe.stripe"`. Running `swytchcode sync` or manually aligning the keys resolved the check.
- **Feedback / Suggestion:** Ensure template bundles published to the example repository have passing `swytchcode doctor` checks out-of-the-box.
