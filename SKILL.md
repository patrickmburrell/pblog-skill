---
name: pblog
description: Use this skill when the user wants to log their daily work, accomplishments, or commits to their Obsidian vault using pblog. Triggers when user mentions logging work, capturing accomplishments, or explicitly references pblog/PB Logger. Also use when user describes work they've done and wants to document it (e.g., "I fixed the bug in X", "just finished feature Y", "log this", "capture what I did"). Also triggers when user mentions cleaning up, deduplicating, or tidying log entries (e.g., "clean up today's entries", "tidy my log", "deduplicate entries"), generating summaries (e.g., "summarize this week", "weekly summary", "summarize Q1"), checking goal status or progress (e.g., "how are my goals?", "goal status", "what needs attention?", "am I behind?"), viewing goal pulse (e.g., "show pulse", "what's overdue?", "goals needing attention"), backfilling commit history, or enriching log entries with AI summaries. Make sure to use this skill whenever the user talks about documenting their work, logging accomplishments, goal tracking, or mentions pblog, even if they don't explicitly say "use the pblog skill".
---

# pblog - Work Logging Skill

This skill helps log daily work to the Obsidian vault using the pblog CLI tool.

## How It Works

**In git repositories:** pblog automatically detects the project from a `.pblog` file in the repo. If no `.pblog` exists, it shows an error with clear instructions.

**Outside git repos:** pblog uses interactive mode to prompt for project selection.

## Command Usage

### Basic log entry
```bash
pblog log "work description"
```

The CLI will:
- Auto-detect project from `.pblog` if in a git repo
- Show error if no `.pblog` found in git repo
- Prompt for project if outside git repo

### With sub-bullets (nested details)

Add indented sub-bullets to an entry using `--details`. Supports **multi-level nesting** using 4-space indentation (0, 4, 8, 12... spaces).

**Simple flat list:**
```bash
pblog log "summary line" --details "item 1
item 2
item 3"
```

**Two-level nesting:**
```bash
pblog log "summary line" --details "$(cat <<'EOF'
top-level item 1
    nested under item 1
    also nested under item 1
top-level item 2
EOF
)"
```

**Multi-level hierarchical structure (3-4 levels deep):**
```bash
pblog log "designed phased refactor plan" --details "$(cat <<'EOF'
Phase 1: migrate PR caching to structured table
    key improvements
        enables efficient per-PR updates/deletes
        fixes stale cache after merge operations
    implementation notes
        rename dependabot_prs table to pull_requests
        add proper indexes on foreign keys
Phase 2: add indexes on remaining tables
    improves invalidation performance
    reduces full table scans
Phase 3: fix cascade invalidation (deferred)
EOF
)"
```

**Indentation rules:**
- Use **0, 4, 8, 12, 16...** spaces (multiples of 4)
- Tabs are automatically converted to 4 spaces
- Leading bullet markers (`-`, `*`) are automatically stripped
- Invalid indentation (2, 3, 6 spaces, etc.) shows a clear error with line number

This creates nested bullet structures in your log:
```markdown
- designed phased refactor plan
    - Phase 1: migrate PR caching to structured table
        - key improvements
            - enables efficient per-PR updates/deletes
            - fixes stale cache after merge operations
        - implementation notes
            - rename dependabot_prs table to pull_requests
            - add proper indexes on foreign keys
    - Phase 2: add indexes on remaining tables
        - improves invalidation performance
        - reduces full table scans
    - Phase 3: fix cascade invalidation (deferred)
```

### With local git commits
```bash
pblog log "work description" --local
```

### With GitHub API commits + PRs (cross-repo)
```bash
pblog log "work description" --remote
```

### Cross-repo with pattern matching
```bash
pblog log "closed dependabot PRs" --match "dependabot"
```
Note: `--match` automatically implies `--remote`

### Time-windowed activity
```bash
pblog log "PR reviews this afternoon" --remote --since 2h
pblog log "commits from last hour" --local --since 1h
```

