# Swytchcode Evaluation: Customer Onboarding Agent & API Execution

## 1. Problem and Intended User
- **Target User**: AI agent developers, backend engineers, and automation teams building multi-service autonomous workflows.
- **Problem**: Modern LLM agents struggle to interact reliably with 3rd-party SaaS APIs (Stripe, HubSpot, Resend). SDKs are bloated, traditional REST endpoints lack runtime input/output validation, and exposing raw API keys directly to agent runtimes introduces major security and execution reliability risks.
- **Solution with Swytchcode**: Swytchcode acts as an **execution authority kernel** for AI agents. By utilizing contract-defined **Wrekenfiles** (`wrekenfile.yaml`) and an isolated `swytchcode_runtime`, agents can discover, execute, and validate API operations with determinism, strict type-checking, and zero direct credential leakage.

---

## 2. API & Integration Explored
- **Provider**: **Stripe** (`stripe.stripe@2026-02-25.clover`)
- **Primary Operation**: `stripe.create_payment` (Demo Mode) and `customers.customer.create`
- **Secondary Integrations**:
  - `hubspot.crm.contacts.create` (HubSpot CRM)
  - `resend.email.create` (Resend Email API)
- **Workflow Orchestration**: LangGraph state machine coordinating customer creation, CRM indexing, and onboarding email delivery.

---

## 3. Setup and Commands

### Prerequisites
- Node.js v18+ (tested on Node.js v22.18.0)
- Python 3.10+
- Swytchcode CLI (`swytchcode` / `swy` v2.25.1)

### Installation & Initialization Steps
```bash
# 1. Install Swytchcode CLI globally
npm install -g swytchcode

# 2. Check binary installation
swytchcode --version

# 3. Initialize workspace (non-interactive mode)
swytchcode init --editor cursor --mode sandbox --non-interactive

# 4. Explore available integrations
swytchcode search stripe
swytchcode demo list

# 5. Download a working example project
swytchcode examples --name customer.onboarding --framework langgraph
```

---

## 4. What Worked
1. **CLI Installation via NPM**: `npm install -g swytchcode` successfully retrieved version 2.25.1 and installed the native Windows x64 binary `swytchcode-cli-win32-x64/bin/swytchcode.exe`.
2. **Project Initialization**: `swytchcode init --editor cursor --mode sandbox --non-interactive` successfully generated `.swytchcode/`, `.cursor/rules/swytchcode.mdc`, and initial configuration files.
3. **Demo API Execution**: Executed `swytchcode demo stripe` which successfully called the live demo sandbox:
   ```text
   ✔ Payment created successfully
     amount: $20.00
     currency: USD
     status: requires_payment_method
     payment_id: pi_3UXOGxRsPvFZeq6G1tsUuJb5
   ```
4. **Wrekenfile Inspection**: Successfully explored the multi-thousand-line `wrekenfile.yaml` specifications for Stripe, HubSpot, and Resend within `.swytchcode/integrations/`.
5. **Python Runtime Integration**: Installed `pip install swytchcode-runtime` and verified deterministic call structures via `swytchcode_runtime.exec()`.

---

## 5. What Did Not Work & Friction Points (Errors Encountered)
1. **Interactive Prompt Hang on Non-TTY Execution**:
   - *Error*: Running `swy init` or `swy --version` in non-interactive terminal sessions hung because the Node.js wrapper (`swytchcode.js`) blocked on `readAllStdin()` awaiting EOF.
   - *Fix*: Called the underlying native binary directly or passed explicit non-interactive flags (`--non-interactive --editor cursor --mode sandbox`).
2. **PostHog Telemetry Timeouts**:
   - *Error*: `posthog WARN: shutdown timeout exceeded... context deadline exceeded`. Every CLI command attempted an external telemetry sync before exiting, causing an unwanted 15–30s delay.
3. **Registry `get` Command Required Authentication**:
   - *Error*: `error: authentication required for get: not authenticated - run swytchcode login`. Running `swytchcode get <provider>` blocked unauthenticated users from pulling public bundles, whereas `swytchcode examples` bypassed this smoothly.
4. **Manifest Version Mismatch in Doctor Check**:
   - *Error*: `swytchcode doctor` reported: `[manifest_version:stripe.stripe@2026-02-25.clover] installed version is missing from manifest.json`. The downloaded example bundle had `stripe.stripe_sdk` in `manifest.json` while `tooling.json` referenced `stripe.stripe`.

---

## 6. How to Reproduce
1. Clone this repository:
   ```bash
   git clone <repo-url>
   cd swytchcode-ppi-submission
   ```
2. Install Python dependencies:
   ```bash
   pip install -r requirements.txt
   ```
3. Test local execution with the demo provider:
   ```bash
   swytchcode demo stripe
   ```
4. Run the customer onboarding agent:
   ```bash
   python main.py
   ```

---

## 7. Notes on the Generated Wrekenfile (`wrekenfile.yaml`)
The Wrekenfile is the core contractual schema powering Swytchcode:
- **Location**: `.swytchcode/integrations/{provider}/{library}/{version}/wrekenfile.yaml`
- **Structure**:
  - `VERSION`: Schema specification format (e.g. `2.0.2`).
  - `DEFAULTS`: Base URLs and authentication schemes (`bearer_token`, `basic_auth`).
  - `METHODS`: Granular endpoint definitions (e.g., `customers.customer.create`, `payment_intents.payment_intent.create`).
  - `INPUTS / RETURNS`: Strict type definitions (`STRUCT`, `[]STRING`, `INT`, `OBJECT`) with parameter locations (`query`, `body`, `header`).
- **Advantage for AI Agents**: Rather than guessing parameter shapes from unstructured OpenAPI docs, the Wrekenfile provides an unambiguous, deterministic contract that the runtime verifies before sending requests over the wire.
