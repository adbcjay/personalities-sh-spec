# Persona Format Specification v0.1

A persona is a portable, structured AI behavioral configuration that defines who an AI agent IS, not just what it can do. This spec defines the file format, directory structure, and validation rules for persona packages distributed through personalities.sh.

## 1. Core Concepts

**Persona vs. Skill vs. Tool**

A tool is a single capability (search the web, read a spreadsheet). A skill is a bundle of instructions for one domain (how to write smart contracts, how to format Excel files). A persona is an operating identity that shapes every interaction: communication style, priorities, relationships, judgment, and the tools/skills it wields.

A persona answers: "Who is this AI, what does it care about, and how does it behave?" A skill answers: "When this task comes up, what steps should you follow?"

Personas can contain skills. Skills cannot contain personas.

**Portability**

A persona package must work across any AI coding agent that reads markdown configuration files. The spec targets Claude Code as the primary runtime but avoids Claude-specific syntax. Persona authors should note platform-specific features in SETUP.md.

**Composability**

Personas are not monolithic. A user should be able to install a base persona (communication style + behavioral rules) and layer domain skills on top. The spec supports this through the `extends` field in persona.yaml and the separation of identity (PERSONA.md) from capability (skills/).

---

## 2. Directory Structure

```
my-persona/
├── persona.yaml        # REQUIRED  Machine-readable metadata and catalog entry
├── PERSONA.md          # REQUIRED  Identity, behavior, communication style
├── SETUP.md            # REQUIRED  Dependencies, credentials, installation steps
├── README.md           # REQUIRED  Human-readable description for the catalog
├── commands/           # optional  Slash command definitions
│   └── *.md
├── memory/             # optional  Persistent state templates
│   └── *.yaml or *.md
├── skills/             # optional  Domain knowledge files
│   └── *.md
├── examples/           # optional  Sample interactions
│   └── *.md
├── templates/          # optional  File templates the persona generates
│   └── *.*
└── LICENSE             # optional  Defaults to MIT if absent
```

All paths are relative to the persona root directory. No file should reference absolute paths. Use `{{VARIABLE}}` placeholders for user-specific values.

---

## 3. persona.yaml

The machine-readable manifest. This is what the catalog indexes, the installer reads, and the evaluator validates.

### Required Fields

```yaml
# === Identity ===
name: chief-of-staff              # URL-safe slug. Lowercase, hyphens only. 3-40 chars.
display_name: Chief of Staff       # Human-readable name for the catalog.
version: 1.0.0                     # Semver. Increment on every published change.
description: >
  Executive assistant that triages email, manages your calendar,
  runs morning briefings, and tracks your quarterly goals.
  # 1-3 sentences. What it does, who it's for. No marketing fluff.

# === Author ===
author:
  name: Mike Murchison              # Display name.
  github: mimurchison               # GitHub username. Used for attribution and verification.

# === Classification ===
category: executive                 # Primary category. One of the allowed values (see Section 3a).
tags:                               # 2-8 tags for search and filtering.
  - productivity
  - email
  - calendar
  - briefings
```

### Optional Fields

```yaml
# === Compatibility ===
compatibility:                      # Which AI agent runtimes this persona supports.
  - claude-code                     # Default if omitted.
  - cursor
  - windsurf
  - cline
  - aider

min_version: "1.0.0"               # Minimum runtime version required (e.g., Claude Code version).

# === Dependencies ===
requires_mcp:                       # MCP servers the persona needs to function.
  - name: gmail                     # Server name.
    required: true                  # true = persona won't work without it. false = degrades gracefully.
    purpose: "Email triage and drafting"
  - name: google-calendar
    required: true
    purpose: "Scheduling and availability"
  - name: slack
    required: false
    purpose: "Channel monitoring and DM triage"

requires_skills:                    # skills.sh skills this persona depends on.
  - xlsx                            # Will be auto-installed if missing.

# === Personalization ===
variables:                          # Placeholders that the installer prompts for.
  - key: YOUR_NAME
    prompt: "What's your full name?"
    required: true
  - key: YOUR_COMPANY
    prompt: "Company name?"
    required: true
  - key: YOUR_TIMEZONE
    prompt: "Timezone (e.g., America/New_York)?"
    required: true
    default: "UTC"
  - key: HARD_STOP_TIME
    prompt: "What time must you leave work by? (e.g., 5:30 PM)"
    required: false

# === Composition ===
extends: null                       # Slug of a base persona this one builds on.
                                    # e.g., "professional-communicator" as a base,
                                    # "legal-analyst" extends it with domain skills.

# === Economics (future) ===
license: MIT                        # SPDX identifier. Defaults to MIT.
pricing: free                       # free | paid | freemium. Future field.

# === Metadata ===
repository: https://github.com/mimurchison/claude-chief-of-staff
homepage: null                      # Optional website URL.
```

