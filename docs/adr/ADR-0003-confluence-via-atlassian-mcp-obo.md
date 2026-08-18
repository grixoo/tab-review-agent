# ADR-0003. Read Confluence via the Atlassian Rovo MCP Server using OAuth on-behalf-of

- **Status:** Proposed
- **Date:** 2026-08-17
- **Deciders:** Infrastructure Architect / Technology Architecture Board
- **Phase:** 1 — Architecture design

## Context

Submissions live in Confluence Cloud. Four integration options were assessed.

| Option | Read page | CQL search | Attachments | Auth | Status |
|---|---|---|---|---|---|
| Atlassian Rovo Remote MCP Server | Yes | Yes | Not documented | OAuth 2.1 / API token | GA, Cloud only |
| Microsoft Confluence connector | Yes | No | No | OAuth | GA connector; Foundry managed-MCP path preview |
| Confluence REST API v2 + CQL, custom wrapper | Yes | Yes | Yes | API token / 3LO | GA |
| Third-party Azure AI Search Confluence connector | Yes | Yes | Yes | Vendor | Commercial product |

There is no first-party Microsoft Confluence knowledge source or Azure AI Search indexer.

## Decision

v1 reads Confluence through the **Atlassian Rovo Remote MCP Server** (`https://mcp.atlassian.com/v1/mcp`), configured as a Foundry project connection using **OAuth identity passthrough (on-behalf-of)**. Only the `read_confluence` and `search_confluence` permission groups are granted; `write_confluence` is withheld at the Atlassian organisation level.

Bulk ingestion of the historical Confluence corpus via REST API v2 + CQL into Blob Storage is deferred to Phase 3 and treated as a separate concern.

## Rationale

- It is the **vendor's own GA server**, which the Foundry documentation explicitly prefers over proxies.
- `searchConfluenceUsingCql` is the only supported option that provides real search — essential for finding related pages, prior submissions and existing capability. The Microsoft connector has no search at all and throttles at 100 calls per 60 seconds.
- OBO means the agent reads Confluence **as the invoking architect**. Space and page restrictions are enforced by Atlassian rather than re-implemented by us. This is the cleanest available answer to "permissions inherited from source systems".
- It requires no custom code and no infrastructure, satisfying the Foundry-native → Azure-native → custom preference order.
- Withholding `write_confluence` makes the read-only mandate a platform-enforced property rather than a prompt instruction.

## Consequences

**Positive:** no custom ingestion code in v1; live rather than stale content; permission enforcement inherited; org-level admin controls, IP allowlisting and audit logging available on the Atlassian side.

**Negative:**
- **Reviews are not reproducible across users** with different Confluence access. This must be stated in the review footer.
- Atlassian is a non-Microsoft service; prompt content transits outside the Azure boundary. Foundry documentation is explicit that Microsoft does not verify third-party MCP servers.
- Attachment access is unproven — no dedicated attachment tool appears in the supported-tools list. If attachments prove load-bearing, Phase 3 ingestion is pulled forward.
- Cloud-only. A Data Center estate would invalidate this decision entirely.
- Community reports of 401 auth-model mismatches between Foundry and the Atlassian MCP server mean OBO must be proven in the first spike.

## Verification required before Phase 2 exit

1. OAuth OBO works end to end from a Foundry project connection.
2. Attachment retrieval is either possible or confirmed out of scope.
3. Read/search can be granted and write withheld on a personal/trial Confluence Cloud site — self-administered, since there is no real org admin in this lab.