### Retroactive entry (different date)
```bash
pblog log "work description" --date 2026-02-28
```

### Tidy / clean up entries
```bash
pblog tidy --yes                    # tidy today, current project
pblog tidy --date 2026-03-18 --yes  # specific date
pblog tidy --all-projects --yes     # all projects for today
```

**IMPORTANT: Always use `--yes` when running from Claude Code.** The interactive confirmation prompt requires stdin which is not available in non-interactive shells. Without `--yes`, the command will fail with exit code 1.

Sends note bullets to Claude via Bedrock for semantic deduplication, consolidation, and formatting cleanup. Shows a before/after preview then writes changes. Commits block is passed as read-only context (not modified).

**When to suggest tidy:**
- User has logged multiple times to the same project/date
- User mentions duplicate entries or messy formatting
- User asks to clean up or consolidate their log
- After a busy day with many small log entries

### Summaries

Generate summaries at different time scales. Each level rolls up from the one below it.

```bash
pblog summary week [YYYY-Wnn]      # weekly (default: current week)
pblog summary month [YYYY-MM]      # monthly from weekly summaries
pblog summary quarter [YYYY-Qn]    # quarterly from monthly summaries
pblog summary year [YYYY]          # yearly from quarterly summaries
```

All accept `--force`/`-f` to overwrite without confirmation. Argument is optional — defaults to the current period.

**Translating user requests:** When the user says "summarize week 14", convert to ISO format: `pblog summary week 2026-W14`. Do NOT pass the week number as a flag — `week` is a subcommand, not an option.

Examples:
```bash
pblog summary week                 # current week
pblog summary week 2026-W14       # specific week
pblog summary month 2026-03       # March 2026
pblog summary quarter 2026-Q1     # Q1 2026
pblog summary year 2026           # full year
```

### Backfill

Retroactively populate project logs with GitHub commit history across a date range.

```bash
pblog backfill --since 2026-03-01                    # backfill from date to today
pblog backfill --since 2026-03-01 --until 2026-03-15 # specific range
pblog backfill --since 2026-03-01 --summarize        # also generate AI summaries
pblog backfill --since 2026-03-01 --dry-run           # preview without writing
pblog backfill --since 2026-03-01 --skip-unmapped     # skip repos without project mapping
```

Uses GitHub API (cross-repo). Maps commits to projects via frontmatter. `--since` is required.

### Enrich

Add AI-generated summaries to log entries that only have commits and no descriptive text.

```bash
pblog enrich                           # enrich all commit-only entries
pblog enrich --project "PB Logger"     # single project only
pblog enrich --since 2026-03-01        # only entries after date
pblog enrich --dry-run                  # preview without API calls
```

### Status

Show goal coverage table for the current quarter. Displays all goals with linked project activity, pulse indicators (Active/Attention/Overdue/Needs setup), entry counts, and last activity dates. Local-only, no API calls.

```bash
pblog status                   # current quarter
pblog status --quarter 1       # specific quarter (1-4)
```

Includes quarter targets/milestones below the table. Goals are tracked via explicit `goals` frontmatter in project files (many-to-many linking).

### Pulse

Focused attention view showing only goals that need action. Filters to flagged goals (ATTENTION, OVERDUE, NO PROJECTS) and groups by goal type. Local-only, no API calls.

```bash
pblog pulse                    # current quarter
pblog pulse --quarter 1        # specific quarter (1-4)
```

Flag meanings:
- **ATTENTION** (yellow) — no activity for 2+ weeks (configurable via `pulse_yellow_weeks`)
- **OVERDUE** (red) — no activity for 4+ weeks (configurable via `pulse_red_weeks`)
- **NO PROJECTS** — goal has no linked projects (needs `goals` frontmatter in a project file)

Shows linked projects, weeks inactive, last activity date, and the quarter's target/milestone for each flagged goal.

**When to suggest pulse vs status:**
- User asks "what needs attention?" or "am I behind on anything?" → `pblog pulse`
- User asks "how are my goals?" or "show goal progress" → `pblog status`
- User asks about a specific goal → `pblog status` (shows all goals in a table)

