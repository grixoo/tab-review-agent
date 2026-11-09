# Repository review and next steps

## Completed

- Adopted root `AGENTS.md`, README, documented layout, Codex runbook and ADR checklist.
- Lab owner confirmed two repositories on 2026-11-09: Terraform and Foundry
  infrastructure in `azure-terraform-lab`; agent details in `tab-review-agent`.
  ADR-0008 now records this separation and the existing in-house module approach.
- Recorded acceptance of the eleven ADRs, retaining their original dates.
- Inspected the Terraform source and documented all nine Key Vault contract entries
  in [the infrastructure contract](infrastructure-contract.md).
- Corrected ADR-0011's claim that only one recent model has full tool coverage;
  retained `gpt-5.5` as selected and `gpt-4.1` as fallback.

## Remaining work

1. Verify the actual infrastructure deployment. Source inspection establishes that
   the workload exists in Git, not that it has been applied successfully.
2. Establish the deploy identity, OIDC trust, Key Vault URI bootstrap input and
   permissions. Confirm separation between project contributor roles and the
   review agent's reader roles before runtime integration.
3. Revalidate current UK South model/tool support and subscription quota before
   deployment. The Terraform configuration has been updated to select `gpt-5.5`;
   `gpt-4.1` remains the explicit fallback, not an automatic substitution.
4. Scaffold the Python project, pinned dependencies, lint, tests and CI (runbook
   Step 3), then prove the minimal integration before adding retrieval/evaluation.

The architecture diagram remains optional documentation work. No application
build or test suite exists in this repository yet.
