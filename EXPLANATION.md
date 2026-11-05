# Swytchcode in My Own Words

Swytchcode is a contract-based API execution tool designed for developers building AI agents and automated backend workflows. 

### What Problem It Solves
When building AI agents that call third-party services like Stripe or HubSpot, developers usually have to install multiple heavy SDKs and expose raw API keys directly to the application environment. More importantly, LLMs frequently hallucinate parameter names or send malformed payloads, leading to failed requests or silent data corruption. Traditional API documentation and massive OpenAPI specs are hard for models to navigate reliably.

### How a Developer Uses It
Instead of client libraries, Swytchcode introduces local contracts called **Wrekenfiles** (`wrekenfile.yaml`). A developer initializes their project with `swytchcode init`, selects the services they need, and executes calls through a single lightweight runtime (`swytchcode-runtime` in Python or the `swy` CLI). Swytchcode acts as a local gateway: it validates payloads against the contract schema before anything touches the network, while keeping API keys isolated.

### Practical Example
In a customer onboarding workflow, rather than importing separate Stripe and HubSpot SDKs, the agent makes a structured call:

```python
from swytchcode_runtime import exec as swytchcode_exec

result = swytchcode_exec("stripe.create_payment", {
    "amount": 2000,
    "currency": "usd"
})
```

Swytchcode validates the inputs locally against the Stripe Wrekenfile, runs the request securely, and returns a clean, typed response.
