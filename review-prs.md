---
argument-hint: "[filter]"
description: "Review PRs assigned to you with AI-generated suggestions"
model: claude-opus-4-5-20251101
allowed-tools: ["Bash", "Read", "Glob", "Grep", "WebFetch", "AskUserQuestion"]
---

**If `$ARGUMENTS` is empty or not provided:**

List and summarize all PRs that need your review.

**Usage:** `/review-prs [filter]`

**Examples:**

- `/review-prs` - All PRs needing your review
- `/review-prs --team` - PRs for your team
- `/review-prs --stale` - PRs waiting >3 days
- `/review-prs 123` - Deep review of specific PR

**Workflow:**

1. Fetch PRs assigned for review
2. Analyze changes and generate summaries
3. Identify potential issues
4. Provide review suggestions
5. Highlight priority items

Proceed with fetching PRs needing review.

---

**If `$ARGUMENTS` is provided:**

Filter PRs or perform deep review of specific PR.

## Configuration

- **Filter**: `$ARGUMENTS`
  - PR number: Deep review specific PR
  - `--team`: Include team member PRs
  - `--stale`: PRs waiting >3 days
  - `--critical`: High-priority PRs only
  - Repo name: Filter by repository

## Steps

1. **Fetch PRs Needing Review**

   ```bash
   # PRs where you are requested reviewer
   gh pr list --search "review-requested:@me" --json number,title,author,createdAt,additions,deletions,url,labels,headRefName,baseRefName

   # Also check team review requests
   gh pr list --search "team-review-requested:@me" --json number,title,author,createdAt,url
   ```text

   Collect:
   - PR number and title
   - Author
   - Age (days waiting)
   - Size (additions/deletions)
   - Labels
   - CI status

2. **Categorize PRs**

   **By Priority:**
   - **Critical**: Has `urgent`, `hotfix`, or `security` label
   - **High**: >3 days waiting or has `priority` label
   - **Normal**: Standard review queue
   - **Low**: Draft PRs or `wip` label

   **By Size:**
   - **Small**: <50 lines changed
   - **Medium**: 50-200 lines
   - **Large**: 200-500 lines
   - **Extra Large**: >500 lines

   **By Type:**
   - Feature (new functionality)
   - Bug fix
   - Refactor
   - Documentation
   - Dependencies

3. **Generate PR Summaries**

   For each PR, analyze:

   ```bash
   # Get PR details
   gh pr view <number> --json title,body,files,commits,reviews,comments

   # Get diff
   gh pr diff <number>
   ```text

   Create summary:
   - What the PR does (1-2 sentences)
   - Key files changed
   - Potential risk areas
   - Test coverage status
   - CI status

4. **Deep Review Analysis (for specific PR)**

   When reviewing a specific PR:

   **a. Understand context**
   - Read PR description
   - Check linked issues
   - Review commit history

   **b. Analyze changes**

   ```bash
   # Get changed files
   gh pr view <number> --json files | jq '.files[].path'

   # Get full diff
   gh pr diff <number>
   ```text

   **c. Code review checklist**
   - Logic correctness
   - Error handling
   - Edge cases covered
   - Security implications
   - Performance impact
   - Test coverage
   - Documentation updates

   **d. Generate suggestions**
   - Specific line comments
   - Alternative approaches
   - Missing test cases
   - Documentation gaps

5. **Check CI Status**

   ```bash
   gh pr checks <number>
   ```text

   Flag:
   - Failing checks (blockers)
   - Pending checks (wait or review anyway)
   - Skipped checks (investigate why)

