---
name: comment-on-pr
description: Use when given a GitHub PR link and asked to review it and leave, post, or drop comments on it, to approve it, or to comment on a teammate's PR for the user.
---

# Comment on a PR

## Overview

Review a GitHub PR and post review comments on specific lines under the user's identity. The output is a
short set of high-value comments, shown to the user before anything is posted.

These comments land on a teammate's work, signed as the user. Volume and confidence both cost
the user credibility. Fewer, verified, friendly comments beat thorough ones.

## Gather

```bash
gh pr view <url> --json title,body,author,baseRefName,headRefName,additions,deletions,changedFiles
gh pr diff <url>
gh pr view <url> --json headRefOid --jq '.headRefOid'   # commit_id for the review
```

Read surrounding code, not just the diff — a local checkout (`git fetch origin <headRefName>`,
then `git show origin/<headRefName>:<path>`) or `gh api repos/<owner>/<repo>/contents/<path>?ref=<branch>`.
Read the spec file when there is one — it's for checking your claims, not for auditing coverage.

## What earns a comment

One test: **would this break something for a real user, or lose real data?** A wrong value, a
silent failure, a security hole, a crash on input a caller actually produces. If you can't name
the concrete thing that goes wrong, it doesn't clear the bar.

Below the bar — leave these out entirely:

- Naming, formatting, import order, comment wording, type-annotation style.
- "Consider extracting", "could be simplified", duplication, structure and architecture
  preferences — a different way to write working code is not a finding.
- Missing tests, unless the untested thing is the defect itself.
- Hypothetical inputs no caller produces; edge cases the code already handles downstream.
- Defensive guards against things that can't happen.
- Anything you'd phrase as "might be worth" or "just a thought".

Most PRs produce zero comments, and zero is a complete review. One or two is a busy PR. If you
have four, the bar slipped — re-check each one against the test above and cut what fails. Never
pad toward a number to look thorough.

## The verdict

Two independent decisions: what to say, and whether to approve. Having findings does not
withhold approval — around here the normal outcome is review, comment, **and** approve, in good
faith that the author addresses the comments before merging.

**Every draft you show the user carries an approval recommendation.** One of:

- **Nothing cleared the bar** — propose `APPROVE` alone with a one-line body (see *The approval
  body* below). Good enough is a real verdict and the most common one on healthy work; a clean
  PR gets approved, not searched until something turns up. Below-bar observations stay off the
  PR entirely — mention them in chat if they're worth anything.
- **Findings cleared the bar** — propose the comment review *and* recommend approving alongside
  it. This is the default. Ask the user; don't decide for them.
- **A finding would ship a real defect** — recommend holding approval and name the finding that
  is the reason. `REQUEST_CHANGES` blocks a colleague's PR and is the user's call alone; never
  put it in a payload unasked.

Never leave approval unmentioned. Silence reads as "not approvable" and strands the author on a
ticket they can't move. If you're not recommending approval, say which finding is the reason.

## The approval body

A review has exactly two kinds of text: the line comments, and one short approval line. There
is no summary paragraph. The line comments carry the substance; the approval body is a
thumbs-up, not a recap.

Short and casual. One line, usually under 12 words.

- **Clean PR** — "LGTM", "Looks good 👍", "Nice, LGTM", "All looks good to me", "LGTM, thanks!"
- **Approving alongside comments** — acknowledge them in passing and approve anyway: "Just a
  couple of small things but otherwise looks good!", "Left a few notes, nothing blocking — LGTM",
  "A couple of suggestions, but happy with this", "Small comments, otherwise LGTM".

Vary it. The user leaves these across many PRs and the same string every time reads like a bot.
Pick a different phrasing than the last approval; the lists above are examples of the register,
not a rotation to cycle through.

No em dashes anywhere in posted text (body or comments). Use a comma, colon, or period instead.

Do not say "inline" in the body (or in the comments). The reader is looking at the comments; say
"left a few notes" or "a couple of comments", not "inline".

Never in an approval body: a recap of what the PR does, a restatement of the findings, a
list of what you checked, praise paragraphs, or "these are just suggestions" caveats. If it
needs more than a line, it belongs in a line comment.

When the user holds approval, the `COMMENT` review still needs a body (the API requires one for
`COMMENT` and `REQUEST_CHANGES`). Same register, one line — "Left a few notes", "Couple of
things worth a look before this goes in". Still not a summary.

## What each comment is

**Two or three sentences. Under 50 words.** What breaks, then the fix. That's the whole comment.

> This throws when `items` is empty — the dashboard 500s on a new account. Guard before the
> `reduce`?

> `retryCount` never resets after a success, so the fourth failure in a session gives up
> immediately. Reset it in the success branch?

