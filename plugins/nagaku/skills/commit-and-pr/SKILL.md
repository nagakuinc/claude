---
name: commit-and-pr
description: >-
  Read this before any commit or pull request on any nagakuinc repository:
  before the first `git commit`, `git commit --amend`, or `gh pr create`. The
  trigger is the action, not the request — it applies when the user asks
  ("コミットして", "PR を作って", "PR 出して", "/commit-and-pr", "commit this",
  "push this up"), when a task ends in work that has to land, and when you
  decide to commit on your own without being asked. Also applies when only
  committing, and when only opening the PR for commits that already exist.
---

# Commit, then open the PR

Committing at all is still the user's call — do it when they ask, or when the
task you were handed plainly ends in one. Work being finished is not by itself
a reason to commit. But once you are committing, you are doing it from here,
whoever decided.

## Which repository, in which words

Work out the repository from the checkout, not from memory:

```bash
gh repo view --json nameWithOwner -q .nameWithOwner
```

and pass it as `-R <owner/repo>` on every `gh` call from then on — a subagent
or a worktree may sit in a directory `gh` would resolve to something else.

The repository decides the surface conventions, and this skill does not:
the language commits are written in, whether a subject carries an `area:`
prefix, which checks gate a push. Read its `CLAUDE.md` and the last twenty
subjects of `git log` and match them. Where the repository says nothing, the
defaults below apply.

## Two texts, one reason

A commit body is permanent: on a squash-merging repository every branch
commit's body becomes part of `main`'s history, and the PR body does not.
Worth keeping → the commit body. Only matters this week (issue links, review
notes, what you could not verify) → the PR.

## Before committing

Branch first, never commit on `main`. Read `git status` and `git diff`; stage
by path, not `git add -A`.

Decide the commit boundaries before writing anything. **One commit, one
statement**: if the subject needs an "and", or names two surfaces, split it.
Needing "and" is evidence worth checking rather than the rule itself — ask
whether it is two independent changes sharing a description.

Run the checks the repository names for what you touched (its `CLAUDE.md`,
its CI workflow), and confirm the command could actually reach your diff. Fix
a failure; note an uncovered check in the PR.

## Writing the message

```
<subject: one line, lowercase after any prefix, no period, 50–75 chars>

<body, wrapped at 72>

<the Co-Authored-By trailer the harness hands you>
```

The subject is one sentence about the **resulting state**, not the action
taken: "requests now sit behind sts auth", not "fix auth" or "update
auth.tf". Declarative on purpose — an imperative subject tends to redescribe
the action the diff already shows, and stating what is true afterwards is one
step closer to the why. A repository that writes imperative subjects wins over
this default; a mechanical `feat:` / `fix:` / `chore:` prefix never does, since
it buckets a change the reader can already see. Name the surface in the
sentence when it is not obvious, or with the `area:` prefix a repository asks
for.

The body carries **why this shape**, never what changed where — the diff says
that already. Weave into prose, not labelled bullets: why this change, what
alternative was rejected and why this one won, anything knowingly deferred.
Record a decision only when it was a real tradeoff someone could otherwise
revert by mistake, including when the honest reason is that nothing documents
the right answer. Record the path as well as the destination — an approach you
tried and dropped, and what it could not do; leave it out and the next person
walks it again. Never editorialize about the code you replace or guess at why
it was that way. Never write the why as a comment in the implementation: a
reader of the code later has no use for reasoning that is not a constraint the
code must obey.

The trailer is the one the harness gives you; failing that,
`Co-Authored-By: Claude <model> <noreply@anthropic.com>` — the model, not the
vendor. Add a human co-author only if one drove the change.

Never include: issue or PR numbers in any form (`Fixes #123`, `Refs`, a bare
`#123`), branch names, "per review", CI links, test plans, anything addressed
to a reviewer rather than a reader. Trackers move, get renumbered, or are not
visible to everyone who reads the log later. Two exceptions, because they do
not rot: a permalink to something public and durable (an RFC, a vendor doc, a
specific commit) and `Co-Authored-By:` trailers.

Write it to a scratch file outside the checkout, then:

```bash
git commit -F "$SCRATCH/commit-message.txt"
```

Before pushing, fold corrections in: `git commit --amend`, or `git commit
--fixup=<sha>` then `git rebase --autosquash main`. `--fixup` alone leaves a
literal `fixup!` commit in permanent history, and `-i` hangs waiting on an
editor. Once a reviewer has seen a commit, correct it with a new one written
for the reader, not the reviewer.

## Opening the PR

```bash
git push -u origin HEAD
gh pr create -R <owner/repo> --title "<commit subject>" \
  --body-file "$SCRATCH/pr-body.md"
```

On a multi-commit branch the title summarises the whole change, under the same
rules.

If the repository has `.github/PULL_REQUEST_TEMPLATE.md`, fill it as-is, no
invented headings. Otherwise the body carries what did not belong in a commit
body: the background a reviewer needs to judge the change — what prompted it,
what constraint shaped it — plus out-of-scope notes and what you could not
verify. Issue references go here and only here: `Refs #123`, plus
`Closes #123` if merging should close it.

## Handing it over

```bash
gh pr checks <number> -R <owner/repo> --watch
```

Report the URL once checks pass. A decision that was not yours gets asked on
the PR, not in your reply.
