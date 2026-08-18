# ADR-0011. Deploy `gpt-5.5` in UK South, with `gpt-4.1` as fallback

- **Status:** Proposed
- **Date:** 2026-08-17
- **Deciders:** Infrastructure Architect / Technology Architecture Board
- **Phase:** 1 — Architecture design (added after region confirmation)

## Context

UK South was confirmed as the target region. Verification against the Foundry Agent Service regional support and tool-by-region matrices shows UK South supports everything this design needs:

| Capability | UK South |
|---|---|
| Responses API | Yes |
| Agents | Yes |
| MCP | Yes |
| Web Search | Yes |
| Grounding with Bing Search | Yes |
| Grounding with Bing Custom Search | Yes |
| Azure AI Search | Yes |
| File Search | Yes |
| Code Interpreter | Yes |
| Private Class A IP ranges (`10.0.0.0/8`) | Yes — matters for Phase 7 |
| Computer Use | No — not required |
| Managed MCP via connector namespaces | **No** — not required, we call Atlassian MCP directly (ADR-0003) |

However, **tool support is gated by model as well as by region**, and the two constraints compose. Several current models silently lack tools this design depends on:

| Model | Azure AI Search | MCP | Web Search | Grounding Bing | Bing Custom | Verdict |
|---|---|---|---|---|---|---|
| `gpt-5-mini` | **No** | Yes | Yes | **No** | **No** | Rejected — no retrieval tool, no Bing fallback |
| `gpt-5` | Yes | Yes | Yes | Yes | Yes | Viable, but requires registration |
| `gpt-5.1` | Yes | Yes | Yes | Yes | **No** | Rejected — loses the ADR-0006 fallback |
| `gpt-5.2` | Yes | Yes | Yes | Yes | **No** | Rejected — loses the ADR-0006 fallback |
| `gpt-5.4` | Yes | Yes | Yes | Yes | Yes | Viable |
| **`gpt-5.5`** | Yes | Yes | Yes | Yes | Yes | **Selected** |
| `gpt-4.1` | Yes | Yes | Yes | Yes | Yes | **Fallback** |

## Decision

Deploy **`gpt-5.5`** as the agent model in UK South. Keep **`gpt-4.1`** as the documented fallback if `gpt-5` family registration, eligibility or quota blocks deployment.

Treat the model deployment name as a **Terraform variable**, and the model reference in the agent definition as a **data-plane variable**, so the model can be changed without touching either layer's structure.

## Rationale

- `gpt-5.5` is the only recent model that supports the **complete** tool set this design needs, including Grounding with Bing Custom Search.
- Bing Custom Search is not used in v1 (ADR-0006 selected unrestricted web search), but it is the documented one-config-change fallback if TAB later rejects the data-boundary risk. **Choosing `gpt-5.1` or `gpt-5.2` would silently delete that fallback** — the mitigation would still be written in the ADR but would no longer be executable without also changing the model, which changes review behaviour at the same time. Preserving a mitigation you might need is worth more than a marginal model preference.
- `gpt-4.1` supports the identical tool set and is broadly available without registration gating, making it a genuine like-for-like fallback rather than a degraded one.
- Reasoning quality matters here: the agent must hold a whole design in context and assess it across sixteen coupled domains (ADR-0001). This is not a task for a mini or nano tier.

## Consequences

**Positive:** full tool set available; the ADR-0006 fallback stays executable; the fallback model requires no architecture change.

**Negative:**
- `gpt-5` family models require registration and access is granted on Microsoft's eligibility criteria — this is a **lead-time dependency on the first implementation step** and must be confirmed before writing the model deployment resource.
- Frontier-tier inference on long documents with multi-tool loops is the dominant running cost. Track tokens per review from Phase 2 onward via Application Insights.
- Model availability changes frequently. Re-verify the model × tool matrix before each phase rather than trusting this table.

## Notes for implementation

- Rate limits are enforced at the **model deployment** level, not the agent level. Implement exponential backoff with jitter for 429s.
- Agents cap at **1,000 valid revisions**; the version-cap error is terminal, not transient. Rotate or delete old agent versions in the deploy pipeline (ADR-0008).
- Structured outputs remain unsupported in Agent Service regardless of model (ADR-0004) — model choice does not change this.
