# General instructions

## Git Workflow

**IMPORTANT**: Never commit directly to the `main` branch. Always create a feature branch before making any commits.

### Creating Branches and Pull Requests

1. **Update main branch** before creating a new branch:
   ```bash
   git checkout main
   git pull origin main
   ```
   **IMPORTANT**: Always ensure your main branch is up-to-date before creating a new feature branch.

2. **Create a branch** from the updated main branch:
   ```bash
   git checkout -b <JIRA-KEY>-<short-description>
   ```
   Example: `git checkout -b AAP-12345-fix-validation`

3. **Make your changes** and commit them to the branch

4. **Push the branch** to remote:
   ```bash
   git push -u origin <branch-name>
   ```

5. **Create a draft PR/MR** using the template from @.github/PULL_REQUEST_TEMPLATE.md
   - For GitHub repositories: Use `gh` CLI
   - For GitLab repositories: Use `glab` CLI

### Creating Pull/Merge Requests

#### For GitLab Repositories (using glab)

**Installation:**
```bash
# macOS
brew install glab

# Other platforms: https://gitlab.com/gitlab-org/cli/-/releases
```

**Authentication:**
```bash
# Interactive authentication
glab auth login --hostname gitlab.cee.redhat.com

# Or with a token
export GITLAB_TOKEN="your-gitlab-token"
glab auth login --hostname gitlab.cee.redhat.com --token $GITLAB_TOKEN
```

**Authenticating to Internal GitLab:**

For Red Hat's internal GitLab instance, you may need to use the full authentication command:

```bash
# Full authentication with explicit parameters
glab auth login --hostname gitlab.cee.redhat.com \
  --api-host gitlab.cee.redhat.com \
  --api-protocol https \
  --git-protocol git \
  -t $GITLAB_TOKEN
```

**Note**: To determine the correct git-protocol to use:
```bash
# Check the remote URL format
git remote -v
```
- If the remote uses `git@gitlab.cee.redhat.com:...`, use `--git-protocol git`
- If the remote uses `https://gitlab.cee.redhat.com/...`, use `--git-protocol https`

**Creating a Merge Request:**
```bash
glab mr create --draft --title "Your MR Title" --description "$(cat <<'EOF'
Jira Issue: https://issues.redhat.com/browse/AAP-NNNN

## Description

[Describe your changes here]

Assisted-by: Claude (Anthropic)

## Testing

### Steps to test
1. Pull down the PR
2. [Add specific test steps]
3. [Additional steps]

### Scenarios tested
- [ ] Test scenario 1
- [ ] Test scenario 2

## Deployment considerations
- [ ] This code change is ready for deployment on its own
- [ ] This code change requires the following considerations before being deployed:

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

**Important Notes:**
- Always use the template format shown above (from https://github.com/Ansible-SaaS/.github/blob/main/.github/PULL_REQUEST_TEMPLATE.md)
- Do NOT use `--fill` flag together with `--title` and `--description` (they are mutually exclusive in glab)
- For internal tooling improvements without a JIRA ticket, use "Jira Issue: N/A (Internal tooling improvement)"
- Always include "Assisted-by: Claude (Anthropic)" when AI assistance is used
- The MR body must follow the template structure

**Note**: To create a personal access token for GitLab:
1. Go to GitLab Settings > Access Tokens
2. Create a token with `api` scope
3. Save the token securely

#### For GitHub Repositories (using gh)

**Creating a Pull Request:**
```bash
gh pr create --draft --title "Title" --body "$(cat <<'EOF'
...
EOF
)"
```

### Commit Message Format

When creating commits, follow this format:

```bash
git commit -m "$(cat <<'EOF'
Brief summary of changes (imperative mood, < 50 chars)

More detailed explanation if needed. Explain what and why, not how.
- Bullet points are acceptable
- Use present tense

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

**IMPORTANT**:
- All commits made with AI assistance MUST include the `Co-Authored-By` field (GitHub standard)
- Format: `Co-Authored-By: <Name> <email>` (e.g., `Co-Authored-By: Claude <noreply@anthropic.com>`)

### Updating Existing Pull/Merge Requests