Plain language, no jargon, no build-up. Don't restate what the code does, don't explain your
reasoning, don't list what else you checked, don't hedge with "I might be missing something".
Ending the fix as a question ("Guard before the reduce?") keeps it collegial without padding.

No code blocks unless the fix genuinely can't be said in a sentence. No preamble, no sign-off.

No summary body. Whatever you would have put there either clears the bar and belongs on a line
of code, or doesn't and belongs nowhere.

## Confidence

Every behavioral claim is verified or marked as unverified.

- Verify pure functions by running them (`node -e`, the existing spec file).
- Verify surrounding behavior by reading the actual file at the head commit.
- **Never write numbered repro steps you have not executed.** Reasoning from source is fine —
  say so in the comment: "I haven't run this, but `handleBlur` at :31 suggests…".

A confident repro that turns out wrong, posted publicly under the user's name, costs more than
the finding was worth.

## The post gate

**Show the complete draft in chat. Wait for an explicit yes. Then post.**

"Review it and comment for me" authorizes posting. It does not authorize posting text the user
has not read.

No exceptions:
- Not when the user said "and comment on it" or "leave comments" — that is the authorization to
  post, not permission to skip the draft.
- Not when the findings are obviously correct.
- Not when showing the draft feels like stalling or wasting a turn.
- Not by posting and showing the draft in the same message.

| Rationalization | Reality |
|---|---|
| "They already told me to post, asking is stalling" | They authorized the action, not the wording. One turn is cheap; a bad comment on a colleague's PR is not. |
| "It's reversible — I can delete the comments" | Deleting review comments leaves the email notifications. Not reversible. |
| "I'll post and mention they can edit after" | The author has already read it by then. |
| "The draft is in my summary, that counts" | Shown after posting is not shown before posting. |

### Red flags — stop, show the draft

- About to call `gh api ... --method POST`, `gh pr review`, or `gh pr comment` in the same
  message that first states the findings
- Thinking "they clearly want this posted"
- A comment in the payload you could not defend against the bar
- A draft with findings that says nothing either way about approving
- Any numbered repro step you did not execute

## Posting

Write the payload to a file, then POST it.

`line` is the line number **in the file at the head commit**, not a diff position. Every line
must fall inside a diff hunk or the API rejects it.

Re-read `headRefOid` immediately before posting. Some branches get force-pushed between review
and post (release-bot branches especially), and a stale `commit_id` describes code that is no
longer what merging would ship.

**Approve with no comments** — one call, `event: "APPROVE"`, a `body` with the approval line,
and no `comments` key at all.

**Approve with comments** — one review, not two. Create it pending (omit `event`), then submit
it as the approval so the line comments and the approval line ship together in a single
notification:

```jsonc
// payload.json — no "event" key: this creates a PENDING review
{
  "commit_id": "<headRefOid>",
  "comments": [
    { "path": "src/foo.ts", "line": 73, "side": "RIGHT", "body": "<comment>" }
  ]
}
```

```bash
REVIEW_ID=$(gh api repos/<owner>/<repo>/pulls/<n>/reviews --method POST \
  --input payload.json --jq '.id')
gh api repos/<owner>/<repo>/pulls/<n>/reviews/$REVIEW_ID/events --method POST \
  -f event=APPROVE -f body='<approval line>' --jq '{id, state, html_url}'
```

**Comments without approval** (the user held approval) — same pending review, submitted with
`event=COMMENT` and the one-line body from *The approval body*.

Verify after posting:

```bash
gh api repos/<owner>/<repo>/pulls/<n>/comments --jq '.[] | "\(.path):\(.line)"'
```

A pending review that never gets submitted is invisible to the author — if the submit call
fails, say so rather than reporting the review as posted.

Report the review URL and the state back to the user.

## Common mistakes

| Mistake | Fix |
|---|---|
| Comments that don't clear the bar, added for coverage | Cut them. A zero-comment review is fine. |
| A "consider extracting / could be cleaner" comment | Not a finding. Cut it. |
| Four or more comments on one PR | The bar slipped. Re-test each against "what breaks?" |
| Withholding approval on a PR with nothing wrong | Propose `APPROVE`. Good enough is a verdict. |
| Showing a comment draft with no approval recommendation | Every draft says approve or names why not |
| Demoting a real finding to the summary to keep the list short | Comment on everything that clears the bar |
| Comments longer than a few sentences | What breaks, then the fix. Under 50 words. Cut the rest. |
| A paragraph in the approval body | One casual line — "LGTM", "Small comments, otherwise looks good!" |
| Writing a summary body for the review | There isn't one. Line comments + one approval line, nothing else. |
| The same approval wording as last time | Vary the phrasing; identical strings read as automated |
| Using diff position for `line` | File line at the head commit, `side: "RIGHT"` |
| Reviewing the diff alone | Read the file and its spec at the head commit |
| Asserting an unexecuted repro | Run it, or mark it as reasoned from source |