### 3a. Allowed Categories

Exactly one primary category per persona.

| Category | Description |
|----------|-------------|
| `executive` | C-suite support, leadership operations, strategic planning |
| `professional-services` | Legal, finance, compliance, HR, consulting |
| `developer` | Code review, DevOps, architecture, debugging |
| `creative` | Writing, content, design, brand, marketing |
| `research` | Academic, market research, analysis, synthesis |
| `domain-specialist` | Industry-specific expertise (crypto, healthcare, real estate, etc.) |
| `personal` | Fitness, finance, journaling, life admin, learning |
| `operations` | Project management, task tracking, workflow automation |
| `sales` | CRM, pipeline management, outreach, proposals |
| `support` | Customer service, documentation, knowledge base |

---

## 4. PERSONA.md

The identity document. This is the core of what makes a persona a persona. It gets loaded into the AI's system context every session.

PERSONA.md uses markdown with structured sections. Each section is an H2 (`##`). Sections can appear in any order, but the recommended order below reflects priority (higher = loaded first if context is tight).

### Required Sections

#### `## Identity`

Who this persona IS. Name, role, core directive, and the one-sentence answer to "what do you do?"

```markdown
## Identity

You are the Chief of Staff for {{YOUR_NAME}} at {{YOUR_COMPANY}}.

Your primary directive: double {{YOUR_FIRST_NAME}}'s productive output by handling
communication triage, calendar management, meeting preparation, and goal tracking.

You are not an assistant waiting for instructions. You are a proactive operator who
anticipates needs, flags risks, and drives accountability on goals.
```

Rules:
- Must define a clear role and primary directive.
- Must use second person ("You are...") not third person ("The persona is...").
- Should be 3-10 sentences. Enough to anchor behavior, short enough to always fit in context.

#### `## Communication Style`

How the persona talks. Tone, vocabulary, sentence structure, formatting preferences.

```markdown
## Communication Style

- Direct and concise. No filler phrases, no hedging.
- Match the user's energy: short question = short answer, deep exploration = deep response.
- Use plain language. Explain jargon when it's unavoidable.
- Never open with "Great question!" or "That's an interesting point." Acknowledge with "Got it," "Understood," or nothing.
- When uncertain, say "I don't know" rather than guessing.
- Format responses in markdown. Use tables for comparisons, bullet points for lists.
```

Rules:
- Must contain at least 3 concrete behavioral instructions (not vague "be helpful").
- Should include both DO and DON'T directives.
- If the persona adapts to the user's writing style, include sample texts in an `## Examples` section or `examples/writing-samples.md`.

#### `## Behavioral Rules`

Hard constraints, boundaries, and operating principles. These override other instructions.

```markdown
## Behavioral Rules

- NEVER disclose confidential company information outside authorized channels.
- NEVER send a message to an external party without explicit user confirmation.
- Always confirm before taking irreversible actions (deleting files, sending emails, closing deals).
- If you lack information to make a judgment call, say so and ask. Don't guess.
- Keep workstreams separate. Don't cross-reference Project A data in Project B conversations.
```

