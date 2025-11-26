# command-review-prs

A Claude Code slash command for reviewing PRs with AI-generated suggestions.

## Installation

```bash
# Clone to your preferred location
git clone git@github.com:claude-commands/command-review-prs.git <clone-path>/command-review-prs

# Symlink (use full path to cloned repo)
ln -s <clone-path>/command-review-prs/review-prs.md ~/.claude/commands/review-prs.md
```

## Usage

```text
/review-prs              # All PRs needing your review
/review-prs --team       # PRs for your team
/review-prs --stale      # PRs waiting >3 days
/review-prs 123          # Deep review of specific PR
```

## What it does

1. Fetches PRs where you're requested reviewer
2. Categorizes by priority (critical, high, normal)
3. Generates summaries for each PR
4. Identifies potential issues
5. Provides AI-generated review suggestions
6. Deep reviews specific PRs with code analysis

## Output Format

### PR List

```markdown
# PRs Awaiting Your Review

**Total**: 5 PRs | **Critical**: 1 | **High**: 2

## Critical Priority
### PR #456: [SECURITY] Fix auth bypass
- Author: @alice | Waiting: 1 day | Size: Small
- Summary: Patches auth vulnerability
- Recommend: Approve after verifying tests
```

### Deep Review

```markdown
# Deep Review: PR #234

## Change Summary
Adds new dashboard with analytics widgets

## Concerns
1. Performance: Consider memoizing calculation
2. Error Handling: Add try-catch for API calls
3. Missing tests for error states

## Verdict: Request Changes
```

## Priority Categories

| Priority | Criteria |
|----------|----------|
| Critical | Security, hotfix, urgent labels |
| High | >3 days waiting, priority label |
| Normal | Standard queue |
| Low | Draft, WIP label |

## Requirements

- GitHub CLI (`gh`) authenticated
- Claude Code with Opus 4.5 model access

## Updates

```bash
cd <clone-path>/command-review-prs && git pull
```
