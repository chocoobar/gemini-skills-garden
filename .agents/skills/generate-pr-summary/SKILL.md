---
name: Generate PR Summary
description: Generates a human-readable PR review summary in Markdown from a GitHub compare URL (tags, branches, or commits) using only the gh CLI.
---

# Generate PR Summary Skill

This skill takes a GitHub compare URL (e.g., `https://github.com/OWNER/REPO/compare/v0.4.8...v0.4.9`) and generates a polished, human-readable PR Review Summary in Markdown. It uses **only the `gh` CLI** and works on any OS (Windows, macOS, Linux).

> **Note**: `gh pr diff` only works for pull requests. This skill uses `gh api` with the compare endpoint, which works for **any** tag, branch, or commit comparison — including repos where no PR exists.

## Trigger

The user provides a GitHub compare URL or specifies an `OWNER/REPO` with a comparison string (e.g., `v1.0.0...v2.0.0`, `main...feature-branch`).

## Prerequisites

- `gh` CLI must be installed and authenticated (`gh auth login`).

## Instructions

When the user asks you to execute this skill, follow these steps precisely:

### Step 1 — Parse and Validate

1. Extract `OWNER`, `REPO`, and `COMPARE_STRING` (e.g., `v0.4.8...v0.4.9`) from the provided GitHub URL.
   - URL format: `https://github.com/<OWNER>/<REPO>/compare/<COMPARE_STRING>`
   - The user may also provide just `OWNER/REPO` and a compare string separately.
2. Sanitize the `COMPARE_STRING` for use in filenames by replacing `...` with `_to_` (e.g., `v0.4.8_to_v0.4.9`).
3. Verify `gh` is installed and authenticated by running:
   ```
   gh auth status
   ```
   If this fails, stop and inform the user they need to install/authenticate `gh`.

// turbo
### Step 2 — Fetch the Comparison as JSON

Use `gh api` to fetch the comparison data as JSON and write it to a temp file. The JSON response contains structured data including: commit list, file list with `filename`, `status` (added/modified/removed/renamed), `additions`, `deletions`, `changes`, and `patch` (per-file diff).

**Command:**
```
gh api "/repos/<OWNER>/<REPO>/compare/<COMPARE_STRING>" --paginate > /tmp/diff_<SANITIZED_COMPARE_STRING>.json
```

**Cross-OS note:** On Windows PowerShell, use `$env:TEMP` instead of `/tmp/`:
```powershell
gh api "/repos/<OWNER>/<REPO>/compare/<COMPARE_STRING>" --paginate | Out-File "$env:TEMP\diff_<SANITIZED_COMPARE_STRING>.json" -Encoding utf8
```

**Error handling:** If the `gh api` command fails (non-zero exit code), stop and report the error to the user. Common causes:
- Repository not found (check OWNER/REPO)
- Invalid comparison string (check tag/branch names)
- Authentication issue (`gh auth login`)

### Step 3 — Read and Analyze the JSON

1. Use the `view_file` tool to read `/tmp/diff_<SANITIZED_COMPARE_STRING>.json` (or `$env:TEMP\diff_<SANITIZED_COMPARE_STRING>.json` on Windows).
2. **For large diffs (300+ files):** The JSON file may be very large. Read it in chunks using `StartLine` and `EndLine` parameters of `view_file` (800 lines at a time max). Process each chunk before moving to the next.
3. From the JSON, extract:
   - `total_commits`: number of commits in the comparison
   - `commits[]`: list of commits with `commit.message` and `author`
   - `files[]`: array of changed files, each containing:
     - `filename` — path of the changed file
     - `status` — one of: `added`, `modified`, `removed`, `renamed`
     - `additions` — lines added
     - `deletions` — lines deleted
     - `changes` — total lines changed
     - `patch` — the unified diff for this file (may be absent for binary files or very large files)
   - `diff_url`, `html_url` for reference links
4. Group files by component or directory for logical analysis.

### Step 4 — Generate the Summary Markdown

Use the `write_to_file` tool to create `PR_Review_Summary_<SANITIZED_COMPARE_STRING>.md` in the **current working directory**.

Use the following template structure, adapting sections based on what's relevant:

```markdown
# PR Review Summary: <COMPARE_STRING>

**Repository:** <OWNER>/<REPO>
**Comparison:** <COMPARE_STRING>
**Total Commits:** <N>
**Files Changed:** <N> | Additions: +<N> | Deletions: -<N>

---

## Executive Summary

<2-3 sentence overview of what this set of changes accomplishes>

## Key Changes

### <Category 1: e.g., New Features>
- **<filename>**: <description of change>

### <Category 2: e.g., Bug Fixes>
- **<filename>**: <description of change>

### <Category 3: e.g., Refactoring>
- **<filename>**: <description of change>

### <Category 4: e.g., Dependency Updates>
- **<filename>**: <description of change>

## Breaking Changes

<List any breaking changes, or "None identified.">

## File Change Summary

| File | Status | +Lines | -Lines |
|------|--------|--------|--------|
| `path/to/file.ts` | modified | +10 | -3 |
| ... | ... | ... | ... |
```

**Guidelines for categorization:**
- Group by functional area (features, fixes, refactoring, config, docs, tests, dependencies).
- For large diffs (50+ files), use the collapsible `<details>` tag for the full file table.
- Always highlight breaking changes and security-relevant changes prominently.
- Summarize the `patch` content for key files — don't just list filenames.

### Step 5 — Cleanup

Remove the temporary JSON file:
- **Linux/macOS:** `rm /tmp/diff_<SANITIZED_COMPARE_STRING>.json`
- **Windows PowerShell:** `Remove-Item "$env:TEMP\diff_<SANITIZED_COMPARE_STRING>.json"`

Use whichever command matches the user's current shell environment.
