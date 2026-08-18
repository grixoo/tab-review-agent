# ADR-0002. Attach all capabilities through a single Foundry Toolbox

- **Status:** Proposed
- **Date:** 2026-08-17
- **Deciders:** Infrastructure Architect / Technology Architecture Board
- **Phase:** 1 — Architecture design

## Context

Tools can be attached directly to an agent definition, or curated into a **Toolbox** — a versioned bundle exposed as one managed MCP endpoint. Microsoft documents the toolbox as the recommended approach. Toolboxes are GA.

The brief requires that agent reasoning, tools/integrations, enterprise knowledge, and governance be separable layers.

## Decision

All tools — Atlassian MCP, the Foundry IQ knowledge base, web search, and the Microsoft Learn MCP server — are curated into **one versioned Foundry Toolbox**. The agent attaches that toolbox as a single MCP tool.

## Rationale

- The toolbox is the **architectural seam** that makes the layer separation real rather than notional. Changing the tool surface becomes a toolbox version bump, not an agent change.
- Credentials, connections and policy are centralised in one place instead of being scattered across agent definitions.
- Toolbox versioning gives explicit control over when a tool change takes effect: create a version, test it, promote it to default.
- The same toolbox is reusable by a future hosted agent, so ADR-0001's escalation path costs less.

## Consequences

**Positive:** clean separation of concerns; controlled rollout of tool changes; reuse across agents and runtimes; one credential management surface.

**Negative:** one extra indirection to reason about when debugging; toolbox versions become an additional artefact to manage in the deployment pipeline.

## Alternatives considered

| Alternative | Why rejected |
|---|---|
| Attach tools directly to the agent | Couples tool changes to agent versions, defeats the layer separation the brief requires, and scatters credentials. |
| One toolbox per tool category | Unnecessary fragmentation at v1 scale. Revisit if tool count grows substantially. |
