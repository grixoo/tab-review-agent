# ADR-0009. Public endpoints in v1 with private networking pre-designed

- **Status:** Proposed
- **Date:** 2026-08-17
- **Deciders:** Infrastructure Architect / Technology Architecture Board
- **Phase:** 1 — Architecture design

## Context

Foundry Agent Service supports a **Standard Setup with private networking**: VNet injection into a delegated subnet (`Microsoft.App/environments`, /27 minimum, /24 recommended, RFC1918 only, one dedicated agent subnet per Foundry account), mandatory BYO Storage, Cosmos DB and AI Search, private endpoints on each, and public network access disabled.

Private networking carries real operational cost: deployment must run from **inside** the VNet (self-hosted GitHub runners or a jump host), the Foundry resource must be in the same region as the VNet, private DNS must resolve end to end, Grounding with Bing is limited to a specific region list, and Code Interpreter loses file support in BYO configurations.

The stakeholder decision was **public endpoints for v1, private later**.

## Decision

v1 deploys with public network access. Private networking is Phase 7. To keep it a change rather than a rebuild:

1. **BYO Storage, Cosmos DB, AI Search and Key Vault from day one** — these are mandatory for the private setup and cannot be retrofitted without redeploying the Foundry account.
2. **VNet and subnet parameters are plumbed through the Terraform module from the start**, defaulted off.
3. **Region is chosen against the private-networking and Bing-grounding region constraints now**, not later.
4. RBAC and managed identity are configured as they would be in the private setup — no keys, no public-access shortcuts.

## Rationale

- Enables a fast pilot while every architectural precondition for hardening is satisfied up front.
- The genuinely irreversible decisions in the private setup are **BYO resources and region**, not the network toggle. Getting those right now is what makes Phase 7 cheap.
- Avoids blocking the pilot on VNet design, self-hosted runners and private DNS before there is evidence the agent is useful.

## Consequences

**Positive:** fast pilot; no self-hosted runner or jump-host requirement yet; standard GitHub-hosted runners work.

**Negative:**
- Data-plane endpoints are internet-reachable during the pilot. Access is controlled by Entra + RBAC only.
- **This is only acceptable because this is a lab exercise using synthetic/fictional submissions only** — there is no real data classification at stake. Revisit before this design is ever pointed at real content.
- Some org network policies prohibit public AI service endpoints outright, which would force Phase 7 forward.

## Phase 7 checklist

Delegated agent subnet · private endpoints on Foundry, AI Search, Storage, Cosmos DB · private DNS zones linked and conditional forwarders to 168.63.129.16 · public network access disabled everywhere · self-hosted GitHub runners inside the VNet · Azure Firewall FQDN allowlist with no TLS inspection · verify Foundry and VNet co-located · confirm the chosen region still supports Grounding with Bing.
