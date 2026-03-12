# Report Formats

This file controls which report skeleton the skill uses and explains how to swap, customize, or create your own.

## Active Format

The skill reads report skeletons from `assets/templates/report-skeletons/`. The **active format** is the one the skill uses by default when generating a finding.

**Current active format:** `default.md`

To change the active format, update the line above to point to a different skeleton file. For example, to use the CodeHawks format:

```
**Current active format:** `codehawks.md`
```

The skill will then read `assets/templates/report-skeletons/codehawks.md` and follow that structure for all generated findings.

## Available Skeletons

| File | Format | Best For |
|---|---|---|
| `default.md` | Description → Impact → PoC → Recommendations | Private audits, general-purpose reports, client deliverables |
| `codehawks.md` | Summary → Vulnerability Details → Impact → Tools Used → Recommendations | CodeHawks competitive audits, contests with tools-used requirements |

## How to Create a Custom Skeleton

You can create your own report format by adding a new `.md` file to `assets/templates/report-skeletons/`. Follow this structure:

### Step 1: Create the file

Create a new markdown file in `assets/templates/report-skeletons/`. Name it after the platform or style it targets (e.g., `sherlock.md`, `immunefi.md`, `my-team-format.md`).

### Step 2: Use this file structure

Every skeleton file should have three parts:

```markdown
# {Format Name} Report Skeleton

{1-2 sentence description of when to use this format.}

To activate this skeleton, set it as the active format in `FORMATS.md`.

## Skeleton

```markdown
{The actual markdown template with placeholder labels in curly braces.}
```

## Section Rules

| Section | Required | Notes |
|---|---|---|
| ... | ... | ... |
```

### Step 3: Define the skeleton

Inside the `## Skeleton` block, write the exact markdown structure you want every finding to follow. Use curly-brace placeholders for dynamic content:

- `{Title}` — the finding title
- `{Critical | High | Medium | Low}` — severity options
- `{Category}` — vulnerability type
- Descriptive placeholders like `{Brief summary of the issue}` for guidance

### Step 4: Define section rules

Add a table explaining each section: whether it's required, when to include it, and any formatting constraints. This helps the skill (and contributors) understand the intent behind each section.

### Step 5: Set it as active

Update the **Active Format** line at the top of this file to point to your new skeleton.

## Guidelines for Custom Skeletons

- Keep sections clearly labeled with `##` headers so the skill can parse them.
- Always include a Title, Severity, and Recommendations section at minimum.
- If your format needs a PoC, specify which severity levels require it.
- If your format uses fields like "Tools Used" or "Difficulty", include default values in the section rules.
- The title format (`{Actor} Can {Impact} {Affected Party}`) applies to all skeletons unless your skeleton explicitly overrides it with a different pattern.
- The severity matrix from `references/severity-guide.md` applies to all skeletons regardless of format.

## Platform-Specific Notes

Different audit platforms have specific formatting expectations. When creating a skeleton for a platform, consider:

- **Code4rena** — Findings typically use a QA Report format for Low/Non-Critical, and individual submissions for Medium/High. Labels like `[H-01]`, `[M-01]` are common.
- **Sherlock** — Requires a clear root cause, and findings are often grouped by root cause. Duplicates are judged strictly.
- **Cantina** — Similar to Code4rena but may have different section expectations per contest.
- **Immunefi** — Bug bounty format. Impact and proof-of-concept are heavily weighted. Requires clear reproduction steps.
- **CodeHawks** — Uses Summary + Vulnerability Details split. Tools Used is a standard field.

These are starting points — always check the current submission guidelines for each platform before submitting.