**IMPORTANT**: When pushing additional commits to a branch that already has an open PR/MR, you MUST update it to reflect the changes:

1. **Update the PR/MR title** if the scope or focus of the changes has evolved
2. **Update the PR/MR body** to document what functionality was:
   - Added (new features or capabilities)
   - Changed (modifications to existing functionality)
   - Deleted (removed features or code)

**The PR/MR body MUST continue to follow the template structure** from [PULL_REQUEST_TEMPLATE.md](https://github.com/Ansible-SaaS/.github/blob/main/.github/PULL_REQUEST_TEMPLATE.md).

This ensures reviewers have a clear understanding of all changes without having to parse through individual commits.

#### For GitLab (using glab):
```bash
# Update MR title
glab mr update <MR-NUMBER> --title "Updated title reflecting all changes"

# Update MR description (use a file for complex updates)
# IMPORTANT: Maintain the template structure and include Assisted-by field
glab mr update <MR-NUMBER> --description "$(cat <<'EOF'
Jira Issue: <https://issues.redhat.com/browse/AAP-NNNN>

## Description
- Added: New validation for instance names
- Changed: Refactored error handling in subscription service
- Deleted: Deprecated legacy API endpoints

[Additional context about the changes]

## Testing
### Steps to test
1. Pull down the MR
2. ...

### Scenarios tested
[Describe tested scenarios]

## Deployment considerations
- [ ] This code change is ready for deployment on its own
- [ ] This code change requires the following considerations before being deployed:

---
🤖 Generated with [Claude Code](https://claude.com/claude-code)

Assisted-by: Claude
EOF
)"
```

#### For GitHub (using gh):
```bash
# Update PR title
gh pr edit <PR-NUMBER> --title "Updated title reflecting all changes"

# Update PR body (use a file for complex updates)
gh pr edit <PR-NUMBER> --body "$(cat <<'EOF'
...
EOF
)"
```

**IMPORTANT**: All PRs/MRs created with AI assistance MUST include the `Assisted-by:` field at the end of the description.

### Branch Naming Convention

Format: `<JIRA-KEY>-<short-description>`
- Use lowercase with hyphens
- Keep the description concise but meaningful
- Examples:
  - `AAP-53659-keep-reason-debugging`
  - `AAP-57911-instance-name-validation`
  - `AAP-12345-fix-subscription-status`

## GITHUB Pull Requests

The template @.github/PULL_REQUEST_TEMPLATE.md must be used to create pull requests.
The PR must be created as draft.

### AI Assistant Attribution

When creating pull requests, the `Assisted-by: <name of code assistant>` field in the PR description must be updated with the name of the code assistant used (e.g., "Cursor AI", "GitHub Copilot", "Claude", etc.).

**Note**: Use the `Assisted-by:` tag instead of `Co-Authored-by:` for AI assistant attribution.

## JIRA

### General Guidelines

**IMPORTANT**: ALL JIRA issue management operations (creating issues, updating fields, adding comments, linking PRs, etc.) MUST be performed using the JIRA REST API with curl commands.

**DO NOT USE THE `jira` CLI TOOL** - The `jira` CLI tool must NEVER be used for automated operations because:
- It does not provide reliable visibility control (cannot guarantee "Red Hat Employee" restriction)
- It does not properly validate required custom fields
- It does not support all custom field types properly
- It can create issues that are publicly visible by default
- Error handling and validation are inconsistent

**DO NOT USE** the JIRA web interface for automated operations - it is for manual use only.

**ALWAYS USE** the JIRA REST API with curl commands as documented below.

### Environment Variables

**Required environment variables for JIRA API access:**
- `JIRA_API_TOKEN`: Your JIRA API token (Personal Access Token)
- `JIRA_AUTH_TYPE`: Authentication type, must be set to `Bearer` for API token authentication

**Example:**
```bash
export JIRA_API_TOKEN="your-api-token-here"
export JIRA_AUTH_TYPE="Bearer"
```

**IMPORTANT**: All JIRA curl commands MUST use the authorization header format based on `JIRA_AUTH_TYPE`. When `JIRA_AUTH_TYPE=Bearer`, use:
```bash
-H "Authorization: $JIRA_AUTH_TYPE $JIRA_API_TOKEN"
```

Required for each created issue:
- **Priority**: Must be set (Critical, Major, Normal, Minor)
- **Workstream**: Must be set to 'SaaS'
- **Component**: Must be 'ansible-saas'
- **Acceptance Criteria**: Mandatory custom field that must be set
- **Visibility**: All issues MUST be restricted to "Red Hat Employee"

**IMPORTANT**: All comments MUST be restricted to Red Hat employees using the API visibility controls.

**IMPORTANT**: For Bug type issues, the affected version MUST be set to "ansible-saas-ga".

### Linking Pull Requests to JIRA Issues

When a pull request is created for a JIRA issue, update the issue with the PR link using the JIRA REST API:

```bash
# Link single PR to JIRA issue
curl -X PUT "https://issues.redhat.com/rest/api/2/issue/AAP-12345" \
  -H "Authorization: $JIRA_AUTH_TYPE $JIRA_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "fields": {
      "customfield_12310220": "https://github.com/Ansible-SaaS/ansible-saas-management-service/pull/552"
    }
  }'
```

**IMPORTANT**: When adding multiple PRs to the same issue (e.g., code PR and documentation PR), the URLs must be **comma-separated** in a single value. Do NOT run the command multiple times as it will overwrite the previous value.

Example with multiple PRs (comma-separated):
```bash
curl -X PUT "https://issues.redhat.com/rest/api/2/issue/AAP-12345" \
  -H "Authorization: $JIRA_AUTH_TYPE $JIRA_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "fields": {
      "customfield_12310220": "https://github.com/Ansible-SaaS/ansible-saas-management-service/pull/552,https://github.com/Ansible-SaaS/ansible-saas-sops/pull/293"
    }
  }'
```

**Workflow**:
1. First, fetch the current PR links (if any) using `GET /rest/api/2/issue/{key}`
2. Append the new PR URL with a comma separator
3. Update using `PUT /rest/api/2/issue/{key}`

**Note**: `customfield_12310220` is the "git-pull-request" field in Red Hat JIRA. This may vary for other JIRA instances.

### Retrieving JIRA Issues

To retrieve issue details including current field values:

```bash
curl -s -X GET "https://issues.redhat.com/rest/api/2/issue/AAP-59069" \
  -H "Authorization: $JIRA_AUTH_TYPE $JIRA_API_TOKEN" \
  -H "Content-Type: application/json"
```

This is useful for:
- Checking current PR links before updating
- Verifying field values
- Retrieving issue status and details

### Creating Issues with JIRA REST API

#### Creating a Bug

**IMPORTANT**:
- For Bug type issues, the affected version MUST be set to "ansible-saas-ga"
- Bugs MUST have Workstream set to "SaaS" using `customfield_12319275`
- To link a bug to an epic, include `customfield_12311140` with the epic key

```bash
curl -X POST "https://issues.redhat.com/rest/api/2/issue" \
  -H "Authorization: $JIRA_AUTH_TYPE $JIRA_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "fields": {
      "project": {"key": "AAP"},
      "issuetype": {"name": "Bug"},
      "summary": "Issue summary",
      "priority": {"name": "Major"},
      "components": [{"name": "ansible-saas"}],
      "customfield_12319275": [{"value": "SaaS"}],
      "customfield_12315940": "- First acceptance criterion\n- Second acceptance criterion",
      "customfield_12311140": "AAP-12345",
      "versions": [{"name": "ansible-saas-ga"}],
      "description": "*Description*\n\nWhat is happening\n\n*Steps to Reproduce*\n\n1. Step one\n2. Step two\n\n*Expected Behavior*\n\nWhat should happen"
    }
  }'
```

**Note**: To link to an epic, include the `customfield_12311140` field in the initial creation (as shown above). You can also add it later with a PUT request if needed.

#### Creating a Story

```bash
curl -X POST "https://issues.redhat.com/rest/api/2/issue" \
  -H "Authorization: $JIRA_AUTH_TYPE $JIRA_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "fields": {
      "project": {"key": "AAP"},
      "issuetype": {"name": "Story"},
      "summary": "Story summary",
      "priority": {"name": "Major"},
      "components": [{"name": "ansible-saas"}],
      "customfield_12311140": "AAP-12345",
      "customfield_12319275": [{"value": "SaaS"}],
      "customfield_12315940": "- Acceptance criterion 1\n- Acceptance criterion 2",
      "description": "h3. *User Story*\n\nAs a user I want to...\n\nh3. *Supporting documentation*\n\nLinks to docs..."
    }
  }'
```

**Important Notes:**
- `customfield_12311140` is the "Epic Link" field (string value of epic key, e.g., "AAP-12345")
- `customfield_12319275` is the "Workstream" field (array of objects with value, e.g., `[{"value": "SaaS"}]`)
- `customfield_12315940` is the "Acceptance Criteria" field (text with JIRA markup)
- After creation, you MUST manually set visibility to "Red Hat Employee" in the JIRA web interface
- **DO NOT use "parent" field** - use customfield_12311140 for epic linking
- Use `\n` for newlines in description field
- Always verify the created issue in JIRA web interface

#### Adding Comments to JIRA Issues

**IMPORTANT**: All comments MUST be restricted to Red Hat employees. Always use the JIRA REST API with visibility set to `"type": "group"`, `"value": "Red Hat Employee"`.

```bash
# Simple comment
curl -X POST "https://issues.redhat.com/rest/api/2/issue/AAP-12345/comment" \
  -H "Authorization: $JIRA_AUTH_TYPE $JIRA_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "body": "Your comment text here",
    "visibility": {
      "type": "group",
      "value": "Red Hat Employee"
    }
  }'

# Long comment with JIRA markup (use a file)
cat > /tmp/comment.json << 'EOF'
{
  "body": "h3. Section Header\n\nContent with {{inline code}}.\n\n{code:bash}\ncode example\n{code}\n\n* Bullet points\n* Work well",
  "visibility": {
    "type": "group",
    "value": "Red Hat Employee"
  }
}
EOF
curl -X POST "https://issues.redhat.com/rest/api/2/issue/AAP-12345/comment" \
  -H "Authorization: $JIRA_AUTH_TYPE $JIRA_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d @/tmp/comment.json
```

**Important Notes:**
- Use JIRA markup syntax in comments (e.g., `*bold*`, `_italic*`, `{code:java}...{code}`, `h3.` for headers)
- In API calls, use `\n` for newlines in the body field
- The `visibility` object with `"type": "group"` and `"value": "Red Hat Employee"` properly restricts access
- Requires `JIRA_API_TOKEN` environment variable to be set
- Always verify comment visibility in JIRA web interface (should show "Internal" badge)

#### Updating Issue Descriptions

When updating the description field of an existing JIRA issue, use the JIRA REST API:

```bash
# Update issue description with JIRA markup
curl -X PUT "https://issues.redhat.com/rest/api/2/issue/AAP-12345" \
  -H "Authorization: $JIRA_AUTH_TYPE $JIRA_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "fields": {
      "description": "h2. Section Header\n\nContent goes here...\n\nh3. Subsection\n\nMore content with {{inline code}} and formatting.\n\n{code:bash}\ncode block example\n{code}"
    }
  }'

# For long descriptions, use a file
cat > /tmp/update.json << 'EOF'
{
  "fields": {
    "description": "h2. Section Header\n\nVery long content...\n\n{code:bash}\nexample\n{code}"
  }
}
EOF
curl -X PUT "https://issues.redhat.com/rest/api/2/issue/AAP-12345" \
  -H "Authorization: $JIRA_AUTH_TYPE $JIRA_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d @/tmp/update.json
```

**Important Notes:**
- Use `\n` for newlines in the description field
- JIRA markup works the same as in comments
- Test the result by viewing the issue in JIRA web interface
- Use PUT to update specific fields without affecting others

### Templates

The following templates must be used for the description when a new issue is created.
If you don't find the template here, please request to update this document with the associated template.

#### Epic

```
h2. *Background*

{color:#0747a6}_Initial completion during New status and then remove this blue text._{color}

<fill out any context, value prop, description needed>
h2. *User Stories*

{color:#0747a6}_Initial completion during New status and then remove this blue text._{color}

Format: "as a <type of user> I want <some goal> so that <some reason>"
h2. *Supporting documentation*

{color:#0747a6}_Initial completion during New status and then remove this blue text._{color}

<include links to technical docs, diagrams, etc>
```

#### Spike
Each Spike must be linked to an Epic.


```
h3. *User Story*

Format: "as a <type of user> I want <some goal> so that <some reason>"
h3. *Supporting documentation*

<include links to technical docs, diagrams, etc>
```

#### Story
Each Story must be linked to an Epic

```
h3. *User Story*

Format: "as a <type of user> I want <some goal> so that <some reason>"
h3. *Supporting documentation*

<include links to technical docs, diagrams, etc>
```

#### Bug

```
*Description*

_<what is happening, why are you requesting this update>_

*Steps to Reproduce*

_<list explicit steps to reproduce, for docs bugs, include the error/issue>_

*Actual Behavior*

_<what is currently happening, for docs bugs include link(s) to relevant section(s)>_

*Expected Behavior*

_<what should happen? for docs bugs, provide suggestion(s) of how to improve the content>_

*Additional Context*

<{_}Provide any related communication on this issue.>{_}
```

#### Task

```
h3. *Problem Description*

<what is the issue, what is being asked, what is expected>
h3. *Supporting documentation*
```


## Documentation

The following Standard Operating Procedures (SOPs) documentation provides important information about the Ansible SaaS platform:

[Ansible-SaaS SOPS Index](https://github.com/Ansible-SaaS/ansible-saas-sops/blob/main/README.md)

## Fetching Files from Private GitHub Repositories

**IMPORTANT**: When you need to fetch files from private GitHub repositories (such as this AGENT.md file or other documentation), you MUST use the GitHub REST API with authentication via the `GITHUB_TOKEN` environment variable.

### Using GitHub REST API to Fetch File Contents

The `GITHUB_TOKEN` environment variable should be set with a valid GitHub Personal Access Token that has access to private repositories.

**To fetch raw file contents:**

```bash
curl -H "Accept: application/vnd.github.raw" \
  -H "Authorization: Bearer ${GITHUB_TOKEN}" \
  -H "X-GitHub-Api-Version: 2022-11-28" \
  "https://api.github.com/repos/{OWNER}/{REPO}/contents/{PATH}"
```

**Example - Fetching this AGENT.md file:**

```bash
curl -H "Accept: application/vnd.github.raw" \
  -H "Authorization: Bearer ${GITHUB_TOKEN}" \
  -H "X-GitHub-Api-Version: 2022-11-28" \
  "https://api.github.com/repos/Ansible-SaaS/.github/contents/AGENT.md"
```

**Example - Fetching a project-specific AGENT.md:**

```bash
curl -H "Accept: application/vnd.github.raw" \
  -H "Authorization: Bearer ${GITHUB_TOKEN}" \
  -H "X-GitHub-Api-Version: 2022-11-28" \
  "https://api.github.com/repos/Ansible-SaaS/ansible-saas-management-service/contents/AGENT.md"
```

**Important Notes:**
- The `Accept: application/vnd.github.raw` header returns the raw file content directly (not base64 encoded)
- Always use the `Authorization: Bearer ${GITHUB_TOKEN}` header for private repositories
- The API endpoint format is: `/repos/{owner}/{repo}/contents/{path}`
- For files in subdirectories, include the full path (e.g., `docs/architecture/README.md`)
- Do NOT use blob URLs (e.g., `https://github.com/.../blob/main/...`) as they return HTML, not file content
- If `GITHUB_TOKEN` is not set or invalid, you will receive a 401 Unauthorized or 404 Not Found error

### When to Use This Method

Use the GitHub REST API to fetch files when:
1. Reading referenced documentation from other private repositories
2. Retrieving templates or configuration files from organization repositories
3. Accessing files that are referenced in project-specific AGENT.md files
4. Any automated operation that needs to read files from private GitHub repositories

**Do NOT** use this for public repositories where direct URL fetching would work without authentication.