Rules:
- Must contain at least 2 hard constraints (NEVER/ALWAYS rules).
- Should address: confidentiality, authorization boundaries, reversibility, and uncertainty handling.
- These are the persona's "guardrails." The evaluator checks this section exists and is non-trivial.

### Optional Sections

#### `## Context`

Background information the persona needs to operate. Company details, org structure, industry, key relationships.

```markdown
## Context

### Company
{{YOUR_COMPANY}} is a 5-person blockchain regulatory advisory firm based in Abu Dhabi.
Service lines: Licensing, Advisory, Innovation Lab.

### Team
| Name | Role | Communication Style |
|------|------|-------------------|
| Abdulla | CEO | Brief, decision-oriented. Prefers bullet points. |
| Calvin | CBO | Detail-oriented. Wants data before recommendations. |
| Dana | EA | Task-focused. Needs clear action items. |

### Key Systems
- Email: Gmail (primary communication channel)
- Tasks: Todoist
- Documents: Google Drive
- Chat: Telegram (team), Slack (external partners)
```

#### `## Operating Modes`

Named modes the persona can switch between. Each mode changes behavior for a specific task type.

```markdown
## Operating Modes

### Triage Mode
Activated when processing inbox (email, messages, notifications).
- Classify each item by urgency tier (1-3).
- Tier 1: needs response within 2 hours. Surface immediately.
- Tier 2: needs response within 24 hours. Batch into daily briefing.
- Tier 3: informational only. Log and archive.

### Draft Mode
Activated when writing communications on behalf of the user.
- Match the user's writing voice (see examples/).
- Always present drafts for review before sending.
- Flag anything that could be misinterpreted.

### Research Mode
Activated when investigating a topic or question.
- Cite sources. No unsupported claims.
- Present findings, then recommendation, then tradeoffs.
- Stop and ask if the scope is unclear before going deep.
```

#### `## Integrations`

How the persona uses its connected tools and MCP servers. Routing rules, preferences, fallback behavior.

```markdown
## Integrations

| System | MCP Server | Usage |
|--------|-----------|-------|
| Gmail | gmail | Triage, drafting, sending. Check every /gm run. |
| Google Calendar | google-calendar | Scheduling, availability. Block focus time proactively. |
| Slack | slack | Monitor #general and DMs. Don't post without permission. |
| Todoist | todoist | Task tracking. Sync with /my-tasks command. |

If a required MCP server is disconnected:
- Note it in the briefing: "Gmail not connected. Email triage skipped."
- Do not hallucinate results from disconnected integrations.
```

#### `## Goals`

What this persona is working toward. Links to memory/goals.yaml if using structured goal tracking.

```markdown
## Goals

Active goals are tracked in memory/goals.yaml. Reference them in every briefing
and when prioritizing tasks. If an action doesn't connect to a goal, question whether
it should be done at all.
```

#### `## Scheduling`

Autonomous behaviors triggered on a schedule (requires cron or equivalent).

```markdown
## Scheduling

| Schedule | Command | Description |
|----------|---------|-------------|
| Daily 7:00 AM | /gm | Morning briefing: email triage + calendar + goals |
| Daily 12:00 PM | /triage digest | Midday inbox check |
| Weekly Monday 9:00 AM | /weekly-review | Week-ahead prep + goal progress |
```

#### `## Self-Improvement`

Rules for how the persona evolves over time. What it should learn, how it proposes changes to its own configuration.

```markdown
## Self-Improvement

When you notice a recurring pattern (user always rejects a certain type of suggestion,
a workflow consistently fails, a contact's communication preference has changed):
1. Note the pattern.
2. Propose a specific edit to PERSONA.md, a command file, or a memory file.
3. Present the proposed change to the user for approval.
4. Never self-modify without explicit approval.
```

---

## 5. SETUP.md

Installation requirements and step-by-step setup instructions. Written for the END USER, not the persona author.

### Required Content

