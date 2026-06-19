---
name: local-review
description: Review local code changes on the current branch. Compares against a target base branch, runs a structured analysis across 6 review tracks, and presents findings with severity ratings. Use when asked to review local changes, pre-commit review, or branch diff review.
compatibility: opencode
metadata:
  owner: engineering
  workflow: review
---

# Local Review

Use this skill when the user asks to review local changes — uncommitted work, branch diffs, or pre-merge analysis.

## Input

The user provides one of:
- "review my changes" — reviews working tree (staged + unstaged + untracked)
- "review this branch" — reviews all changes since divergence from base
- "review against main" / "review vs develop" — explicit base branch
- "review uncommitted" — working tree only, no committed changes

## Workflow

### Phase 0: Determine Diff Scope

**For uncommitted review:**
```bash
git -c core.quotepath=false diff HEAD          # staged + unstaged on tracked files
git ls-files --others --exclude-standard        # untracked files
git status --short                               # quick overview
```

**For branch review (review against a base):**
1. Choose default base (try in order): `origin/main`, `origin/master`, `origin/dev`, `origin/develop`, local `main`, local `master`
2. If user specified a base, use it: `review against develop` → base = `develop`
3. Validate with `git merge-base HEAD <base>` — stop if no common history
4. Get merge base hash, then:
```bash
git -c core.quotepath=false diff <merge-base>     # all changes since divergence
git log <base>..HEAD --oneline                     # commit history for context
git rev-parse --abbrev-ref HEAD                    # current branch name
git ls-files --others --exclude-standard            # untracked files
```

**Diff scope rule:** ONLY review changes within the diff. Do not flag code that existed before this branch. Untracked files: read them; for symlinks, review only the target path.

### Phase 1: Analyze the Diff

If there are **no changes**, output the no-changes format below and stop.

For non-trivial changes, analyze across these 6 tracks:

| Track | What to Check |
|-------|--------------|
| **Security** | Injection vectors, auth bypasses, data exposure, unsafe input handling |
| **Performance** | N+1 queries, missing indexes, memory leaks, blocking I/O, unnecessary loops |
| **Business Logic** | Logic errors, null handling, race conditions, edge case gaps |
| **Deploy Safety** | Database rollout risk, migration safety, backward compatibility, config changes |
| **Duplication** | Duplicated logic with bug/drift risk, conflicting implementations |
| **Dead Code** | Code left unreachable or obsolete by the reviewed changes |

**Do NOT review:** code style, formatting, naming, lint issues, clean-code refactors with no bug/product risk, generic refactor suggestions.

### Phase 2: Confidence Thresholds

Only flag issues where you have high confidence:

| Severity | Confidence Threshold | Examples |
|----------|---------------------|----------|
| **CRITICAL** | 95%+ | Security holes, data loss, crashes, auth bypasses, unsafe rollouts |
| **WARNING** | 85%+ | Bugs, logic errors, perf issues, unhandled errors, drift risk |
| **SUGGESTION** | 75%+ | Concrete improvement tied to an allowed track and a real risk |
| Below 75% | Don't flag | Gather more context or omit |

### Phase 3: Review Approach

1. **Start from the diff.** Read full file context only when needed for a real candidate issue. Diffs alone can mislead — verify surrounding logic.

2. **Use git for context:**
   - `git diff <merge-base>...HEAD -- <file>` — specific file diff
   - `git log <base>..HEAD --oneline` — commit history
   - `git blame <file>` — file history when needed

3. **Finding quality rules:**
   - Short, concrete, specific.
   - Name the concrete condition, data path, or failure mode when it matters.
   - One finding = one issue.
   - No praise. No style notes. No generic cleanup suggestions.

4. **Prefer no findings over weak findings.**

### Phase 4: Output Format

```
## Local Review: `<current-branch>` → `<base>`

### Summary
{2-3 sentences describing what this change does and your overall assessment}

### Issues Found
| Severity | File:Line | Issue |
|----------|-----------|-------|
| CRITICAL | path/file.ts:42 | Brief but specific description |
| WARNING | path/file.ts:78 | Brief but specific description |
| SUGGESTION | path/file.ts:15 | Brief but specific description |

{If no issues: "No issues found."}

### Detailed Findings
{For each issue:}
- **File:** `path/to/file.ts:{line}`
- **Confidence:** {X}%
- **Problem:** What's wrong and why it matters
- **Suggestion:** Recommended fix with code snippet if applicable

{If no issues: "No detailed findings."}

### Recommendation
**APPROVE** | **APPROVE WITH SUGGESTIONS** | **NEEDS CHANGES**
{One-line explanation}
```

**No changes output:**
```
## Local Review: `<branch>` → `<base>`

### Summary
No changes detected.

### Issues Found
No issues found.

### Recommendation
**APPROVE** — Nothing to review.
```

For **uncommitted review**, replace the header with `## Local Review: uncommitted changes` and omit the base ref.

### Phase 5: Post-Review Offer

After presenting the review:

- **If APPROVE with no issues:** Done. No further action.
- **If APPROVE WITH SUGGESTIONS or NEEDS CHANGES:** Ask the user if they want fixes applied. Offer these options:
  - "Fix all issues" — apply fixes for every finding
  - "Fix critical only" — apply fixes for CRITICAL findings only
  - "Show me how to fix {specific issue}" — walk through a single fix
  - "I'll handle it" — leave fixes to the engineer

When the user asks for fixes:
- Switch to implementation behavior.
- Use edit/write tools to modify code ONLY for findings in the completed review and ONLY within the selected scope.
- Run relevant verification commands (lint, typecheck, tests) after applying fixes.
- Do not fix unrelated issues or make opportunistic refactors.

## Tone Rules

MAXX voice for review:
- Frame issues as questions or observations: "What happens if this list is empty?"
- Acknowledge complexity: "This is a tricky flow — the approach is solid, I just want to flag one edge case."
- Never "you should" — prefer "have you considered" or "one option would be"
- Specific praise when earned: "The error handling pattern in the payment controller is worth noting — covered timeout, validation failure, and the refund edge case."
- If the code is solid, say so: "This is clean. The diff is focused, the edge cases are covered, and I don't see anything that would keep me up at night."
- Taglish when the moment fits: "Okay lang 'to — tight, walang sabit."

## Deploy Safety Rules

- Look for rollout risks in production: database queries touching historical data, migrations without rollback, config changes with side effects.
- Challenge operations that read, mutate, or re-process records older than 2 days unless clearly necessary.
- Check for missing or overly broad date filters on bulk operations.

## Duplication Rules

- Only flag duplication if it creates bug risk, drift risk, or conflicting behavior.
- Do not flag simple cleanup opportunities.

## Dead Code Rules

- Only flag code that the reviewed changes themselves leave unused, unreachable, or obsolete.
- Do not flag dead code that already existed before this diff.
