# comment-on-pr

A Claude Code skill for reviewing a GitHub PR and posting review comments **under your own
identity** — with a high bar for what earns a comment and a hard stop for your approval before
anything is posted.

## Why

Most PR-review automation fails the same two ways: it posts too much, and it posts without asking.
Volume and false confidence both cost you credibility with the person whose PR it is — and the
comments are signed with your name, not the model's.

This skill is built around that constraint:

- **One test for every comment** — *would this break something for a real user, or lose real data?*
  Naming, formatting, "consider extracting", missing tests, and hypothetical inputs are explicitly
  below the bar. Most PRs produce **zero** comments, and zero is a complete review.
- **Nothing is posted until you say so.** The full draft — every comment, verbatim, plus the
  approval recommendation — is shown in chat first. "Review it and comment for me" authorizes the
  action, not the wording.
- **Approval is a separate decision from having findings.** The normal outcome is review, comment,
  *and* approve. Every draft carries an explicit approval recommendation, so a clean PR never
  strands its author on a ticket they can't move.
- **`REQUEST_CHANGES` is never sent unasked.** Blocking a colleague's PR is your call alone.
- **Claims get verified or labeled.** Pure functions are run, surrounding behavior is read at the
  head commit, and no repro steps are written that weren't actually executed.

## Install

```
/plugin marketplace add jackmparker/comment-on-pr
/plugin install comment-on-pr
```

## Requirements

- The [`gh` CLI](https://cli.github.com/), authenticated (`gh auth login`) with `repo` scope.
  Reviews post as the authenticated account.
- Read access to the repo, and permission to submit reviews on it.

## Usage

Paste a PR link and ask for a review:

```
/comment-on-pr https://github.com/owner/repo/pull/123
```

Or just say it in your own words — "review this PR and leave comments:
`https://github.com/owner/repo/pull/123`".

You'll get back a draft: the comments with their file and line, an approval recommendation,
and a note on anything that was deliberately left off the PR. Reply yes and it posts as a single
review — line comments and the approval line in one notification.

### Review your whole queue

Skip the link to review every open PR waiting on you (GitHub's "Needs your review"):

```
/comment-on-pr review everything that needs my review
```

It finds the PRs, reviews them in parallel, and shows each draft as it finishes. One reply
("post all 8", or "post all but #123") posts the set. Before posting, it checks that no PR got
new commits after its review.

## What it will not do

- Post anything you haven't read.
- Pad the comment list to look thorough.
- Send `REQUEST_CHANGES` unless you explicitly ask for it.
- Write a summary paragraph on the review. The line comments carry the substance; the approval
  body is one short line.

## License

MIT — see [LICENSE](LICENSE).
