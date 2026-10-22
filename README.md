# m365-github-agent

A minimal Microsoft 365 Copilot declarative agent that connects to GitHub's hosted MCP server:

`https://api.githubcopilot.com/mcp/`

The goal is to provide a small, inspectable starting point for using live GitHub tools from Microsoft 365 Copilot instead of only indexing GitHub content with a Copilot Connector.

## Architecture

```text
Microsoft 365 Copilot
        |
        v
Declarative Agent
        |
        v
MCP plugin (RemoteMCPServer)
        |
        v
GitHub hosted MCP server
https://api.githubcopilot.com/mcp/
        |
        v
GitHub
```

This starter follows the current Microsoft 365 Agents Toolkit MCP pattern: the plugin uses dynamic tool discovery with `functions: []`, `RemoteMCPServer`, and `run_for_functions: ["*"]`.

## Prerequisites

- A Microsoft 365 tenant where Copilot extensibility / declarative-agent sideloading is allowed.
- Microsoft 365 Copilot.
- Visual Studio Code.
- Microsoft 365 Agents Toolkit 6.12.0 or later.
- A GitHub account.
- A GitHub OAuth App for the MCP connection.

Official walkthrough: <https://learn.microsoft.com/microsoft-365/copilot/extensibility/build-mcp-plugins>

## 1. Create a GitHub OAuth App

Open <https://github.com/settings/developers>, choose **OAuth Apps -> New OAuth App**, and use:

- **Application name:** for example `M365 GitHub Agent`
- **Homepage URL:** this repository URL or another URL you control
- **Authorization callback URL:** `https://teams.microsoft.com/api/platform/v1.0/oAuthRedirect`

After creating the OAuth App, copy its **Client ID** and generate a **Client secret**.

## 2. Configure local environment values

Edit `env/.env.dev`:

```dotenv
MCP_DA_OAUTH_CLIENT_ID_APIGITHUBC=<your GitHub OAuth client ID>
```

Put the client secret in the ignored user environment file `env/.env.dev.user`:

```dotenv
SECRET_MCP_DA_OAUTH_CLIENT_SECRET_APIGITHUBC=<your GitHub OAuth client secret>
```

Do **not** commit the client secret.

`MCP_DA_AUTH_ID_APIGITHUBC` is populated during provisioning by the `oauth/register` lifecycle action.

## 3. Provision

Open this repository in VS Code with Microsoft 365 Agents Toolkit installed, sign in to Microsoft 365, then run **Provision** from the Toolkit Lifecycle pane.

Provisioning:

1. Creates the Microsoft 365 app registration/package.
2. Registers the GitHub OAuth connection in the plugin vault.
3. Packages and validates the declarative agent.
4. Publishes the development agent to Microsoft 365 Copilot.

## 4. Test

Open <https://m365.cloud.microsoft/chat>, select the development agent, and try prompts such as:

- `List the GitHub repositories I own.`
- `Find repositories related to Microsoft Kiota.`
- `Show my GitHub profile.`
- `Find open issues in owner/repo related to authentication.`
- `Review the recent pull requests in owner/repo.`

The first GitHub operation should prompt you to sign in and authorize the GitHub OAuth App.

## Project layout

```text
appPackage/
  ai-plugin.json          # Remote GitHub MCP plugin
  declarativeAgent.json   # Declarative agent definition
  instruction.txt         # Agent behavior
  manifest.json           # Microsoft 365/Teams app manifest
  color.png
  outline.png
env/
  .env.dev                # Non-secret development values
.vscode/
  mcp.json                # MCP endpoint for development tooling
  extensions.json
  launch.json
  settings.json
m365agents.yml            # Provision/publish lifecycle
```

## Notes

- The GitHub MCP server is hosted by GitHub; this repository does not run an MCP server.
- Tool availability is discovered dynamically at runtime, so GitHub can add or update tools without requiring a pinned tool list in this repository.
- The permissions available to the agent are ultimately constrained by the GitHub account/OAuth authorization used by the signed-in user.
- For production use, review the discovered tool set, tenant governance policy, OAuth permissions, and whether write-capable GitHub tools should be exposed.

## References

- Microsoft: [Build a plugin for a declarative agent from an MCP server](https://learn.microsoft.com/microsoft-365/copilot/extensibility/build-mcp-plugins)
- Microsoft 365 Agents Toolkit: <https://github.com/OfficeDev/microsoft-365-agents-toolkit>
- GitHub MCP Server: <https://github.com/github/github-mcp-server>