6. **Generate Review Report**

   ```markdown
   # PRs Awaiting Your Review

   **Total**: 5 PRs | **Critical**: 1 | **High**: 2 | **Normal**: 2

   ## Critical Priority

   ### PR #456: [SECURITY] Fix authentication bypass
   - **Author**: @alice
   - **Waiting**: 1 day
   - **Size**: Small (32 lines)
   - **Status**: CI passing
   - **Summary**: Patches auth bypass vulnerability in login endpoint

   **Quick Assessment**:
   - Security fix, should prioritize
   - Changes are focused and well-tested
   - Recommend: Approve after verifying test coverage

   ---

   ## High Priority

   ### PR #234: Add user dashboard feature
   - **Author**: @bob
   - **Waiting**: 4 days
   - **Size**: Large (423 lines)
   - **Status**: CI passing
   - **Summary**: New dashboard with analytics widgets

   **Review Notes**:
   - New feature, needs thorough review
   - Consider performance of data fetching
   - Missing tests for error states

   **Suggested Comments**:
   1. `src/dashboard/Analytics.tsx:45` - Consider memoizing this computation
   2. `src/api/dashboard.ts:23` - Add error handling for API failure

   ---

   ## Normal Priority

   ### PR #567: Update dependencies
   - **Author**: @dependabot
   - **Waiting**: 2 days
   - **Size**: Small (12 lines)
   - **Status**: CI passing
   - **Summary**: Routine dependency updates

   **Quick Assessment**: Safe to approve if CI passes

   ---

   ## Review Queue Summary

   | PR | Title | Author | Age | Size | Priority |
   |----|-------|--------|-----|------|----------|
   | #456 | Security fix | @alice | 1d | S | Critical |
   | #234 | Dashboard | @bob | 4d | L | High |
   | #345 | Refactor auth | @carol | 3d | M | High |
   | #567 | Deps update | @bot | 2d | S | Normal |
   | #678 | Docs update | @dave | 1d | S | Normal |

   ## Recommended Actions

   1. **Approve now**: PR #456 (security), #567 (deps)
   2. **Review today**: PR #234 (waiting 4 days)
   3. **Schedule time**: PR #345 (medium complexity)
   ```text

7. **Deep Review Output (for specific PR)**

   ```markdown
   # Deep Review: PR #234

   ## Overview
   - **Title**: Add user dashboard feature
   - **Author**: @bob
   - **Branch**: `feature/dashboard` → `main`
   - **Size**: +423 -12 across 15 files

   ## Change Summary

   This PR adds a new user dashboard with:
   - Analytics overview widget
   - Recent activity feed
   - Quick action buttons
   - Responsive layout

   ## Files Changed

   | File | Changes | Notes |
   |------|---------|-------|
   | src/dashboard/index.tsx | +120 | Main dashboard component |
   | src/dashboard/Analytics.tsx | +89 | Analytics widget |
   | src/api/dashboard.ts | +45 | API integration |
   | src/types/dashboard.ts | +23 | TypeScript types |

   ## Code Analysis

   ### Strengths
   - Clean component structure
   - Good TypeScript typing
   - Follows existing patterns

   ### Concerns

   #### 1. Performance (Medium)
   **File**: `src/dashboard/Analytics.tsx:45-67`
   ```typescript
   // Current: Recalculates on every render
   const stats = calculateStats(data);

   // Suggested: Memoize expensive calculation
   const stats = useMemo(() => calculateStats(data), [data]);
   ```text

   #### 2. Error Handling (High)

   **File**: `src/api/dashboard.ts:23`

   ```typescript
   // Current: No error handling
   const data = await fetch('/api/dashboard');

   // Suggested: Add try-catch and error state
   try {
     const data = await fetch('/api/dashboard');
     if (!data.ok) throw new Error('Failed to fetch');
   } catch (error) {
     // Handle error
   }
   ```text

   #### 3. Missing Tests

   - No tests for Analytics component
   - No tests for error states
   - No tests for loading states

   ## Suggested Review Comments

   1. **Line 45**: Consider memoizing `calculateStats` to avoid recalculation
   2. **Line 23**: Add error handling for API failures
   3. **General**: Please add tests for Analytics component

   ## Verdict

   **Recommendation**: Request Changes

   The feature looks good overall but needs:
   - [ ] Error handling for API calls
   - [ ] Tests for new components
   - [ ] Performance optimization for stats calculation

   Once addressed, this should be ready to merge.

   ```text

8. **Quick Actions**

   Provide commands to take action:

   ```bash
   # Approve PR
   gh pr review <number> --approve

   # Request changes
   gh pr review <number> --request-changes --body "Please address the comments above"

   # Add comment
   gh pr comment <number> --body "Looks good! Just a few minor suggestions."

   # Merge when ready
   gh pr merge <number> --squash
   ```text

## Output Structure

```markdown
# PRs Awaiting Review

## Summary
[Count by priority and status]

## Critical/High Priority
[PRs needing immediate attention]

## Normal Priority
[Standard review queue]

## Recommended Actions
[What to do next]
```text

## For Deep Review

```markdown
# Deep Review: PR #XXX

## Overview
[PR metadata]

## Change Summary
[What the PR does]

## Code Analysis
[Detailed review with suggestions]

## Verdict
[Approve/Request Changes/Comment]
```text

## Notes

- Prioritize security and bug fixes
- Large PRs may need in-person discussion
- Check if author is waiting or still working
- Consider reviewer fatigue for large queues
- Stale PRs may have merge conflicts
