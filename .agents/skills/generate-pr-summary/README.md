# 📋 Generate PR Summary

A Gemini CLI skill that generates human-readable PR review summaries from GitHub compare URLs using only the `gh` CLI.

## What It Does

Give it a GitHub compare URL (tags, branches, or commits) and it will:

1. Fetch the comparison as **structured JSON** via `gh api`
2. Analyze all file changes (supports 300+ files)
3. Generate a categorized Markdown summary with stats, key changes, and breaking changes

## Prerequisites

- [GitHub CLI (`gh`)](https://cli.github.com/) installed
- Authenticated: `gh auth login`

## Usage

Tell Gemini CLI to summarize a comparison:

```
Summarize this diff: https://github.com/owner/repo/compare/v1.0.0...v2.0.0
```

Or provide the parts separately:

```
Generate a PR summary for owner/repo comparing main...feature-branch
```

### Example Output

The skill produces a `PR_Review_Summary_<comparison>.md` file with:

- **Executive Summary** — high-level overview
- **Key Changes** — grouped by category (features, fixes, refactoring, deps)
- **Breaking Changes** — highlighted prominently
- **File Change Summary** — table with per-file stats

## Installation for Gemini CLI

Install **globally** so the skill is available in any project/directory.

### Option 1 — Manual global install (recommended)

Copy the skill folder to your Gemini CLI user skills directory:

**Linux / macOS:**
```bash
mkdir -p ~/.gemini/skills
cp -r generate-pr-summary ~/.gemini/skills/
```

**Windows PowerShell:**
```powershell
Copy-Item -Recurse generate-pr-summary "$env:USERPROFILE\.gemini\skills\generate-pr-summary"
```

### Option 2 — Install from Git

```bash
gemini skills install https://github.com/<owner>/git-diff.git --path .agents/skills/generate-pr-summary
```

### Option 3 — Install from local directory

```bash
gemini skills install /path/to/git-diff/.agents/skills/generate-pr-summary
```

### Option 4 — Link an entire skills repo

If you maintain multiple skills in one repo, link the whole directory:

```bash
gemini skills link /path/to/git-diff/.agents/skills
```

This creates symlinks in `~/.gemini/skills/` for every skill discovered in that directory.

### Verify Installation

```bash
# List all installed skills
gemini skills list

# Or in an interactive session
gemini
> /skills list
```

You should see `Generate PR Summary` in the list. Test it:
```
> Summarize this diff: https://github.com/owner/repo/compare/v1.0.0...v2.0.0
```

### Managing the Skill

```bash
# Disable the skill
gemini skills disable generate-pr-summary

# Re-enable it
gemini skills enable generate-pr-summary

# Uninstall
gemini skills uninstall generate-pr-summary
```

## Cross-OS Support

Works on **Windows**, **macOS**, and **Linux**. The skill provides OS-specific commands for temp file handling and cleanup.

## Limitations

- Requires `gh` CLI authentication (no anonymous access)
- Very large diffs may need chunked processing (handled automatically)
- Binary file diffs won't include patch content
