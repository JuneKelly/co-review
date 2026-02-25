---
description: Co-review changes with the human operator
---

# Co-Review

Collaborate with the human operator to review a unit of work. Your dual role:
review the code AND help the operator build their understanding by walking
through changes together. This defines walkthrough structure only;
review policy (what to look for, how to evaluate) comes from elsewhere.

## 1. Resolve the review subject

If operator specifies a target, use it. Otherwise infer or ask:

- Non-main branch with pending changes → review pending changes
- Non-main branch, no pending changes → review current branch vs main
- Main branch → list 3–5 most recent branches
  (`git branch --sort=-committerdate | head -5`), ask operator to pick

Operator input:

```text
$ARGUMENTS
```

### Gathering the diff

Always diff from the divergence point, not from the current checkout.

**Default branch detection:**
`git rev-parse --abbrev-ref origin/HEAD`, fallback `git rev-parse --verify
origin/main`, then `origin/master`. Below, `<default>` = detected branch.

**Pending local changes:** `git diff` and `git diff --staged`.

**Current branch vs default:**
`git merge-base <default> HEAD` → diff and log from that merge-base, not from
the tip of the default branch.

**Remote branch** (operator pasted a branch name / PR):
Follow these steps exactly:

1. `git fetch` — ensure remote-tracking refs are current.
2. Verify ref exists: `git rev-parse --verify origin/<branch>`
   If unresolved: tell operator (typo? already merged/deleted?). Do not
   fallback or guess.
3. Merge-base from remote refs only:
   `git merge-base origin/<default> origin/<branch>`
4. Diff and log from merge-base:
   `git diff <merge-base>..origin/<branch>`
   `git log <merge-base>..origin/<branch>`
5. If merge-base is far behind default tip, flag it — branch may need rebase,
   and this affects the review.

CRITICAL: After fetch, ALWAYS use `origin/<ref>` remote-tracking refs. Never
bare local names (`main`/`master`) — local branches are stale, inflating diffs.

**Scale:** If diff >~2,000 lines, tell operator upfront. Ask: full detail,
focused areas, or high-level pass. Agree on scope before proceeding.

## 2. Orientation & review plan

Present concise overview (mental map before diving in):

- **Scope**: files touched, rough change size
- **Intent**: what the changes accomplish (commit messages, PR description,
  code reading)
- **Structure**: notable architectural or structural shifts (new modules,
  moved files, changed interfaces)

Group changes into logical units to ease review (individual modules,
groups of related functions, vertical slices of functionality). Propose
walkthrough order (dependency / data-flow / cohesion) — for easiest
comprehension.

Invite operator to adjust order or skip areas. Keep this brief — orientation,
not analysis.

## 3. Walkthrough

For each review unit, present:

- What changed and why
- Anything notable: complex logic, potential issues, intent questions,
  things worth a closer look

Operator has the diff open. Explain and flag; don't recite code. Operator
decides what matters — you present and point things out.

CRITICAL: Present ONE unit, then STOP. Do not continue until the operator
responds. This is a conversation, not a report — presenting all units at once
defeats the purpose. Operator reactions within and between units are the point.

NEVER move on to the next unit or step until the human operator says to do so.

## 4. Wrap-up

- Raise any cross-cutting concerns: consistency, data-flow / error-handling
  patterns spanning boundaries, missing change propagation
- Recap open questions or concerns from walkthrough
- List concrete action items if any emerged
- Offer follow-up help: fixes, tests, commit messages, etc.
