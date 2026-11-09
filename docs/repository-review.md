# Repository review and tidy-up plan

## Completed

- Adopt root `AGENTS.md` so Codex discovers repository instructions.
- Add a README describing the actual documentation-only status and reading order.
- Move the architecture document into `docs/architecture/` as existing references expect.
- Move the implementation runbook into `docs/` and replace Claude-specific setup,
  hooks, keyboard shortcuts, and slash commands with Codex-compatible guidance.
- Replace the hidden Claude command with a portable ADR review checklist.
- Preserve the numbered ADRs and architecture decisions.

## Resolve before implementation

1. ADR-0008 says “one repository” and co-located definitions, while the runbook and
   agent instructions specify separate control-plane and data-plane repositories.
   Reconcile this explicitly through the ADR process before implementing the seam.
2. All eleven ADRs are marked Proposed, while agent instructions call the decisions
   settled. Confirm their intended acceptance status; no status was changed by cleanup.
3. The runbook references a Key Vault outputs contract, but exact secret names are
   absent here. Inspect the Terraform repository and document that contract before
   implementing `deploy/config.py`.
4. ADR-0011 lists `gpt-5.4` with full tool support but calls `gpt-5.5` the only recent
   model supporting the full set. Recheck official availability and clarify the rationale.
5. Terraform progress reported by the runbook is unverified in this checkout.
   Inspect that repository before treating those steps as done.

## Next coding task

Scaffold the planned Python package, dependency manifest and lockfile, development
checks, and CI as described in runbook Step 3. Then implement and test the minimal
integration before expanding retrieval and evaluation. Verify current Foundry APIs
and required authentication first. The architecture diagram referenced in the old
runbook has not been supplied and remains optional documentation work.
