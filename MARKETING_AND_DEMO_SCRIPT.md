# Marketing Assets & 2-Minute Demo Script

**Author:** Prince Kumar  

---

## Option 1: Reddit Post (Target Subreddits: r/MachineLearning, r/LocalLLaMA, r/Python)

**Live Reddit Post:** [View on Reddit](https://www.reddit.com/user/No-Cranberry9977/comments/1xojlt2/why_i_stopped_giving_ai_agents_raw_api_keys/)  
**URL:** `https://www.reddit.com/user/No-Cranberry9977/comments/1xojlt2/why_i_stopped_giving_ai_agents_raw_api_keys/`

**Title:** Why I stopped giving AI agents raw API keys: testing contract-based tool calling with Swytchcode

**Post Body:**
Hey devs,

Anyone building autonomous workflows with tools like LangGraph or AutoGen knows the pain points with API integrations:
1. You have to install dozens of heavy vendor SDKs (Stripe, HubSpot, Resend, etc.) just to execute one or two calls.
2. Models regularly hallucinate parameters or pass malformed types.
3. Exposing live API keys in application memory or agent tool contexts is always risky.

I spent the last couple of days testing **Swytchcode** (https://swytchcode.com), and the approach is pretty neat:

Instead of installing client SDKs, it uses **Wrekenfiles** (`wrekenfile.yaml`) — local declarative contracts defining method endpoints, input types, and required headers. 

How it works in practice:
- `swytchcode init` sets up local boundaries in `tooling.json`.
- `swytchcode demo stripe` lets you test execution in sandbox mode right from your terminal without even needing an API key.
- Inside Python, a single lightweight runtime handles everything:
```python
from swytchcode_runtime import exec as swytchcode_exec

res = swytchcode_exec("stripe.create_payment", {"amount": 2000, "currency": "usd"})
```

The runtime validates the payload against the local contract schema before anything leaves your machine, while keeping credentials isolated.

Has anyone else experimented with contract-driven tool calling for agents instead of full SDKs? Curious to hear how others handle deterministic tool execution.

---

## Option 2: LinkedIn Post

When building multi-agent workflows, the biggest failure point usually isn't the model's reasoning — it's **API execution fragility**.

Between heavyweight vendor SDKs, hallucinated parameter schemas, and the security risk of giving autonomous agents direct access to raw API keys, productionizing agent tools requires a lot of defensive boilerplate.

Over the past few days, I evaluated **Swytchcode**, a developer tool tackling this problem:

🔑 **Key Takeaways from my exploration:**
- **Contract-First Schemas (Wrekenfiles):** Local YAML schemas define explicit API shapes, payload types, and authentication schemes.
- **Zero Credential Exposure:** Agents call standardized operations without holding raw secrets in runtime state.
- **Local Validation:** Tool calls are validated locally before any network request is dispatched, preventing failed or corrupted requests.
- **Unified Runtime:** Replaces multiple disparate vendor SDKs with one clean interface (`swytchcode-runtime`).

Tools that introduce determinism and security boundaries into agentic systems are going to be essential as workflows become more autonomous.

Check out their docs: https://docs.swytchcode.com

#AIAgents #Python #SoftwareEngineering #DevTools #LangGraph #APIs

---

## Option 3: 2–3 Minute Demo Video Script

*Use this script when recording your screen with Loom or OBS.*

**[0:00 - 0:30] Introduction & Problem**
> "Hi, I'm Prince Kumar. In this video, I'm going to demonstrate my exploration of Swytchcode and show how it provides a contract-driven execution layer for AI agents. 
> When we build agents that call external services like Stripe or HubSpot, managing separate SDKs and preventing hallucinated parameters is a constant issue. Let's see how Swytchcode handles this."

**[0:30 - 1:15] CLI & Wrekenfile Overview**
*(Show your VS Code terminal and file tree)*
> "First, I installed the CLI globally via npm. Here in the terminal, running:
> `swytchcode --version` shows version 2.25.1.
> In my project tree, you can see the `.swytchcode` directory. The heart of Swytchcode is the **Wrekenfile** — here in `wrekenfile.yaml`. 
> Unlike a massive OpenAPI document, the Wrekenfile defines precise endpoint methods, expected input structures, and return types right on my local machine."

**[1:15 - 1:55] Live API Execution**
*(In your VS Code terminal, run `swytchcode demo stripe`)*
> "Now let's test a live API call. I'll run `swytchcode demo stripe`.
> Notice the output: it creates a payment intent (`pi_...`) through the demo account without requiring me to configure a Stripe account or expose an API key. The CLI validates the input contract and handles the execution directly."

**[1:55 - 2:35] Python Agent Integration & Key Learning**
*(Show `main.py` in VS Code)*
> "In Python, we don't need the Stripe or HubSpot client libraries. We import `swytchcode-runtime` and execute calls using the validated contract names.
> During onboarding, I kept a detailed friction diary. One interesting discovery was on Windows: the CLI's Node.js wrapper can hang in non-TTY environments due to an stdin wait loop, which I resolved by invoking the underlying binary directly and utilizing interactive shells."

**[2:35 - 3:00] Conclusion**
> "Overall, Swytchcode makes agent tool calling deterministic, secure, and much lighter than traditional SDK bundles. Thanks for watching!"
