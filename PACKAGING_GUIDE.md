# Packaging Guide: Turn Your AI Setup Into a Persona

Feed this document to your AI coding agent. It will read your existing configuration and generate a publishable persona package.

---

## Instructions for the AI Agent

You are about to package this user's existing AI configuration into a persona package for personalities.sh. Follow every step below. Do not skip steps. Do not ask unnecessary questions. When you need information, check the user's files first and only ask if the information genuinely cannot be inferred.

### Step 1: Discover the Current Setup

Scan for existing configuration files. Check all of these locations:

**Claude Code:**
- `~/.claude/CLAUDE.md` (global config)
- `./.claude/CLAUDE.md` (project config)
- `~/.claude/commands/*.md` (custom commands)
- `~/.claude/settings.json` (MCP server declarations)
- `~/.claude.json` (MCP server declarations, alternate location)

**Cursor:**
- `.cursorrules` (project rules)
- `~/.cursor/rules/*.md` (global rules)

**Windsurf:**
- `.windsurfrules` (project rules)

**Generic:**
- `.ai/` directory
- `AGENTS.md`
- `SYSTEM_PROMPT.md` or `SYSTEM_PROMPT.txt`
- Any `*.yaml` or `*.yml` files that look like goal tracking, task lists, or contact databases

Report what you found before proceeding. List each file, its path, and a one-line summary of what it contains.

### Step 2: Analyze the Configuration

Read every file you discovered. Identify:

1. **Identity elements**: Who is this AI supposed to be? What role, what primary directive?
2. **Communication rules**: Tone, formatting, vocabulary, things to avoid.
3. **Hard constraints**: NEVER/ALWAYS rules, boundaries, confidentiality.
4. **Context**: Company info, team members, org structure, industry.
5. **Operating modes**: Named behavioral modes (triage, drafting, research, etc.).
6. **Integrations**: MCP servers, APIs, external tools referenced.
7. **Commands**: Custom slash commands or named workflows.
8. **Persistent state**: Goals, tasks, contacts, logs, journals.
9. **Domain knowledge**: Industry-specific rules, frameworks, or reference material.
10. **Personal data**: Names, emails, company names, timezones -- anything that should become a `{{VARIABLE}}`.

### Step 3: Identify Personal Data for Replacement

Go through every piece of information from Step 2 and classify it:

- **Public/generic**: Keep as-is. Example: "Use markdown formatting" stays as-is.
- **Personal but structurally important**: Replace with a `{{VARIABLE}}`. Example: "You work for ADBC" becomes "You work for {{YOUR_COMPANY}}".
- **Private and not needed by others**: Remove entirely. Example: specific API keys, internal URLs, personal health notes.

Use these reserved variable names when they fit:

| Variable | Use for |
|----------|---------|
| `{{YOUR_NAME}}` | User's full name |
| `{{YOUR_FIRST_NAME}}` | User's first name |
| `{{YOUR_ROLE}}` | Job title |
| `{{YOUR_COMPANY}}` | Company name |
| `{{YOUR_EMAIL}}` | Primary email |
| `{{YOUR_TIMEZONE}}` | IANA timezone |
| `{{YOUR_CURRENCY}}` | ISO currency code |
| `{{YOUR_LANGUAGE}}` | Primary language |

For anything not covered by reserved names, create a custom variable with a descriptive ALL_CAPS name. Example: `{{HARD_STOP_TIME}}`, `{{TEAM_CHAT_PLATFORM}}`, `{{CRM_TOOL}}`.

### Step 4: Generate the Package

Create a new directory called `persona-package/` and generate these files:

#### 4a. persona.yaml

```yaml
name: [slug from the persona's role, e.g., "chief-of-staff", "legal-analyst", "code-reviewer"]
display_name: [human-readable name]
version: 1.0.0
description: [1-3 sentences. What it does, who it's for. No marketing language.]
author:
  name: [ask the user]
  github: [ask the user]
category: [pick ONE from: executive, professional-services, developer, creative, research, domain-specialist, personal, operations, sales, support]
tags: [2-8 relevant tags]
compatibility:
  - claude-code
  [add others if the config is platform-agnostic]
```

Add `requires_mcp` for every MCP server found in Step 1:
```yaml
requires_mcp:
  - name: [server name]
    required: [true if the persona breaks without it, false if it degrades gracefully]
    purpose: [one sentence]
```

Add `variables` for every `{{VARIABLE}}` you created in Step 3:
```yaml
variables:
  - key: YOUR_NAME
    prompt: "What's your full name?"
    required: true
  [etc.]
```

#### 4b. PERSONA.md

Restructure the user's configuration into these sections. Every persona must have the first three. Add the rest if the source material supports them.

**Required sections:**

`## Identity` -- Pull from any "who you are" / "your role" / "primary directive" content. Rewrite in second person ("You are..."). 3-10 sentences. Must state the role and the core directive clearly.

`## Communication Style` -- Pull from any tone/formatting/vocabulary rules. Convert to bullet points. Each bullet should be a concrete instruction, not a vague aspiration. Minimum 3 rules.

`## Behavioral Rules` -- Pull from any NEVER/ALWAYS constraints, boundaries, safety rules. These are hard limits that override everything else. Minimum 2 rules.

**Optional sections (include if source material exists):**

`## Context` -- Company info, team members, industry, key systems. Use `{{VARIABLES}}` for personal details.

`## Operating Modes` -- Named modes with distinct behavioral patterns. Each mode gets a subsection with activation trigger and specific rules.