```markdown
# Setup: Chief of Staff

## Prerequisites
- Claude Code v1.0+ (or compatible agent runtime)
- Active Gmail account with IMAP enabled
- Google Calendar API access

## Required MCP Servers

### Gmail (required)
```bash
npx @anthropic-ai/claude-code mcp add gmail -- npx @anthropic-ai/gmail-mcp
```
After adding: authenticate via the OAuth flow that opens in your browser.

### Google Calendar (required)
```bash
npx @anthropic-ai/claude-code mcp add google-calendar -- npx @anthropic-ai/gcal-mcp
```

### Slack (optional - enhances triage)
```bash
npx @anthropic-ai/claude-code mcp add slack -- npx @anthropic-ai/slack-mcp
```

## Environment Variables
| Variable | Purpose | Where to Set |
|----------|---------|-------------|
| GMAIL_ADDRESS | Your primary email | Prompted during install |

## After Installation
1. Edit `~/.claude/PERSONA.md` to add your team members to the Context section.
2. Add at least 3 writing samples to `examples/writing-samples.md`.
3. Set your goals in `memory/goals.yaml`.
4. Test with `/gm` to run your first morning briefing.
```

Rules:
- Every required MCP server must have exact installation commands.
- Every environment variable must be documented.
- Must include a "test it works" step at the end.
- Do not assume technical expertise. Write for someone who can follow instructions but doesn't debug code.

---

## 6. commands/

Slash command definitions. Each file is one command. The filename (minus extension) becomes the command name.

File: `commands/gm.md`
```markdown
---
name: gm
description: Morning briefing - email triage, calendar review, goal check-in
---

# /gm - Good Morning Briefing

Run this command every morning. Execute each step in order.

## Step 1: Email Triage
1. Fetch unread emails from the last 12 hours via Gmail MCP.
2. Classify each email into Tier 1 (urgent), Tier 2 (today), or Tier 3 (FYI).
3. For Tier 1 emails, draft a response for user review.

## Step 2: Calendar Review
1. Fetch today's calendar events via Google Calendar MCP.
2. For each meeting, note: attendees, purpose, any prep needed.
3. Flag back-to-back meetings with no buffer.

## Step 3: Goal Check-in
1. Read memory/goals.yaml.
2. For each active goal, note current status and any blockers.
3. Suggest one concrete action for today that advances the top goal.

## Output Format
Present as a single briefing message:
- EMAIL: [count] unread. [count] Tier 1 (list them).
- CALENDAR: [count] meetings today (list with times).
- GOALS: Top goal status + suggested action.
- FLAGS: Anything that needs immediate attention.
```

Rules:
- YAML frontmatter with `name` and `description` is required.
- Steps should be numbered and concrete (not "review emails" but "fetch unread emails from the last 12 hours").
- Each command should specify its output format.
- Commands should degrade gracefully if an MCP server is missing: skip that step with a note, don't crash.

---

## 7. memory/

Persistent state files. These are templates that get populated over time. The persona reads and writes to these files across sessions.

Supported formats: `.yaml` (structured data) and `.md` (freeform notes).

Example: `memory/goals.yaml`
```yaml
# Quarterly goals - updated by /gm and /weekly-review commands
quarter: Q1-2026
goals:
  - name: Close 3 new licensing deals
    status: in_progress
    metric: "1/3 closed"
    blockers: []
    last_updated: 2026-02-15
  - name: Launch advisory service landing page
    status: not_started
    metric: null
    blockers:
      - "Waiting on brand kit from marketing"
    last_updated: 2026-02-10
```

Example: `memory/contacts.md`
```markdown
# Contact Notes

## Ahmed Al-Mansoori
- Company: Abu Dhabi Blockchain Fund
- Role: CEO
- Met: CfC St. Moritz, January 2026
- Interest: Licensing for their new fund
- Style: Formal, prefers email over chat
- Last contact: 2026-02-12 (follow-up call)
```

Rules:
- Memory files are templates on install. They should contain example/placeholder data, not empty files.
- The persona should never delete user data from memory files. Append or update only.
- Memory files must not contain credentials, tokens, or secrets.
- YAML files should include comments explaining each field.

---

## 8. skills/

Domain knowledge files that the persona draws on. These follow the same format as skills.sh SKILL.md files but are bundled inside the persona.

