# ADR-0005. Encode source authority as separate knowledge sources, using GA features only

- **Status:** Proposed
- **Date:** 2026-08-17
- **Deciders:** Infrastructure Architect / Technology Architecture Board
- **Phase:** 1 — Architecture design

## Context

The agent must distinguish **authoritative architecture standards** from general documentation, and must not treat a stale wiki page as enterprise policy. Foundry IQ offers `retrievalInstructions` and `alwaysQuerySource` to steer source selection — but both are **preview-only**; the stable `2026-04-01` Search API drops them, along with answer synthesis and retrieval reasoning effort.

Of the eleven knowledge source kinds, only `searchIndex`, `azureBlob`, `indexedOneLake` and `web` are GA. `mcpServer`, `indexedSharePoint`, `remoteSharePoint`, `workIQ`, `azureSql`, `file` and the two Fabric kinds are preview.

## Decision

Model authority as **structure, not instruction**. One knowledge base composed of four separate GA `azureBlob` knowledge sources, each with a self-describing name and description:

| Knowledge source | Authority tier | Content |
|---|---|---|
| `ks-standards-authoritative` | Tier 1 — binding | Ratified enterprise architecture standards and mandated patterns |
| `ks-reference-architectures` | Tier 2 — approved | Approved reference architectures and blueprints |
| `ks-tab-decisions` | Tier 3 — precedent | Previously reviewed and approved TAB submissions and decisions |
| `ks-platform-docs-general` | Tier 4 — informational | General internal platform and service documentation |

Documents in Blob Storage carry metadata: `authority_tier`, `owner`, `effective_date`, `review_date`, `status`, `source_system`, `source_url`.

Because every retrieved reference names its knowledge source, the agent can state the authority tier of each citation without any preview feature.

## Rationale

- Keeps the entire retrieval path on GA APIs, so the knowledge layer does not inherit preview risk.
- Structural separation is **more robust than instructional steering** — it cannot be overridden by prompt injection in a retrieved document, and it survives model changes.
- Blob knowledge sources auto-generate the full indexer pipeline (data source, skillset, indexer, chunked index) with scheduled incremental refresh, so there is no custom ingestion code.
- Curation becomes an explicit, ownable governance activity rather than an emergent property of what happens to be in the wiki.

## Consequences

**Positive:** all-GA; authority is explicit and auditable; injection-resistant; the agent can say "this deviates from a Tier 1 standard" with a citation.

**Negative:**
- Requires deliberate corpus curation — someone must own tier assignment and the review cadence. This is real, recurring work and is the single largest non-technical dependency in the design.
- Content is a **point-in-time copy**, not live. Staleness is managed by indexer schedule plus `effective_date` / `review_date` metadata surfaced in citations.
- Four sources rather than one adds modest retrieval cost.

## Alternatives considered

| Alternative | Why rejected |
|---|---|
| Single knowledge source + preview `retrievalInstructions` | Puts a preview feature on the critical path and makes authority a prompt-level concern, which injection can attack. |
| `mcpServer` knowledge source pointing at Atlassian MCP | Preview; live Confluence is already covered by the MCP *tool*; conflates "search the corpus" with "read this page". |
| `indexedSharePoint` / `remoteSharePoint` | Preview. Revisit in Phase 8 once GA, if standards live in SharePoint. |
| Flat corpus with authority in document text | Unverifiable and injection-prone. |
