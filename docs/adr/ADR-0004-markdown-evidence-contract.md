# ADR-0004. Use a markdown output contract with evidence tags rather than JSON schema

- **Status:** Proposed
- **Date:** 2026-08-17
- **Deciders:** Infrastructure Architect / Technology Architecture Board
- **Phase:** 1 — Architecture design

## Context

The review must reliably distinguish facts found in the submission, information from internal sources, externally researched information, architectural inference, and assumptions. The obvious mechanism is a JSON schema with typed fields.

Microsoft documentation states that structured outputs (`text.format` with `json_schema`, and `strict: true` function calling) are **not supported with Foundry Agents Service**, although they are GA for the underlying models on the `v1` API.

## Decision

The review output is **markdown with a fixed twelve-section structure and a mandatory inline evidence tag on every substantive statement**:

`[SUBMITTED]` · `[INTERNAL]` · `[EXTERNAL]` · `[INFERRED]` · `[ASSUMPTION]` · `[GAP]`

The contract is enforced by the agent instructions and verified by the evaluation harness, not by the platform.

**Hard rule:** an enterprise standard may only be asserted as `[INTERNAL]` with a citation to a retrieved document. If retrieval returns nothing, the agent must emit `[GAP]` and a question for the submitting team. It must never present `[INFERRED]` content as an enterprise standard.

## Rationale

- Schema enforcement is not available on the chosen platform surface, so the choice is between prompt-enforced structure and abandoning the prompt agent (ADR-0001).
- The primary consumers are human architects and TAB, who read prose. Markdown is directly usable; JSON would need rendering anyway.
- Evidence tags give most of the traceability benefit of typed fields at a fraction of the complexity, and they survive copy-paste into Confluence.
- The hard rule above is the single most important control against the dominant failure mode of a review agent: fabricating enterprise policy.

## Consequences

**Positive:** works within GA platform constraints; human-readable; no post-processing; the tagging discipline is directly measurable by an evaluator.

**Negative:** no machine guarantee of structure; a run can omit a section or drop a tag. Downstream systems cannot parse the review reliably.

**Mitigation:** a custom rubric evaluator that scores section completeness and evidence-tag discipline, run in CI against a golden set. Two consecutive failures of the structure check is one of the ADR-0001 escalation triggers.

## Revisit when

Structured output support lands in Agent Service, or a consumer needs machine-readable output — at which point the hosted-agent path (ADR-0001 escalation) allows a direct Responses API call with `text.format`.
