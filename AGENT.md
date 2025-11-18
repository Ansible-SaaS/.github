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

5. **Create a draft PR** using the template from @.github/PULL_REQUEST_TEMPLATE.md

### Updating Existing Pull Requests

**IMPORTANT**: When pushing additional commits to a branch that already has an open PR, you MUST update the PR to reflect the changes:

1. **Update the PR title** if the scope or focus of the changes has evolved
2. **Update the PR body** to document what functionality was:
   - Added (new features or capabilities)
   - Changed (modifications to existing functionality)
   - Deleted (removed features or code)

**The PR body MUST continue to follow the template structure** from [PULL_REQUEST_TEMPLATE.md](https://github.com/Ansible-SaaS/.github/blob/main/.github/PULL_REQUEST_TEMPLATE.md).

This ensures reviewers have a clear understanding of all changes in the PR without having to parse through individual commits.

You can update the PR using the GitHub CLI:
```bash
# Update PR title
gh pr edit <PR-NUMBER> --title "Updated title reflecting all changes"

# Update PR body (use a file for complex updates)
# IMPORTANT: Maintain the template structure
gh pr edit <PR-NUMBER> --body "$(cat <<'EOF'
Jira Issue: <https://issues.redhat.com/browse/AAP-NNNN>

## Description
- Added: New validation for instance names
- Changed: Refactored error handling in subscription service
- Deleted: Deprecated legacy API endpoints

[Additional context about the changes]

## Testing
### Steps to test
1. Pull down the PR
2. ...

### Scenarios tested
[Describe tested scenarios]

## Deployment considerations
- [ ] This code change is ready for deployment on its own
- [ ] This code change requires the following considerations before being deployed:
EOF
)"
```

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

## JIRA

Use the `jira` CLI to create new issue.
For each created issue, the priority must be set (Critical, Major, Normal, Minor ), if it is not provided, you must ask for it.
The workstream must be set to 'SaaS'.
The component must be 'ansible-saas'.
The custom field 'acceptance-criteria' is a mandatory field and so must be set.
**IMPORTANT**: All issues MUST have their visibility restricted to "Red Hat Employee". Note: The `--custom security="Red Hat Employee"` flag is not currently supported by the jira CLI, so this must be set manually in the JIRA web interface after creation.
**IMPORTANT**: All comments MUST be restricted to Red Hat employees using the `--internal` flag.
**IMPORTANT**: For Bug type issues, the affected version MUST be set to "ansible-saas-ga" using `--affects-version ansible-saas-ga`.

### Linking Pull Requests to JIRA Issues

When a pull request is created for a JIRA issue, update the issue with the PR link using:

```bash
jira issue edit <ISSUE-KEY> --no-input --custom git-pull-request="<PR-URL>"
```

**IMPORTANT**: When adding multiple PRs to the same issue (e.g., code PR and documentation PR), the URLs must be **comma-separated** in a single command. Do NOT run the command multiple times as it will overwrite the previous value.

Example with single PR:
```bash
jira issue edit AAP-57740 --no-input --custom git-pull-request="https://github.com/Ansible-SaaS/ansible-saas-sre/pull/1279"
```

Example with multiple PRs (comma-separated):
```bash
jira issue edit AAP-57911 --no-input --custom git-pull-request="https://github.com/Ansible-SaaS/ansible-saas-management-service/pull/552,https://github.com/Ansible-SaaS/ansible-saas-sops/pull/293"
```

**Workflow**: Always check if the issue already has a PR link before adding a new one. If it does, append the new PR URL with a comma separator.

### Usage Examples

#### Custom Field Usage Example

When creating or editing issues, use the `--custom` flag with the field name in lowercase with dashes:

```bash
# Creating a simple issue (short body)
jira issue create --type Story --project AAP --parent AAP-12345 \
  --priority Major \
  --summary "Issue summary" \
  --component ansible-saas \
  --custom workstream=SaaS \
  --custom security="Red Hat Employee" \
  --custom acceptance-criteria="- First acceptance criterion
- Second acceptance criterion
- Third acceptance criterion" \
  --body "h3. *User Story*
..."

# RECOMMENDED: Creating an issue with complex JIRA markup using --template
# This approach is more reliable and avoids timeout issues with complex body text
cat > /tmp/issue-body.txt << 'EOF'
*Description*

This is the issue description with JIRA markup.

*Steps to Reproduce*

1. Step one
2. Step two

{code:bash}
example command
{code}

*Expected Behavior*

What should happen
EOF
jira issue create --type Bug --project AAP \
  --priority Major \
  --summary "Issue summary" \
  --component ansible-saas \
  --custom workstream=SaaS \
  --custom acceptance-criteria="- Acceptance criterion" \
  --affects-version ansible-saas-ga \
  --template /tmp/issue-body.txt \
  --no-input

# Editing an existing issue to set acceptance criteria
jira issue edit AAP-12345 --no-input \
  --custom acceptance-criteria="- Updated criterion 1
- Updated criterion 2"
```

**Important Notes:**
- For issue bodies with complex JIRA markup or multiple lines, use the `--template` file approach
- The `--template` approach is more reliable than inline `--body` for complex content
- Simple, short bodies can still use inline `--body "..."`
- Always test by viewing the created issue: `jira issue view AAP-XXXXX`

#### Adding Comments to JIRA Issues

To add a comment to an existing JIRA issue, use the `jira issue comment add` command with the `--internal` flag to restrict visibility to Red Hat employees:

```bash
# Simple comment with internal flag (REQUIRED)
jira issue comment add AAP-12345 "This is a simple comment" --internal

# RECOMMENDED: Using a template file for complex/long comments with JIRA markup
# This approach is more reliable and avoids timeout issues
cat > /tmp/comment.txt << 'EOF'
h3. Section Header

Content of the comment with {{inline code}}.

{code:bash}
code block example
{code}

* Bullet points
* Work well too
EOF
jira issue comment add AAP-12345 --template /tmp/comment.txt --internal --no-input

# AVOID: Heredoc with command substitution for long comments (can timeout)
# This may work for short comments but often times out with complex JIRA markup
jira issue comment add AAP-12345 "$(cat <<'EOF'
This is a multi-line comment.
It can contain multiple paragraphs.
EOF
)" --internal
```

**Important Notes:**
- Use JIRA markup syntax in comments (e.g., `*bold*`, `_italic_`, `{code:java}...{code}`, `h3.` for headers)
- **For comments with JIRA markup or longer than a few lines, ALWAYS use the `--template` file approach**
- Heredoc comments with complex JIRA markup frequently timeout (3+ minutes)
- File-based comments complete successfully and quickly

#### Updating Issue Descriptions

When updating the description field of an existing JIRA issue, you MUST use a heredoc with command substitution to avoid formatting issues:

```bash
# CORRECT: Use heredoc with command substitution
jira issue edit AAP-12345 --no-input -b "$(cat <<'EOFBODY'
h2. Section Header

Content goes here...

h3. Subsection

More content with {{inline code}} and formatting.

{code:bash}
code block example
{code}
EOFBODY
)"

# INCORRECT: Piping from stdin often fails with complex formatting
cat /tmp/description.txt | jira issue edit AAP-12345 -b - --no-input

# INCORRECT: Using -b - with stdin can result in malformed descriptions
echo "content" | jira issue edit AAP-12345 --no-input -b -
```

**Important Notes:**
- Always use single quotes in `<<'EOFBODY'` to prevent shell variable expansion
- The heredoc must be wrapped in `"$(cat <<'EOFBODY' ... EOFBODY)"` for proper formatting
- Test the result by viewing the issue after update: `jira issue view AAP-12345`
- For very long descriptions, the heredoc approach is more reliable than stdin piping

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

