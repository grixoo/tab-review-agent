# ADR-0006. Use the Foundry web search tool for external research, with query sanitisation

- **Status:** Proposed
- **Date:** 2026-08-17
- **Deciders:** Infrastructure Architect / Technology Architecture Board
- **Phase:** 1 — Architecture design

## Context

The agent must research current vendor documentation, Microsoft/Azure guidance and industry best practice. Options: the built-in web search tool (Grounding with Bing), Bing Custom Search restricted to an approved domain allowlist, or the Microsoft Learn MCP server alone.

The stakeholder decision was **unrestricted public web**.

Microsoft documentation is explicit: the **Data Protection Addendum does not apply** to data sent to Grounding with Bing Search, and data transfers occur outside Azure compliance and geographic boundaries.

## Decision

Use the built-in **web search tool** (Grounding with Bing) for general external research, **plus** the **Microsoft Learn MCP server** as a preferred-first source for anything Microsoft or Azure.

Mandatory compensating controls:

1. Agent instructions require web queries to be formulated from **generic technology terms only** — never verbatim submission text, project names, hostnames, IP ranges, account names, or internal identifiers.
2. A PII control at the **tool call** intervention point (preview) scans outbound tool arguments.
3. Microsoft Learn MCP is consulted **before** general web search for Azure and Microsoft topics.
4. All web-derived content is tagged `[EXTERNAL]` with a URL citation and is never treated as an enterprise standard.
5. Traces record every outbound query for audit.

## Rationale

- Web search is GA, requires no separate Bing resource or project connection, and returns inline `url_citation` annotations, which directly satisfies the traceability requirement.
- Microsoft Learn MCP returns clean markdown from official documentation and is materially higher signal than general web results for Azure topics, which will dominate this workload.
- Unrestricted search gives the vendor-documentation breadth the review process needs. The residual risk is a data-boundary risk, which is addressed by controlling *what* is sent rather than *where* it goes.

## Consequences

**Positive:** widest source coverage; GA; citations by construction; no extra Bing resource to manage.

**Negative:**
- **Submission-derived content leaves the Azure compliance and geographic boundary.** There is no real TAB to record acceptance in this lab; risk-accepted by the lab owner on condition that only synthetic/fictional submissions are ever used. This is the most significant residual risk in the v1 design, and worth mitigating properly anyway since it's the practice that matters.
- Public web content is untrusted input and a prompt-injection vector — mitigated by the indirect-attack guardrail at the tool-response intervention point (ADR-0007) and by the agent's total lack of write capability (ADR-0010).
- Source quality is variable; the agent must weight Microsoft Learn and vendor documentation above blogs and forums, which is prompt discipline rather than an enforced control.
- Grounding with Bing is restricted to a specific region list, which constrains the region choice.
- Agent evaluators for tool-call accuracy and groundedness have **limited support** for conversations containing web search calls, so evaluation must rely on Task Adherence and custom rubrics.

## Fallback

Switching to **Bing Custom Search with a domain allowlist** (learn.microsoft.com plus approved vendor domains) is a single tool-configuration change inside the toolbox, requiring no agent change. Retain this as the documented response if the data-boundary risk is later rejected.