`## Integrations` -- Table of MCP servers/tools with routing rules and fallback behavior. Include the line: "If a required integration is disconnected, note it and continue. Do not hallucinate results from missing integrations."

`## Goals` -- Goal tracking structure. Reference memory/goals.yaml if applicable.

`## Scheduling` -- Cron-style automation table if the persona runs on a schedule.

`## Self-Improvement` -- Rules for how the persona proposes changes to its own config.

**Formatting rules:**
- Use `##` for section headers. Not `#` (reserved for the file title) or `###` (for subsections within a section).
- Replace all personal data with `{{VARIABLES}}` per Step 3.
- Do not include setup instructions (those go in SETUP.md).
- Do not include the persona.yaml metadata (that's a separate file).
- Write in imperative/directive voice. "Do X" not "The persona should do X."

#### 4c. SETUP.md

Generate installation instructions:

```markdown
# Setup: [Display Name]

## Prerequisites
[List runtime requirements: Claude Code version, accounts needed, etc.]

## Required MCP Servers
[For each required MCP server, provide the exact install command and any auth steps]

## Optional MCP Servers
[Same format, for optional integrations]

## Environment Variables
[Table of any env vars needed, with purpose and where to set them]

## After Installation
[Numbered steps the user should take after install: edit config, add writing samples, set goals, test a command]
```

Write this for someone who can follow instructions but does not debug code. Every command must be copy-pasteable.

#### 4d. README.md

```markdown
# [Display Name]

[One paragraph: what this persona does and who should use it]

## What It Does
[5-10 bullet points of capabilities]

## Who It's For
[1-3 sentences describing the target user]

## Requirements
[Bullet list of MCP servers and accounts needed]

## Quick Start
\```bash
npx personas install [name]
\```

## Example
[Paste the best example interaction from examples/]

## Author
[Name, GitHub link, one-line bio]
```

#### 4e. commands/ (if applicable)

For each custom command found in Step 1, create a file in `commands/`:

```markdown
---
name: [command-name]
description: [one sentence]
---

# /[command-name] - [Title]

[When to use this command]

## Step 1: [Action]
[Numbered, concrete instructions]

## Step 2: [Action]
[...]

## Output Format
[How the result should be presented]
```

Replace personal data with variables. If a command references a specific MCP server, note the graceful degradation behavior.

#### 4f. memory/ (if applicable)

For each persistent state file (goals, tasks, contacts, logs), create a template version in `memory/`:

- Replace real data with realistic placeholder data.
- Keep the structure intact.
- Add YAML comments explaining each field.
- Name files descriptively: `goals.yaml`, `contacts.md`, `tasks.yaml`.

#### 4g. skills/ (if applicable)

For domain-specific knowledge files, create skill files in `skills/`:

```markdown
---
name: [skill-name]
description: [when this knowledge applies]
---

[Domain rules, frameworks, reference material]
```

#### 4h. examples/ (strongly recommended)

Create at least one example interaction file in `examples/`:

```markdown
# Example: [Scenario Name]

**User:** [input]

**Persona:** [response]
```

Use the most impressive or representative interaction pattern. Replace personal data with variables or realistic fictional data.

### Step 5: Security Self-Check

Before finalizing, scan every generated file for:

1. **Credentials**: API keys, tokens, passwords, connection strings. Remove all.
2. **Real personal data that slipped through**: Names, emails, phone numbers, addresses that should be variables. Replace.
3. **Hardcoded URLs**: Internal company URLs, private dashboards, local network addresses. Remove or variablize.
4. **Dangerous instructions**: Commands that delete files, send unsolicited messages, access other apps' credentials, or make network requests to fixed endpoints. Rewrite to require user confirmation.
5. **Absolute paths**: File paths like `/Users/mike/...` or `C:\Users\...`. Convert to relative paths or variables.

Report what you found and fixed.

### Step 6: Validate the Package

Check the generated package against these rules:

| Check | Pass? |
|-------|-------|
| `persona.yaml` exists and has all required fields | |
| `PERSONA.md` has ## Identity, ## Communication Style, ## Behavioral Rules | |
| `SETUP.md` exists and has install commands for every required MCP server | |
| `README.md` exists and has Quick Start section | |
| `name` in persona.yaml is lowercase, hyphens only, 3-40 chars | |
| `version` is semver (X.Y.Z) | |
| Every `{{VARIABLE}}` in any file is declared in persona.yaml `variables` | |
| No hardcoded credentials anywhere | |
| No absolute file paths anywhere | |
| Total package would be under 5 MB | |

Report results as a checklist. If anything fails, fix it before presenting to the user.

### Step 7: Present to the User

Show the user:
1. The complete directory structure with file sizes.
2. The full content of `persona.yaml`.
3. The full content of `PERSONA.md`.
4. A summary of what was included vs. what was stripped (personal data, credentials, etc.).
5. Any decisions you made that the user should review (category choice, which content goes in which section, what was marked required vs. optional).
6. The validation checklist results.

Ask the user to review before writing the files to disk.

---

## For the Human Reading This

You don't need to understand the steps above. Just do this:

1. Open your AI coding agent (Claude Code, Cursor, etc.).
2. Paste this entire document into the chat.
3. Say: **"Package my current AI setup as a persona. Follow these instructions."**
4. Review what it generates. Strip anything you don't want public.
5. Push the `persona-package/` folder to a GitHub repo.
6. Submit to personalities.sh.

That's it. Your AI does the work. You review the output.