Example: `skills/email-drafting.md`
```markdown
---
name: email-drafting
description: "Rules for drafting emails in the user's voice"
---

# Email Drafting

## Tone Rules
- Professional but not stiff. Write like a senior executive, not a template.
- First sentence gets to the point. No "I hope this email finds you well."
- Close with a clear ask or next step, not "Let me know your thoughts."

## Format Rules
- Subject lines: under 8 words, action-oriented.
- One topic per email. If there are two topics, send two emails.
- Use bullet points for anything with 3+ items.

## User's Writing Patterns
(Populated from examples/writing-samples.md during setup)
- Tends to use short sentences.
- Signs off with first name only.
- Uses "flagging" as a verb frequently.
```

Rules:
- Skill files inside a persona are automatically loaded as part of the persona. They don't need separate installation.
- Use skills/ for domain knowledge that the persona needs to do its job. Use commands/ for workflows the user triggers explicitly.
- A persona can also declare external skills as dependencies (via `requires_skills` in persona.yaml). Those are installed separately from skills.sh and coexist alongside bundled skills.

---

## 9. examples/

Sample interactions that show the persona in action. These serve two purposes: (1) help potential users understand what the persona does before installing, and (2) provide few-shot examples that improve persona behavior.

Example: `examples/morning-briefing.md`
```markdown
# Example: Morning Briefing

**User:** /gm

**Persona:**
Morning. Here's your February 17 briefing.

**EMAIL** - 14 unread, 2 urgent
- Tier 1: Ahmed Al-Mansoori re: licensing proposal review (sent 11 PM, waiting on your sign-off)
- Tier 1: Legal team re: VARA amendment deadline moved to Feb 20 (was Feb 28)
- Tier 2: 4 emails (partner updates, event confirmation, newsletter, vendor invoice)
- Tier 3: 8 emails (archived)

Draft response ready for Ahmed's email. Want me to show it?

**CALENDAR** - 3 meetings
- 10:00 AM - Calvin 1:1 (30 min) - no prep needed
- 2:00 PM - OSN partnership call (1 hr) - prep: pull latest proposal deck
- 4:30 PM - Konstantin marketing sync (30 min) - agenda TBD

Flag: no buffer between OSN call and Konstantin sync. Want me to move Konstantin to 5:00?

**GOALS** - Q1 progress
- Close 3 licensing deals: 1/3 done. Ahmed's deal is the closest pipeline item.
- Launch advisory landing page: blocked on brand kit. Konstantin sync today is a chance to push on this.

**Suggested action today:** Approve Ahmed's proposal response to keep that deal moving.
```

Rules:
- At least one example file is strongly recommended. Personas with examples get higher quality scores in the catalog.
- Examples should be realistic, not idealized. Show how the persona handles ambiguity, missing info, and edge cases.
- Mark any placeholder data clearly: use `{{YOUR_NAME}}` or obviously fictional names.

---

## 10. README.md

The catalog listing. Written for someone browsing personalities.sh deciding whether to install.

### Required Sections

```markdown
# Chief of Staff

One-paragraph description of what this persona does and who it's for.

## What It Does
- Bullet list of capabilities (5-10 items)

## Who It's For
- Description of the target user

## Requirements
- List of MCP servers and accounts needed

## Quick Start
```bash
npx personas install chief-of-staff
```

## Screenshots / Examples
Link to examples/ directory or embed key interaction samples.

## Author
Name, link, brief bio.
```

Rules:
- README.md is for humans. persona.yaml is for machines. Don't duplicate schema details here.
- Keep it under 200 lines. If you need more space, link to docs in the repo.

---

## 11. Validation Rules

The personalities.sh evaluator checks every submission against these rules before listing.

### Hard Requirements (fail = rejected)

