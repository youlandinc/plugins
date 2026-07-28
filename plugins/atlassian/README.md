# Atlassian

The official Atlassian Rovo MCP server plus workflow skills, vendored for the
Corepass plugin marketplace. Gives AI assistants secure access to Jira,
Confluence, and Compass — search, create, and manage your work.

## What's included

- **MCP server** (`.mcp.json`) — the cloud-hosted Atlassian Rovo MCP server at
  `https://mcp.atlassian.com/v1/mcp/authv2` (OAuth 2.1). No local install.
- **Skills** (`skills/`) — ready-made workflows:
  - `capture-tasks-from-meeting-notes` — turn meeting notes into Jira issues
  - `generate-status-report` — build status reports from JQL queries
  - `jira-sprint-dashboard-canvas` — render a sprint dashboard on a Confluence canvas
  - `search-company-knowledge` — search across Jira and Confluence
  - `spec-to-backlog` — break a spec into epics and tickets
  - `triage-issue` — triage and label incoming issues

## Installation

Install from the Corepass marketplace:

```bash
harness marketplace add youlandinc/plugins
harness plugin install atlassian
```

On first use the MCP server will prompt you to authorize with your Atlassian
account via OAuth.

## License & provenance

- **Upstream**: <https://github.com/atlassian/atlassian-mcp-server>
- **License**: Apache-2.0 (see `LICENSE`)

All content remains the copyright of Atlassian. See `LICENSE` for terms.

## Support

- **Atlassian Rovo MCP docs**: <https://support.atlassian.com/atlassian-rovo-mcp-server/docs/getting-started-with-the-atlassian-remote-mcp-server/>
- **Report issues**: <https://github.com/atlassian/atlassian-mcp-server/issues>
