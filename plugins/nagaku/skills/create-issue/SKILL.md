---
name: create-issue
description: >-
  Use this skill when the user wants to file an observation, bug, or task as a
  GitHub issue on a nagakuinc repository — or to trim an existing bloated issue
  down. Triggers: "Issue を作成する", "/create-issue", "issue にして", "issue
  立てて", "これを issue に書いて", "この issue を書き直して", "file an issue",
  "open an issue".
---

# Create a GitHub Issue

This file is English; the output is not necessarily. Issues are written in
the language the repository's existing issues are written in — for this
organization that is **Japanese** unless the repository's `CLAUDE.md` or its
issue list says otherwise.

Work out the repository from the checkout, not from memory:

```bash
gh repo view --json nameWithOwner -q .nameWithOwner
```

and pass it as `-R <owner/repo>` on every `gh` call. A subagent or a worktree
may sit in a directory `gh` would resolve to something else. When the issue
belongs to a repository other than the one you are in, the user names it.

## The rule

**The issue body follows the same policy as a code comment.** The codebase is
the code; the issue is the comment. Default to near-zero. Anything the reader
can recover by reading the repo stays out — they will read it anyway, and their
reading will be fresher than yours.

An issue exists to say **the one thing nobody can find by looking**: that this
is wrong, or that this should be done. Nothing else is owed.

Getting there takes two passes. You cannot judge what is excessive until it is
on the page, so write the long version first and cut it second. Never draft
straight into `gh issue create`.

## Where this runs

Decide first, from how the issue relates to what this session is actually about:

- **The issue is the subject** — the user came to file it, or the work just
  finished and this is the wrap-up. Do it inline. The context you already have
  is the material.
- **The issue is incidental** — you noticed it while doing something else, or
  the user wants to keep going on another task. Delegate Steps 1 and 2 to a
  subagent so the drafting does not consume this session's context or pull the
  work off course. Leave it in the background and carry on.

A subagent starts blank, so its prompt has to carry everything: the observation
in full, any evidence already in your context quoted verbatim (payload, log
line, URL, symbol names — never make it re-derive what you have), the
repository, and an instruction to invoke this skill.

Have it return the title, the final body, the labels, and the draft's path — not
create the issue. Step 3 stays here, because the user's go-ahead happens here.

## Step 1 — Draft it long

Write the full version to a scratch file outside the checkout. Everything you
observed and traced: entry paths, impact, the fix you have in mind, the
alternatives you rejected, how you would verify it. Structure it however comes
naturally. Do not edit while drafting — the point of this pass is to get what
you know out where you can see it.

## Step 2 — Cut it to the minimum

Now read the draft as the assignee, and keep only what is **not** recoverable
from the repo:

- **The observation itself** — the symbol and the defect, precisely named
- **External evidence** — a response payload, a failing URL, a log line, a link
  to the thread it came from (but see *Links are not content* below)
- **A decision only a human can make** — a trade-off with no answer in the code.
  State the options and your pick, briefly.
- **A constraint learned outside the code** — a vendor behavior, a user report,
  an intent the code contradicts

Everything else goes, even when true, even though you just wrote it:

- Call-site traces, entry-path enumerations, affected-file lists
- Impact analysis, severity ranking
- A proposed patch or code sketch
- Rejected alternatives
- Verification commands, spec case lists, checklists
- Restating in prose what a linked line already says
- An "investigated by Claude" footer

Sentence by sentence: *could the assignee derive this by reading the code, once
the first sentence told them where to look?* If yes, delete it. Usually only the
first sentence survives. One to three lines is normal. Reaching for a section
heading (`## …`) is the signal that a report is creeping back in — keep one only
when the body has genuinely separable parts, typically evidence plus a decision.

### Links are not content

A Claude Artifact, a Google Doc, a Notion page, a chat message: an agent picking
up the issue cannot open any of them, and they rot — edited, moved, permissions
revoked, deleted — while the issue stays. So ask what the link is doing there:

- **It carries the grounds for the issue** — the measurement, the payload, the
  sentence that makes the case. That goes in the body, stated concretely. Keep
  the link beside it if you like, but the issue has to stand on its own with
  that link dead.
- **It only says where this came from** — meeting minutes, a thread, a support
  ticket. A bare link is fine; nobody has to open it to act.

The length norm does not move for this. Inlining is not a licence to run long,
and an external analysis is not material to pad the body with: what crosses over
is the grounds, not the document holding them and not its narrative. Grounds are
usually a line or two of fact — a rate, a payload, a quoted constraint. If yours
are not, you are copying the report rather than lifting its point. Nor does a
pile of pointers count as grounds: "see the doc, the thread and the sheet"
leaves the assignee with nothing to act on.

Write the result to a second file; leave the draft on disk. The draft is
scaffolding, not a deliverable — it is worth keeping for the session because it
is the head start on the fix, but it does not get posted anywhere. Do not
relocate the cut material into a comment on the issue.

## Step 3 — Create

Title: symptom for a bug, action for a task, prefixed with the surface when it
is specific to one (`console: …`). The title carries most of the weight — it is
what people read in a list of a hundred.

Labels: read the current set with `gh label list -R <owner/repo> --limit 100`,
do not assume names — every repository has its own. One kind label plus every
affected area label where the set has those; a repository with no labels gets
none invented. Leave workflow and triage labels to the user unless they ask
for one.

Check for a duplicate:
`gh issue list -R <owner/repo> --search '<keyword>' --state all`.

Show the user the title, the final body, and the labels, and **wait for their
go-ahead** — an issue is public to everyone on the repository. The go-ahead
comes wherever the conversation is: a terminal, or the thread a box was
launched from. Mention the draft's path so they can pull something back if the
cut went too far. Then:

```bash
gh issue create -R <owner/repo> --title "<title>" --body-file <final> \
  [--label <kind> --label <area> …]
```

Trimming an existing issue instead — same two passes, with the current body as
the draft:

```bash
gh issue edit <number> -R <owner/repo> --body-file <final>
```

Report the resulting URL.
