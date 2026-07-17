# Atlassian CLI

Command-line interface for Atlassian Jira and Confluence written in Zig.

## Features

- **Jira Operations**: Search issues, manage projects, track sprints and boards
- **Confluence Operations**: Search pages, manage spaces, view comments and labels
- **Fast**: Built with Zig for minimal overhead
- **Simple**: Straightforward command-line interface
- **Secure**: Uses API tokens for authentication

## Prerequisites

- Zig 0.15.2 or later
- Atlassian account with API token
- Jira and/or Confluence instance

## Installation

```bash
# Clone the repository
git clone https://github.com/ainoya/atlassian-cli.git
cd atlassian-cli

# Build
zig build

# Install (optional)
zig build install
```

The binary will be available at `./zig-out/bin/atlassian-cli`

## Configuration

The CLI reads each value from an environment variable first, then falls back to a config file at `~/.config/atlassian-cli/config.json`.

**Provide secrets through the environment, not the config file.** The API token (and Confluence token) should come from an exported environment variable or a secrets manager. The config file stores values in plaintext on disk, so it is not an appropriate place for credentials — use it, if at all, only for the non-secret `atlassian_url` and `atlassian_username`.

### Environment Variables (recommended)

Environment variables take precedence over the configuration file. Set the following:

```bash
export ATLASSIAN_URL="https://your-domain.atlassian.net"
export ATLASSIAN_USERNAME="your-email@example.com"
export ATLASSIAN_API_TOKEN="your-api-token"
export ATLASSIAN_CLOUD="true"  # true for Cloud, false for Server/DC
export ATLASSIAN_JIRA_API_VERSION="3"  # optional override: 2, 3, or latest

# Confluence-specific settings
export CONFLUENCE_URL="https://your-confluence-domain.example.com"  # optional separate Confluence host
export CONFLUENCE_USERNAME="your-email@example.com"  # optional separate Confluence username
export CONFLUENCE_API_TOKEN="your-confluence-token"  # optional separate Confluence token
export CONFLUENCE_BASE_PATH="/wiki"
```

### Config File (optional, non-secrets only)

The `config` command persists values to `~/.config/atlassian-cli/config.json`. Because that file is plaintext, use it only for non-secret values such as the URL and username — keep the token in the environment.

```bash
atlassian-cli config set atlassian_url https://your-domain.atlassian.net
atlassian-cli config set atlassian_username your-email@example.com

# Read a value back
atlassian-cli config get atlassian_url
```

Only the three core keys are supported: `atlassian_url`, `atlassian_username`, `atlassian_api_token`. An exported environment variable always overrides the stored value.

### Creating an API Token

1. Go to https://id.atlassian.com/manage-profile/security/api-tokens
2. Click "Create API token"
3. Give it a name and copy the token
4. Provide this token as the `ATLASSIAN_API_TOKEN` environment variable (or via your secrets manager). Keep it out of the config file, source control, and shell history where practical. For Jira/Confluence Server/DC, this value is your account password or a personal access token rather than a Cloud API token.

### Using direnv (recommended)

This keeps credentials in your environment rather than a plaintext config file. `.envrc` is already listed in `.gitignore`, so your secrets stay out of version control — never commit it.

Create a `.envrc` file in the project directory:

```bash
export ATLASSIAN_URL="https://your-domain.atlassian.net"
export ATLASSIAN_USERNAME="your-email@example.com"
export ATLASSIAN_API_TOKEN="your-api-token"
export ATLASSIAN_CLOUD="true"
export ATLASSIAN_JIRA_API_VERSION="3"

# Confluence-specific settings
export CONFLUENCE_URL="https://your-confluence-domain.example.com"
export CONFLUENCE_USERNAME="your-email@example.com"
export CONFLUENCE_API_TOKEN="your-confluence-token"
export CONFLUENCE_BASE_PATH="/wiki"
```

If `ATLASSIAN_JIRA_API_VERSION` is not set, the CLI defaults to `3` for Atlassian Cloud and `2` for Jira Server/Data Center. Set it explicitly if you want to force a specific version such as `latest`.
If `CONFLUENCE_URL` is not set, Confluence commands fall back to `ATLASSIAN_URL`.
If `CONFLUENCE_USERNAME` or `CONFLUENCE_API_TOKEN` are not set, Confluence commands fall back to `ATLASSIAN_USERNAME` and `ATLASSIAN_API_TOKEN`.

Then run:
```bash
direnv allow
```

## Usage

### General Syntax

```bash
atlassian-cli <service> <command> [options]
```

### Jira Commands

#### Get Issue
```bash
atlassian-cli jira issue PROJECT-123
```

#### Search Issues
```bash
# Basic JQL search
atlassian-cli jira search "project=DEV AND status=Open"

# With max results
atlassian-cli jira search "assignee=currentUser()" --max=50

# Recent issues
atlassian-cli jira search "created >= -7d ORDER BY created DESC" --max=20
```

#### List Projects
```bash
atlassian-cli jira projects
```

#### Get Project Issues
```bash
atlassian-cli jira project-issues DEV --max=20
```

#### List Agile Boards
```bash
# All boards
atlassian-cli jira boards

# Scrum boards only
atlassian-cli jira boards --type=scrum

# Kanban boards only
atlassian-cli jira boards --type=kanban
```

#### List Sprints
```bash
# All sprints for board
atlassian-cli jira sprints 123

# Active sprints only
atlassian-cli jira sprints 123 --state=active

# Closed sprints
atlassian-cli jira sprints 123 --state=closed
```

#### Get Sprint Issues
```bash
atlassian-cli jira sprint-issues 456 --max=50
```

#### Get Current User
```bash
atlassian-cli jira user
```

