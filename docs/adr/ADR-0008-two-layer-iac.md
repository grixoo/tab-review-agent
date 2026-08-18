# ADR-0008. Two-layer IaC: Terraform for the control plane, azd/SDK for the data plane

- **Status:** Proposed
- **Date:** 2026-08-17
- **Deciders:** Infrastructure Architect / Technology Architecture Board
- **Phase:** 1 — Architecture design

## Context

The target is Terraform plus GitHub Actions with OIDC, using Azure Verified Modules where appropriate. Foundry infrastructure — account, project, model deployment, AI Search, Storage, Cosmos DB, Key Vault, Application Insights, RBAC — is ARM-manageable and covered by the AVM pattern module `Azure/terraform-azurerm-avm-ptn-aiml-ai-foundry`.

Agent definitions, toolboxes, knowledge bases and knowledge sources are **data-plane objects**. There is no Terraform provider coverage for them; they are created via SDK, REST or `azd ai`.

## Decision

Two layers, two pipelines, one repository.

**Layer 1 — control plane (Terraform):** Foundry account and project, model deployment, Azure AI Search (with the `knowledgeRetrieval` billing consent property set), Storage, Cosmos DB, Key Vault, Application Insights, Log Analytics, RBAC, diagnostic settings, and — from Phase 7 — VNet, subnets, private endpoints and private DNS. Uses the AVM pattern module in **BYOR** mode, which its documentation states is the only supported path for production AI landing zones.

**Layer 2 — data plane (declarative definitions + `azd`/SDK):** agent versions, toolbox versions, knowledge base and knowledge source definitions, and project connections, held as version-controlled YAML/JSON in the same repository and applied by a deploy step.

Both layers run from GitHub Actions with OIDC federated credentials. No service principal secrets.

## Rationale

- Terraform is used where it is genuinely authoritative, and not stretched into a surface it cannot express. Attempting a Terraform-only model would mean `null_resource`/`local-exec` shims — untestable, non-idempotent and unreviewable.
- Splitting the layers matches their **different change cadences**. Infrastructure changes rarely; agent instructions will change constantly during tuning.
- AVM gives WAF-aligned defaults, private endpoint support and diagnostics without hand-rolling modules.
- Keeping definitions declarative and in-repo preserves version control, code review and rollback for the data plane, even without a provider.

## Consequences

**Positive:** each layer uses the right tool; independent change cadence; no brittle provisioner shims; agent instructions get PR review like any other artefact.

**Negative:**
- Two pipelines and an ordering dependency (control plane must exist before data plane).
- Data-plane state is not tracked by Terraform, so drift detection must be built (a reconcile step comparing declared versus deployed definitions).
- The `azd` `microsoft.foundry` extension is preview; the `azure-ai-projects` SDK is the GA fallback for the deploy step.
- Publishing an agent creates a **new distinct agent identity**, so RBAC assignments must be re-applied. This must live in the pipeline, not a runbook.

## Revisit when

Terraform provider resources for Foundry agents, toolboxes or knowledge bases become available.
