# ADR-0001. Use a single Foundry prompt agent for v1

- **Status:** Proposed
- **Date:** 2026-08-17
- **Deciders:** Infrastructure Architect / Technology Architecture Board
- **Phase:** 1 — Architecture design

## Context

The Infrastructure Architecture Review Agent must comprehend a submission, search internal knowledge, research externally, assess against roughly 16 infrastructure domains, and produce a structured evidence-based review. Microsoft Foundry offers three viable shapes: a prompt agent (configuration only, fully managed), a hosted agent (your code in a Foundry-run container), and multi-agent orchestration via Workflows or Microsoft Agent Framework.

The brief explicitly warns against assuming multi-agent is preferable and asks for the simplest architecture that satisfies the requirements.

## Decision

v1 is a **single prompt agent** with all capabilities attached through one Foundry Toolbox.

## Rationale

- The 16 review domains are **coupled, not independent**. A networking decision constrains DR; a resiliency decision drives cost. Splitting domains across specialist agents fragments the shared context that makes an architecture review worth reading, and forces expensive context re-sharing between agents.
- Multi-agent architectures solve *concurrency across distinct tasks*. This is one task with many facets.
- A prompt agent has **no compute to manage, no container image, no registry, no build pipeline**. Tracing, versioning, identity, publishing and RBAC are all provided.
- Every capability required in v1 is reachable from a prompt agent: MCP tools, Foundry IQ knowledge base, web search, Microsoft Learn.
- Foundry supports Workflows and Agent Framework orchestration when needed, so this decision is reversible without re-platforming.

## Consequences

**Positive:** fastest path to a working review; minimal operational surface; lowest cost; all-GA critical path.

**Negative:** domain coverage is a matter of prompt discipline rather than code-enforced control flow. Consistency across runs is weaker than a staged pipeline would give. Output cannot be schema-validated (see ADR-0004).

**Mitigation:** an explicit review rubric in the instructions; a fixed output contract; and an evaluation harness that measures domain coverage against a golden set.

## Escalation trigger

Move to a hosted agent with staged orchestration when any of the following holds:

1. Evaluation shows review domains are skipped in more than ~15% of runs.
2. Consumers require machine-readable (JSON) review output.
3. Deterministic phase separation with per-phase validation becomes necessary.
4. Confluence write-back with an approval gate is required.

## Alternatives considered

| Alternative | Why rejected |
|---|---|
| Orchestrator + specialist domain agents | Premature. Fragments coupled context, multiplies cost, harder to trace, no evidence yet that a single agent is insufficient. |
| Hosted agent (Agent Framework) from day one | Superior end state but adds container build, ACR, image lifecycle, deployment pipeline and compute cost before any quality evidence exists. |
| Microsoft Copilot Studio | Weaker control over retrieval, tool orchestration and evaluation; the workload is developer-owned, not maker-owned. |
| Foundry Workflows | Solves branching and approvals, which v1 does not have. Workflow tracing is preview. |
