---
name: evidence
description: "Assemble and judge the proof that one change fixes one bug, as one file per bug and head commit: the test failing on the base and passing on the head, the bug seen in the browser on the base build and gone on the head, the same check on the test environment, and the gaps, each as raw output taken on a named commit. Ends proven, partly proven or not proven. Never fixes. Use when a flow reaches its evidence step, before shipping or closing a bug fix, when another skill says to prove a fix, or when the user asks for proof that a bug is fixed rather than a claim that tests pass."
summary: "Proves a fix with raw red and green output"
---

# Evidence

"Tests pass" and "checked in the browser" are claims. Evidence is the output itself: the test failing before the fix and passing after it, the bug on screen before and gone after, each taken on a commit you can name. This unit gathers that output for one bug at one head commit, says what is missing, and gives a verdict.

It reads and judges. It never fixes anything.

## Input

- **Bug**: the ticket key and its expected vs actual. Without a ticket, `<repo>-<iid>` stands in for the key.
- **Change**: the repo, the head commit, and the base it is compared to (the merge base with the target branch). For an open MR or PR, its current head.
- **Sources**: where the raw output already is: the worker's report and log, the review report, the `UI-<KEY>-*.md` reports and their captures, the MR description.
- **Guard tests**: the tests every change must pass, from the flow's config.
- **User-visible**: whether a person sees the bug on a screen. When not, the browser parts are `not applicable`, with the reason.
- **May run** (optional): whether it may delegate a `ui-check` on the base build to produce a missing red browser part. Default: no.
- **Run folder**: `<tasks>/<KEY>/`.

## The file

`<tasks>/<KEY>/EVIDENCE-<KEY>-<short sha>.md`, one per bug and head commit. A new head is a new file; the old one stays as history.

At the top, a table with one row per part: the part, its state (`present`, `missing`, `stale`, `pending` or `not applicable`), and one line why. Then one section per part:

1. **Red test**: the new or changed test fails on the base. The command, the commit, and the raw failing output, trimmed to the lines that show the failure. For a test the change adds, that is the head's test file run against the base's code.
2. **Green test**: the same test passes on the head, and so do the guard tests. Each command, the commit, and the raw result lines with their counts.
3. **Red in the browser** (user-visible bugs): the check file's steps on the base build reproduce the bug. What was seen, a text snapshot excerpt or a screenshot path, and the console and network lines around it.
4. **Green in the browser**: the same steps on the head build, with the same kind of capture.
5. **Test environment**: the same check after release, on the deployed version that holds the head. `pending` until the flow has released; the flow fills it then.
6. **Gaps**: what wasn't checked and why. A gap is stated, never hidden.

The verdict closes the file.

## Steps

1. **Skip if done**: a file for this head exists and no source changed since → return it.
2. **Gather** each part from the sources. Copy the raw lines into the file, with the path of the full log or capture beside them. A sentence in a report ("all tests pass", "checked in the browser, works") is not a part: the part is the output the sentence is about.
3. **Check each part** before counting it:
   - it names the commit it was taken on: red parts on the base, green parts on the head
   - a green part taken on another head is `stale`, and counts as missing
   - the output shows what the part says: the failing assertion is the bug's, the counts add up, the snapshot shows the ticket's actual (red) or expected (green)
   - the green test is the same test as the red one
4. **Produce the red browser part** when it is missing, the bug is user-visible and **May run** says yes: delegate `ui-check` local on the base build with the bug's check file. `done reproduced` fills the part; `done not reproduced` is a gap ("the check doesn't catch the bug").
5. **Name the producer** of every part still missing: the implementer for the red and green tests (its report must carry the raw output), `review-change` for the guard tests, `ui-check` for the browser parts (`base` for red, `head` for green, the test environment URL for part 5). Never write a part from what it ought to say.
6. **Judge**, and write the verdict at the end of the file.

## Verdict

- **`proven`**: every part that applies and is due is present and current, and every green part passes. A `pending` part doesn't count against it.
- **`partly proven`**: some parts are present, none contradicts the fix, and others are missing or stale. List the gaps.
- **`not proven`**: neither red part is present (nothing shows the bug ever happened), or a green part fails, or a guard test fails.

## Rules

- **Raw output, never a summary.** Trim to the relevant lines; never paraphrase them.
- **Every item names its commit.** An item without one is missing.
- **Fixes nothing.** No edits, commits or pushes. A failing part is reported, not repaired.
- **Missing parts are named, not invented.** Say which unit produces each.
- **A gap is stated.** "Not checked in the browser: the bug is in an exported file no screen shows" records a choice; silence hides one.

## Output

`done proven`, `done partly proven <gaps>` or `done not proven <why>`, with the file's path. Or `blocked <reason>` when the sources can't be read.