### Init

Initialize pblog configuration (`~/.config/pblog/config.toml`).

```bash
pblog init
```

## When .pblog is Missing

If the user is in a git repo without a `.pblog` file, pblog will error:

```
Error: No .pblog file found in git repository

Create one with:
  echo 'project = "Project Name"' > .pblog

Available projects:
  - Project A
  - Project B
  [...]
```

**What to do:** Help the user create a `.pblog` file with the correct project name:

```bash
echo 'project = "PB Logger"' > .pblog
```

Then retry the pblog command.

## Work Description Guidelines

### Entry structure

Use a concise summary on the top-level bullet, with supporting details as indented sub-bullets.

Use the `--details` parameter to add sub-bullets:
```bash
pblog log "summary line" --details "item 1
item 2
item 3"
```

Good:
```markdown
- evaluated awesome-claude-skills repo (https://github.com/karanb192/awesome-claude-skills)
    - no new useful skills found; already running all verified content
    - only gap: Office doc skills (docx/xlsx/pptx), declined as not relevant
```

Bad (everything crammed into one line):
```markdown
- evaluated awesome-claude-skills repo — mostly a catalog of obra/superpowers + anthropics/skills with wishlist placeholders. No new useful skills found; already running all verified/real content. Only gap: Office doc skills, declined as not relevant
```

### Note text style

The note text should:
- Be concise but specific (1-3 sentences)
- Use lowercase first letter unless proper noun/acronym
- Focus on what was accomplished, not how
- Avoid trailing periods
- Be written in past tense

Good examples:
- "implemented user authentication with JWT tokens"
- "fixed race condition in payment processor"
- "reviewed and merged 3 dependabot PRs"
- "added .pblog file auto-detection feature"

Avoid:
- "I worked on the authentication system" (too vague)
- "Added code to handle tokens." (trailing period, uppercase)
- "Did some work" (not specific)
- "Working on feature X" (present tense, sounds incomplete)

## Commit Source Integration

### When to use --local vs --remote

**Use `--local`** (fetches from local git log):
- User mentions commits we just made during this conversation
- Work clearly involved coding and we're in the git repo
- User wants commits from all branches (GitHub API only sees default branch)
- User explicitly requests "local commits" or "git log"
- Default choice for normal daily logging with commits

**Use `--remote`** (fetches from GitHub API):
- User mentions PRs, code reviews, cross-repo activity
- Work involved reviewing others' code
- Need to capture activity from multiple repositories
- User explicitly requests "GitHub activity" or "remote commits"

