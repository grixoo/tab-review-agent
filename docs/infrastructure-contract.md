# Infrastructure repository inspection and outputs contract

Inspected `grixoo/azure-terraform-lab` at commit
`c588dded943e18202881ae32a950054e67cf3470` on 2026-11-09.
This is source inspection only: no Terraform plan, apply, workflow-run status,
or live Azure resource checks were performed.

## Implemented source

`experiments/tab-review-agent/` contains the control-plane composition: Foundry
account, project, model, Storage, Cosmos DB, AI Search, managed identity, Key Vault,
observability, three keyless project connections and both capability hosts.
The project capability host explicitly depends on the account host and connections.
The workflow `.github/workflows/terraform-tab-review-agent.yml` dispatches the
shared Terraform workflow using OIDC. Source exists; deployment success is unverified.

## Key Vault contract

The producer is `experiments/tab-review-agent/keyvault.tf`. The consumer will be
`deploy/config.py` in this repository, using `DefaultAzureCredential`.

| Secret name | Meaning |
| --- | --- |
| `foundry-project-endpoint` | Full Foundry project API endpoint |
| `foundry-account-name` | Foundry account name |
| `foundry-project-name` | Foundry project name |
| `model-deployment-name` | Actual primary model deployment name |
| `search-endpoint` | AI Search endpoint |
| `search-service-name` | AI Search service name |
| `knowledge-storage-account` | Knowledge storage account name |
| `appinsights-connection-string` | Application Insights connection string |
| `workload-resource-group` | Workload resource group name |

Read these entries from the vault; do not copy their values into this repository.
The vault URI itself is a bootstrap input, exposed as Terraform output
`key_vault_uri`. Choose and document its delivery to the agent deploy workflow
before implementing the consumer. Fail clearly on missing contract entries;
no fallback environment variables for the endpoint values.

## Remaining integration checks

- At the inspected commit, `terraform.tfvars` selected `gpt-4.1`. The lab owner
  subsequently requested `gpt-5.5` as primary throughout, and the local Terraform
  configuration has been updated accordingly. Azure deployment is not yet verified;
  model access and quota must be confirmed first. Consume the actual deployment
  name from the contract rather than hard-coding either model name.
- `agent_repo_deploy_principal_id` is null. The conditional Key Vault Secrets User
  grant is therefore absent until the agent deploy identity is supplied. Verify
  OIDC trust and data-plane deployment permissions separately; vault read access
  alone does not authorize agent deployment.
- The Terraform runner needs Key Vault write access; the workload does not grant
  that bootstrap permission itself. Verify it before deployment.
- `identity.tf` grants contributor roles to the Foundry project identity for its
  managed backing services. ADR-0010 requires reader roles for the review agent.
  Verify the runtime/published identity and permission boundary before invoking
  the agent; do not assume the project identity satisfies the read-only rule.
- The project endpoint is constructed from the account endpoint and project name.
  Verify it against the deployed project's actual API endpoint.
