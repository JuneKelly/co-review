---
description: Co-review changes with the human operator
---

Collaborate with the human operator to review a unit of work. The goal is to
jump-start their understanding of the changes and walk through them together
systematically. This command defines only the structure of a co-review — what to
look for and how to evaluate code should come from review policy defined
elsewhere.

## 1. Resolve the review subject

The operator might specify what to review, like "the current changes" or
"remote branch foo". Otherwise, infer the likely target or ask:

- non-main branch with pending changes: review pending changes
- non-main branch without pending changes: review current branch vs main
- main branch: list the 3–5 most recently active branches
  (`git branch --sort=-committerdate | head -5`) and ask the operator to pick

Operator input: ```text $ARGUMENTS```

### Gathering the diff

The diff must be relative to where the changes diverge from their base — not
relative to whatever is currently checked out locally.

**Detecting the default branch:**
Not all repos use `main`. Detect it with
`git rev-parse --abbrev-ref origin/HEAD`, falling back to
`git rev-parse --verify origin/main` then `origin/master`.
The steps below use `<default>` as a placeholder.

**Pending local changes:**
Use `git diff` and `git diff --staged`.

**Current branch vs default:**
Find the divergence point with `git merge-base <default> HEAD`, then diff and
log from there — not from the tip of the default branch.

**Remote branch (operator pasted a branch name from a PR):**
This is the case most likely to go wrong. The local checkout is almost
certainly stale relative to the remote. Follow these steps exactly:

1. `git fetch` to ensure remote-tracking refs are current.
2. Verify `origin/<branch>` exists:
   `git rev-parse --verify origin/<branch>`
   If it does not resolve, tell the operator immediately — the name may be
   misspelled, or the branch may already be merged and deleted. Do not fall
   back to a different ref or guess.
3. Find the merge base using **remote-tracking refs only**:
   `git merge-base origin/<default> origin/<branch>`
4. Diff and log from the merge base:
   `git diff <merge-base>..origin/<branch>`
   `git log <merge-base>..origin/<branch>`
5. If the merge base is far behind the current default branch tip, tell the
   operator — the branch may need a rebase, and that context affects the
   review.

CRITICAL: Always use `origin/<ref>` remote-tracking refs after fetching. Never
use bare local branch names like `main` or `master` — the local branch is
almost always behind the remote branch, producing a wildly inflated diff.

### Scale check

After gathering the diff, note its size. If it exceeds ~2,000 lines, tell the
operator upfront and ask whether to review at full detail, focus on specific
areas, or do a high-level pass. Negotiate the scope before proceeding.

## 2. Orientation & review plan

Present a concise overview to give the operator a mental map before diving in:

- **Scope**: which files and modules are touched, rough size of the change
- **Intent**: what the changes accomplish (from commit messages, PR description,
  or your reading of the code)
- **Structure**: any notable architectural or structural shifts (new modules,
  moved files, changed interfaces)

Then group the changed files into logical modules or areas. Present the proposed
walkthrough order — by dependency, data flow, or logical cohesion — whatever
makes the changes easiest to follow.

Invite the operator to adjust the order or skip areas they don't need help with.

Keep this brief. The goal is orientation, not analysis.

## 3. Module-by-module walkthrough

For each module in the plan, present:

- What changed and why
- Anything notable: complex logic, potential issues, questions about intent,
  things worth a closer look

Assume the operator can see the diff in another UI. Focus on explaining and
flagging, not reciting code. The operator decides what matters — you present
and point things out.

CRITICAL: Present ONE module, then STOP. Do not continue to the next module
until the operator responds. This is a conversation, not a report — if you
present all modules in a single message, you have defeated the purpose of
this command. The operator's reactions and questions between modules are the
point.

## 4. Wrap-up

- Raise any cross-cutting concerns: consistency across modules, data flow or
  error handling patterns that span boundaries, missing propagation of changes
- Recap open questions or concerns from the walkthrough
- List concrete action items if any emerged
- Offer to help with follow-up: fixing issues, writing tests, drafting commit
  messages, etc.