**Use `--match "pattern"`** for cross-repository work:
- Automatically implies `--remote` (local git can't search across repos)
- Dependabot PRs across multiple repos
- CI/workflow configuration changes
- Any work spanning many repos not in the project's repo list
- Keywords: "dependabot", "workflow", "CI", "all repos", "across repos"

**Use `--since`** for time windows (works with both --local and --remote):
- "this morning" → `--since 3h`
- "this afternoon" → `--since 2h`
- "last hour" → `--since 1h`
- Requires either `--local` or `--remote` to be specified

### No flags (note text only)
If neither `--local` nor `--remote` is specified, only the note text is logged (no commits).
This is useful for logging work that doesn't involve commits (meetings, planning, etc.)

## Handling User Input

When the user describes their work informally, translate it to proper format:

**User says:** "I just finished adding that feature we talked about"
**You ask:** "What feature specifically?" then log with clear description

**User says:** "log this"
**You do:** Infer work from recent conversation context and log it

**User says:** "I fixed the bug"
**You do:** Ask which bug or use conversation context to identify it

## Error Handling

### Error: "No .pblog file found in git repository"

**Cause:** User is in a git repo without a `.pblog` file.

**Solution:**
1. Ask which Obsidian project this repo belongs to
2. Create `.pblog` file: `echo 'project = "Project Name"' > .pblog`
3. Retry the pblog command

### Error: "No project found matching: X"

**Cause:** Project name in `.pblog` doesn't exist in vault.

**Solution:**
1. Show the user the error (pblog provides suggestions)
2. Either fix the `.pblog` file or ask user which project is correct
3. Update `.pblog` with correct name

## Output and Confirmation

After successfully running `pblog log`, show:
1. What was logged (project name, date, note text)
2. How many commits were included (if any)

Example confirmation:
```
✓ Logged to PB Logger (2026-03-02):
  "implemented .pblog file auto-detection"
```

If commits were included:
```
✓ Logged to PB Logger (2026-03-02):
  "implemented .pblog file auto-detection"
  Added 4 commits
```

## Examples

### Example 1: Simple work log in git repo

User: "log today's work - I added the .pblog file feature"

Context: We're in a git repo with a `.pblog` file

```bash
pblog log "implemented .pblog file auto-detection" --local
```

### Example 2: Missing .pblog file

User: "log what we just did"

Context: We're in a git repo but no `.pblog` exists

**Step 1:** pblog will error:
```bash
pblog log "created pblog skill"
# Error: No .pblog file found in git repository
```

**Step 2:** Ask user which project:
"Which Obsidian project should I associate with this repo? (I see options like 'PB Logger', 'GitHub Tools', etc.)"

**Step 3:** Create `.pblog`:
```bash
echo 'project = "PB Logger"' > .pblog
```

**Step 4:** Retry:
```bash
pblog log "created pblog skill for Claude Code" --local
```

### Example 3: Cross-repo work

User: "I spent the morning closing dependabot PRs across all our repos"

```bash
pblog log "closed dependabot PRs across all repos" --match "dependabot"
```

### Example 4: Retroactive entry

User: "oh I forgot to log yesterday's work. I added the new charts to the dashboard"

```bash
pblog log "added revenue and engagement charts" --date 2026-03-01
```

### Example 5: Multi-level nested sub-bullets

User: "log today's architecture analysis work"

```bash
pblog log "analyzed cache architecture and designed refactor plan" --details "$(cat <<'EOF'
root cause analysis
    PR caching uses two conflicting approaches
        dependabot_prs table exists but never populated (dead code)
        actual implementation stores 21KB JSON blobs in generic cache_metadata table
    when PRs merge, only clear_dependabot_prs() called, leaving JSON stale
designed phased refactor plan
    Phase 1: migrate to structured pull_requests table
        enables efficient per-PR updates/deletes
        fixes stale cache after merge operations
    Phase 2: add indexes on foreign key columns
        improves invalidation performance
    Phase 3: fix cascade invalidation (deferred)
decision: implement Phases 1+2 together (low risk, high value)
EOF
)"
```

This creates:
```markdown
## 2026-04-20

- analyzed cache architecture and designed refactor plan
    - root cause analysis
        - PR caching uses two conflicting approaches
            - dependabot_prs table exists but never populated (dead code)
            - actual implementation stores 21KB JSON blobs in generic cache_metadata table
        - when PRs merge, only clear_dependabot_prs() called, leaving JSON stale
    - designed phased refactor plan
        - Phase 1: migrate to structured pull_requests table
            - enables efficient per-PR updates/deletes
            - fixes stale cache after merge operations
        - Phase 2: add indexes on foreign key columns
            - improves invalidation performance
        - Phase 3: fix cascade invalidation (deferred)
    - decision: implement Phases 1+2 together (low risk, high value)
```

## Technical Notes

- pblog stores logs in Obsidian markdown files at the project level
- Commits are merged into a single `- commits:` block per day
- GitHub integration uses `gh` CLI under the hood
- Time windows (--since) use formats like: 30m, 1h, 2h, 1h30m
- Dates use ISO format: YYYY-MM-DD
- `.pblog` format: `project = "Project Name"` (TOML)
- `--vault` flag goes BEFORE the subcommand: `pblog --vault pmb summary week`
- Bedrock commands (`enrich`, `tidy`, `summary`) may need `uv run pblog` to inherit AWS credentials