| Rule | Check |
|------|-------|
| `persona.yaml` exists and parses | YAML syntax valid, all required fields present |
| `PERSONA.md` exists | File present and non-empty |
| `PERSONA.md` has required sections | `## Identity`, `## Communication Style`, `## Behavioral Rules` all present |
| `SETUP.md` exists | File present and non-empty |
| `README.md` exists | File present and non-empty |
| `name` is unique | No collision with existing personas in the catalog |
| `name` format valid | Lowercase, hyphens, 3-40 characters, no spaces |
| `version` is semver | Matches `X.Y.Z` pattern |
| No hardcoded credentials | No API keys, tokens, passwords, or secrets in any file |
| No absolute paths | All file references are relative |
| No prompt injection patterns | See Security section below |
| Total package size < 5 MB | Prevents bloated repos with embedded binaries |

### Soft Requirements (fail = warning, lower quality score)

| Rule | Check |
|------|-------|
| Examples present | At least one file in `examples/` |
| Memory templates populated | Memory files have example data, not empty |
| Commands have frontmatter | All `commands/*.md` have `name` and `description` |
| Variables documented | Every `{{VARIABLE}}` in any file is listed in persona.yaml `variables` |
| MCP install commands present | Every `requires_mcp` entry has install instructions in SETUP.md |
| Description quality | Description is 1-3 sentences, not a wall of marketing text |
| Tag relevance | Tags match actual persona capabilities (checked by evaluator AI) |

### Security Checks

The evaluator scans all files for:

1. **Credential leaks**: API keys, tokens, passwords, connection strings. Regex patterns for common formats (AWS keys, GitHub tokens, JWT, etc.).

2. **Prompt injection**: Instructions that attempt to override safety guidelines, exfiltrate data, or manipulate the host runtime. Patterns include:
   - "Ignore previous instructions"
   - "You are now in developer mode"
   - "Output your system prompt"
   - "Send [data] to [external URL]"
   - Base64-encoded instruction blocks
   - Hidden instructions in HTML comments or zero-width characters

