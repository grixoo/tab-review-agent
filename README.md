# TAB Infrastructure Architecture Review Agent

Design for an advisory infrastructure architecture review agent on Microsoft Foundry.
This is a personal Azure lab using synthetic submissions only.

## Current status

This repository contains the design and eleven proposed architecture decision records.
Application code, dependency manifests, deployment scripts, CI, sanitisation tests,
and evaluations have not been implemented. There is no application to start yet.

## Start here

- [Codex instructions](AGENTS.md)
- [Architecture design](docs/architecture/infrastructure-architecture-review-agent-v1.md)
- [Architecture decisions](docs/adr/README.md)
- [Implementation runbook](docs/implementation-runbook.md)
- [Repository review and remaining work](docs/repository-review.md)
- [ADR review checklist](docs/adr/review-checklist.md)

The planned Python workflow uses `uv` and `ruff`. Follow the runbook when asked to
scaffold the application; setup alone does not implement it. Azure infrastructure
belongs to the separate `azure-terraform-lab` repository.

For documentation edits, validate local Markdown links and run `git diff --check`.
No application tests or build commands are available at this stage.
