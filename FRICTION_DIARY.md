# Swytchcode First-Time Onboarding Friction Diary

This log details the specific friction points, roadblocks, expected vs. actual behaviors, and resolutions encountered during initial onboarding.

---

### Friction Point 1: CLI Wrapper Hang on Windows (Non-TTY / Subprocess Execution)
- **Step Attempting**: Checking CLI version and running help commands (`swy --version`, `swytchcode --help`).
- **Command**: `swy --version` / `node bin/swytchcode.js --help`
- **What was Expected**: Instant printout of the CLI version (e.g., `swytchcode version 2.25.1`) and available command flags.
- **What Actually Happened**: Process hung indefinitely with 0% CPU without returning control or output.
- **Root Cause Discovered**: In `bin/swytchcode.js` line 125, the Node.js wrapper executes `readAllStdin()` whenever `!process.stdin.isTTY`. The `Atomics.wait()` loop waits for an EOF token that standard automated subprocess runners or Windows background tasks don't emit synchronously.
- **Next Steps & Resolution**: Located the underlying Go-compiled executable at:
  `%APPDATA%\npm\node_modules\swytchcode\node_modules\swytchcode-cli-win32-x64\bin\swytchcode.exe`
  and executed it directly, which yielded instantaneous `<10ms` response times.
- **Recommendation**: Add a check in `bin/swytchcode.js` to skip `readAllStdin()` when flags like `--version` or `--help` are supplied, or implement a 200ms read timeout.

---

### Friction Point 2: PostHog Telemetry Timeouts & Process Exit Delays
- **Step Attempting**: Executing standard CLI commands (`swytchcode demo list`, `swytchcode demo stripe`).
- **Command**: `swytchcode demo stripe`
- **What was Expected**: Command executes quickly and exits immediately after displaying output.
- **What Actually Happened**: Command succeeded, but hung for 15–20 seconds before exiting with:
  ```text
  posthog WARN: shutdown timeout exceeded, some messages may be lost
  posthog WARN: sending request - Post "https://us.i.posthog.com/batch/": context deadline exceeded
  posthog ERROR: 1 messages dropped after 1 attempts
  ```
- **Next Steps & Resolution**: Identified that outbound network requests to PostHog were blocking the process exit loop. Setting `SWYTCHCODE_TELEMETRY=false` or providing an explicit timeout alleviates this in restricted/firewalled network environments.
- **Recommendation**: Make telemetry asynchronous in a detached daemon or reduce the shutdown flush timeout from 15s to 1s so developer workflow is never blocked.

---

### Friction Point 3: Unclear Mandatory Flags for `swytchcode init`
- **Step Attempting**: Initializing a clean project in a new directory.
- **Command**: `swytchcode init`
- **What was Expected**: CLI prompts with a default editor (or initializes without editor config).
- **What Actually Happened**:
  - Attempt 1: `error: init requires --editor when running non-interactively`
  - Attempt 2 (`--editor cursor`): `error: init requires --mode when running non-interactively`
- **Next Steps & Resolution**: Consulted `swytchcode init --help` and ran:
  `swytchcode init --editor cursor --mode sandbox --non-interactive` which succeeded immediately.
- **Recommendation**: Provide smart defaults for `--editor` (e.g. `none`) and `--mode` (e.g. `sandbox`) when `--non-interactive` is passed instead of erroring out.

---

### Friction Point 4: Authentication Barrier on Public Integration Bundles (`swytchcode get`)
- **Step Attempting**: Fetching provider bundles as suggested in the post-init guide (`swytchcode get Stripe`).
- **Command**: `swytchcode get Stripe`
- **What was Expected**: Downloads public Stripe Wrekenfile schemas into `.swytchcode/integrations/`.
- **What Actually Happened**:
  ```text
  error: authentication required for get: not authenticated - run `swytchcode login` for registry features
  ```
- **What was Tried Next**: Searched documentation for offline or public templates. Found that `swytchcode examples` allows downloading complete projects (e.g., `customer.onboarding`) that bundle full, pre-downloaded Wrekenfiles without needing prior account login.
- **Recommendation**: Allow anonymous/unauthenticated read-only access for core public integrations (Stripe, GitHub, HubSpot) via `swytchcode get` so new developers can explore Wrekenfiles without signing up first.

---

### Friction Point 5: Manifest Inconsistency in `swytchcode doctor`
- **Step Attempting**: Running diagnostics on the downloaded example project.
- **Command**: `swytchcode doctor`
- **What was Expected**: All checks passing green for official template projects.
- **What Actually Happened**:
  ```text
  × [manifest_version:stripe.stripe@2026-02-25.clover] installed version is missing from manifest.json - run: swytchcode sync
  ```
- **Investigation**: Inspected `.swytchcode/integrations/manifest.json` vs `.swytchcode/tooling.json`. `manifest.json` registered the key as `"stripe.stripe_sdk"`, whereas `tooling.json` referenced `"stripe.stripe"`.
- **Resolution**: Updated the key in `manifest.json` or ran `swytchcode sync` to align provider package names.
- **Recommendation**: Ensure example project generators run `swytchcode doctor` validation prior to publishing templates to the registry.
