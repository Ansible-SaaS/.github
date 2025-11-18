# Claude Code Configuration

This directory contains recommended configuration files for [Claude Code](https://claude.com/claude-code).

## Files

### settings-permissions-deny-example.json

This file contains recommended `permissions.deny` settings to prevent Claude Code from accessing sensitive files across all your projects.

## What is permissions.deny?

The `permissions.deny` setting in Claude Code allows you to specify file patterns that Claude Code should never access. Files matching these patterns will be **completely invisible to Claude Code**, preventing any accidental exposure of sensitive data.

This configuration blocks access to:
- **Credentials and Secrets**: `.env` files, SSH keys, AWS credentials, GPG keys, service account files
- **Personal Files**: diary, notes, personal directories
- **System Files**: `.DS_Store`, swap files, temp files
- **Dependencies**: `node_modules`, `vendor`, virtual environments, `__pycache__`
- **Build Artifacts**: `dist`, `build`, `.next`, minified files
- **IDE Files**: workspace settings, IDE-specific configs
- **Logs and Databases**: `.log`, `.db`, `.sqlite` files
- **Large Binary Files**: ISOs, archives, executables
- **Document Files**: PDFs, Office documents
- **Media Files**: Videos, audio files
- **Git Internals**: Git object store, logs

## Installation

### Option 1: Merge with existing settings.json

If you already have a `~/.claude/settings.json` file with other settings:

1. Open your existing `~/.claude/settings.json`
2. Copy the `permissions` object from `settings-permissions-deny-example.json`
3. Merge it with your existing settings:

```json
{
  "alwaysThinkingEnabled": false,
  "permissions": {
    "deny": [
      "Read(**/.env)",
      "Read(**/.env.*)",
      ...
    ]
  }
}
```

### Option 2: Copy for new installation

If you don't have a `~/.claude/settings.json` file yet:

```bash
# Copy the example file to your Claude settings directory
cp settings-permissions-deny-example.json ~/.claude/settings.json
```

### Option 3: Manual merge using jq (if you have it installed)

```bash
# Backup existing settings
cp ~/.claude/settings.json ~/.claude/settings.json.backup

# Merge permissions from example into your settings
jq -s '.[0] * .[1]' ~/.claude/settings.json settings-permissions-deny-example.json > ~/.claude/settings-temp.json
mv ~/.claude/settings-temp.json ~/.claude/settings.json
```

## Testing the Configuration

After installing the configuration, you can verify it's working:

1. **Restart Claude Code** to ensure settings are loaded

2. **Use the `/permissions` command** in Claude Code to check your permission configuration

3. **Test with a denied file**: Create a test `.env` file and ask Claude to read it. It should be blocked:
   ```bash
   echo "SECRET=test123" > /tmp/test/.env
   ```
   Then ask Claude Code to read `/tmp/test/.env` - it should not be able to access it.

## Customization

You can customize the deny patterns based on your specific needs:

- Add patterns for project-specific sensitive files
- Remove patterns for files you want Claude to access
- Use glob patterns (`**`) to match files in any directory
- Use specific paths for files in particular locations

### Example customizations:

```json
{
  "permissions": {
    "deny": [
      "Read(**/.env)",
      "Read(**/my-company-secrets/**)",
      "Read(/specific/path/to/sensitive/file.txt)"
    ]
  }
}
```

## Documentation

For more information about Claude Code permissions, see:
- [Claude Code Settings Documentation](https://code.claude.com/docs/en/settings)
- [Claude Code Documentation](https://code.claude.com/docs)

## Notes

- The `permissions.deny` setting replaces the deprecated `ignorePatterns` configuration
- Settings changes may require restarting Claude Code to take effect
- Denied files are completely invisible to Claude Code and cannot be accidentally accessed
- Project-specific `.claudeignore` files (if supported in future versions) would work alongside these global settings