### Confluence Commands

#### Get Page
```bash
atlassian-cli confluence page 123456
```

#### Search Pages
```bash
# Simple CQL search
atlassian-cli confluence search "type=page AND space=DEV"

# Search with limit
atlassian-cli confluence search "siteSearch ~ \"documentation\"" --limit=20

# Date range
atlassian-cli confluence search "created >= \"2024-01-01\" AND space=TEAM" --limit=10
```

#### List Spaces
```bash
atlassian-cli confluence spaces --limit=50
```

#### Get Space Details
```bash
atlassian-cli confluence space DEV
```

#### Get Child Pages
```bash
atlassian-cli confluence children 123456 --limit=25
```

#### Get Page Comments
```bash
atlassian-cli confluence comments 123456
```

#### Get Page Labels
```bash
atlassian-cli confluence labels 123456
```

## JQL Examples

Jira Query Language (JQL) examples for searching:

```bash
# Issues assigned to you
"assignee=currentUser()"

# Open bugs in specific project
"project=PROJ AND issuetype=Bug AND status=Open"

# Updated in last 7 days
"updated >= -7d"

# High priority issues
"priority=High ORDER BY created DESC"

# Issues with specific labels
"labels=urgent"

# Date range
"created >= \"2024-01-01\" AND created <= \"2024-12-31\""
```

## CQL Examples

Confluence Query Language (CQL) examples for searching:

```bash
# Pages in specific space
"type=page AND space=DEV"

# Text search
"siteSearch ~ \"important concept\""

# Pages with specific label
"type=page AND label=documentation"

# Recent pages
"type=page AND created >= \"2024-01-01\""

# Pages by specific user
"type=page AND creator=john.doe"

# Personal space search (requires quoting)
"type=page AND space=\"~username\""
```

## Using with Claude Code

This CLI is designed to work seamlessly as a Claude Code skill.

### Skill Setup

The CLI is already configured as a Claude Skill. The skill definition is located at:

```
.claude/skills/atlassian-search/SKILL.md
```

### Using the Skill

Once the binary is built, Claude Code will automatically detect and use the skill. Simply ask Claude to:

- "Search Jira for open bugs in project DEV"
- "Find Confluence documentation about API authentication"
- "Show me issues assigned to me in the current sprint"
- "Search for security-related pages in Confluence"

### Manual Usage Examples

You can also use the CLI directly:

```bash
# Search Jira issues
atlassian-cli jira search "assignee=currentUser() AND status=Open"

# Search Confluence
atlassian-cli confluence text-search "API documentation" --full-content

# Get issue details
atlassian-cli jira issue PROJECT-123
```

### Programmatic Usage

For programmatic integration:

```typescript
import { exec } from 'child_process';
import { promisify } from 'util';

const execAsync = promisify(exec);

async function searchJiraIssues(jql: string) {
  const { stdout } = await execAsync(`atlassian-cli jira search "${jql}" --format=json`);
  return JSON.parse(stdout);
}

async function getConfluencePage(pageId: string) {
  const { stdout } = await execAsync(`atlassian-cli confluence page ${pageId} --format=json`);
  return JSON.parse(stdout);
}
```

## Output Format

Commands print human-readable **text by default**. Pass `--format=json` to get the raw API response, which is easy to parse and process:

```bash
# Pretty print with jq (note --format=json)
atlassian-cli jira issue PROJECT-123 --format=json | jq

# Extract specific fields
atlassian-cli jira search "project=DEV" --format=json | jq '.issues[].key'

# Count results
atlassian-cli confluence spaces --format=json | jq '.results | length'
```

Confluence search commands also accept `--full-content` to include full page bodies in the text output instead of previews.

## Development

### Build
```bash
zig build
```

### Run without installing
```bash
zig build run -- jira projects
zig build run -- confluence spaces
```

### Run tests
```bash
zig build test
```

## Architecture

The CLI is structured in layers:

- **atlassian_client.zig**: Core HTTP client with Basic Auth
- **jira_client.zig**: Jira-specific API methods
- **confluence_client.zig**: Confluence-specific API methods
- **main.zig**: CLI argument parsing and command routing

## API Coverage

### Jira
- ✅ Get issue details
- ✅ Search with JQL
- ✅ List projects
- ✅ Get project issues
- ✅ List agile boards
- ✅ List sprints
- ✅ Get sprint issues
- ✅ Get current user

### Confluence
- ✅ Get page by ID
- ✅ Search with CQL
- ✅ List spaces
- ✅ Get space details
- ✅ Get child pages
- ✅ Get page comments
- ✅ Get page labels

## Troubleshooting

### Authentication Errors

If you get `401 Unauthorized`:
- Verify your `ATLASSIAN_API_TOKEN` is correct
- Check that `ATLASSIAN_USERNAME` is your email address
- Ensure the token hasn't expired

### Permission Errors

If you get `403 Forbidden`:
- Check your user has the required permissions
- Verify you have access to the project/space
- Contact your Atlassian admin if needed

### Not Found Errors

If you get `404 Not Found`:
- Verify the issue key, page ID, or space key is correct
- Check you have permission to view the resource
- Ensure you're using the correct `ATLASSIAN_URL`

## License

MIT

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## References

- [Jira REST API Documentation](https://developer.atlassian.com/cloud/jira/platform/rest/v3/)
- [Confluence REST API Documentation](https://developer.atlassian.com/cloud/confluence/rest/v1/)
- [JQL Documentation](https://support.atlassian.com/jira-service-management-cloud/docs/use-advanced-search-with-jira-query-language-jql/)
- [CQL Documentation](https://developer.atlassian.com/cloud/confluence/advanced-searching-using-cql/)