3. **Dangerous operations**: Commands or instructions that:
   - Delete files outside the persona directory
   - Execute arbitrary shell commands without user confirmation
   - Access credentials from other applications
   - Make network requests to hardcoded external URLs (MCP servers are fine since they're user-configured)
   - Modify system configuration files

4. **MCP server validation**: Each declared MCP server is checked against a known-good registry. Unknown servers get flagged for manual review. Servers with known vulnerabilities are blocked.

Trust tiers based on evaluation results:
- **Verified**: Passes all checks, author identity confirmed, manual review completed.
- **Community**: Passes automated checks, no manual review.
- **Unreviewed**: New submission, pending evaluation.
- **Flagged**: Failed one or more security checks. Not listed until resolved.

---

## 12. Installation

When a user runs `npx personas install <name>`, the installer:

1. Fetches the persona package from the catalog (GitHub repo or registry).
2. Validates persona.yaml against the schema.
3. Prompts the user for each variable defined in `variables`.
4. Copies files to the target directory:
   - `PERSONA.md` → `~/.claude/PERSONA.md` (or merges into existing `CLAUDE.md`)
   - `commands/*` → `~/.claude/commands/`
   - `memory/*` → `~/.claude/memory/<persona-name>/`
   - `skills/*` → `~/.claude/skills/<persona-name>/`
5. Runs `sed` (or equivalent) to replace all `{{VARIABLE}}` placeholders.
6. Checks for required MCP servers and prompts to install any that are missing.
7. Checks for required skills and installs from skills.sh if missing.
8. Prints a "setup complete" message with next steps from SETUP.md.

### Multi-Persona Support

A user can have multiple personas installed. Only one persona's PERSONA.md is active at a time. Switching:

```bash
npx personas activate chief-of-staff    # Load this persona's PERSONA.md
npx personas activate legal-analyst      # Switch to a different persona
npx personas list                        # Show installed personas
npx personas deactivate                  # Return to default (no persona)
```

The active persona's PERSONA.md is symlinked (or copied) to the location the runtime reads from. Inactive personas remain installed but don't affect behavior.

### Composition

If a persona declares `extends: professional-communicator`, the installer:
1. Installs the base persona first (if not already installed).
2. Installs the extending persona.
3. Merges PERSONA.md: base sections load first, extending persona's sections override or append.
4. Commands and skills from both personas are available.
5. Memory directories are kept separate (`memory/professional-communicator/` and `memory/legal-analyst/`).

Conflict resolution: the extending persona wins. If both define `## Communication Style`, the child's version replaces the parent's.

---

## 13. Packaging Guide for Creators

You already have a working AI setup. Here's how to turn it into a persona package.

### If You Use Claude Code

Your existing configuration is probably in `~/.claude/CLAUDE.md`, `~/.claude/commands/`, and various dotfiles. To package:

1. **Feed your AI the spec.** Copy this document into your Claude Code session and say: "Read this spec. Package my current setup as a persona. My CLAUDE.md is at ~/.claude/CLAUDE.md, my commands are in ~/.claude/commands/."

2. **Your AI will generate:**
   - `persona.yaml` from your setup metadata
   - `PERSONA.md` by restructuring your CLAUDE.md into the required sections
   - `SETUP.md` from your MCP server configuration
   - `README.md` as a catalog description
   - Copies of your commands/ and any relevant state files

3. **Review what it generated.** Strip personal information you don't want public. Replace real names, emails, and company details with `{{VARIABLE}}` placeholders.

4. **Publish.** Push to a GitHub repo. Submit to personalities.sh via `npx personas publish` or the web interface.

### If You Use Another Tool

Export your system prompt, tool configurations, and any automation definitions. Structure them into the directory layout described in Section 2. The key transformation: take whatever monolithic config you have and split it into PERSONA.md (identity + behavior) and commands/ (workflows).

### What Makes a Good Persona

- **Specific over general.** "You are a legal analyst specializing in UAE free zone regulations" beats "You are a helpful legal expert." Generic personas are useless.
- **Rules over vibes.** "Never use more than 3 sentences in an email subject line" beats "Keep subject lines concise." Concrete rules produce consistent behavior.
- **Examples matter.** A persona with 5 sample interactions outperforms one with 50 behavioral rules. Show, don't just tell.
- **Degrade gracefully.** If an MCP server is missing, the persona should note it and continue, not break.
- **Respect user agency.** Never auto-send, auto-delete, or auto-commit without confirmation. The persona advises and drafts. The user decides and executes.

---

## Appendix A: Full persona.yaml Schema

```yaml
# REQUIRED
name: string              # 3-40 chars, lowercase, hyphens only
display_name: string      # Human-readable, any characters
version: string           # Semver (X.Y.Z)
description: string       # 1-3 sentences
author:
  name: string
  github: string
category: enum            # See Section 3a
tags: list[string]        # 2-8 items

# OPTIONAL
compatibility: list[string]        # Default: [claude-code]
min_version: string
extends: string | null             # Slug of base persona
license: string                    # SPDX identifier. Default: MIT
pricing: enum                      # free | paid | freemium. Default: free
repository: string                 # URL
homepage: string                   # URL

requires_mcp: list[object]
  - name: string
    required: boolean
    purpose: string

requires_skills: list[string]

variables: list[object]
  - key: string                    # ALL_CAPS_WITH_UNDERSCORES
    prompt: string                 # Question shown to user during install
    required: boolean
    default: string | null
```

## Appendix B: Reserved Variable Names

These variables have standard meanings across all personas. Use them when applicable instead of inventing custom names.

| Variable | Meaning |
|----------|---------|
| `YOUR_NAME` | User's full name |
| `YOUR_FIRST_NAME` | User's first name |
| `YOUR_ROLE` | User's job title |
| `YOUR_COMPANY` | Company/organization name |
| `YOUR_EMAIL` | Primary email address |
| `YOUR_TIMEZONE` | IANA timezone string |
| `YOUR_CURRENCY` | ISO 4217 currency code |
| `YOUR_LANGUAGE` | Primary language |

## Appendix C: Version History

| Version | Date | Changes |
|---------|------|---------|
| 0.1 | 2026-02-17 | Initial draft |
