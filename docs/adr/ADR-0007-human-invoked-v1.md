# ADR-0007. Human-invoked reviews in v1; defer automatic submission detection

- **Status:** Proposed
- **Date:** 2026-08-17
- **Deciders:** Infrastructure Architect / Technology Architecture Board
- **Phase:** 1 — Architecture design

## Context

The brief asks the system to identify TAB submissions and detect updated ones. Foundry Routines (preview) provide timer, recurring and event triggers — but the only event trigger is `github_issue`, and connector triggers are not supported in the managed MCP path. A routine fires an agent; it does not tell the agent what changed.

## Decision

In v1, an infrastructure architect **explicitly invokes** the agent against a named Confluence page. Automatic detection and change tracking are deferred to Phase 6.

## Rationale

- "Which pages are new or changed since last time" is a **stateful polling problem**. Handing it to an LLM is expensive, non-deterministic and hard to audit. It needs a cursor, a state store and idempotency — none of which belong in a prompt.
- Deferring it removes an entire class of failure from v1 and lets the pilot focus on the genuinely hard question: is the review any good?
- The agent is **advisory by mandate**, so a human is in the loop regardless. Human invocation is not a workaround; it is consistent with the governance model.
- Routines are preview. Keeping the v1 critical path on GA is a stated principle.

## Consequences

**Positive:** no state store, no cursor, no duplicate-review logic in v1; smallest possible surface; all-GA.

**Negative:** no proactive coverage — an unreviewed submission stays unreviewed until someone asks. Architect effort is not reduced until Phase 6.

## Phase 6 options, to be decided later

| Option | Notes |
|---|---|
| Foundry Routine, recurring trigger | Project-native, no separate scheduler; preview; still needs the agent or a tool to determine what changed |
| Azure Function on a timer + state store | Deterministic, fully controllable, GA; introduces custom code and a state store |
| Logic App with the Confluence connector | Connector has no trigger and only four actions; weakest option |

Change detection will key on the Confluence page **version number**, not `lastmodified` alone, to avoid re-reviewing trivial edits.
