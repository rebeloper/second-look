---
name: second-look
description: Use when the user asks to review code, including local uncommitted or staged changes, a branch before commit or push, or a pull request or merge request on GitHub, GitLab, or Bitbucket by number or URL, or asks to work through, answer, reply to, or resolve review comments on a PR or MR.
---

# Second Look

Code review in chat that looks and feels like a GUI. Four modes, one visual language.

**REQUIRED:** Read `review-tiers.md` before reviewing, and `ui-kit.md` before writing any output. Read `security-checklists.md` sections only when the diff touches their area. Every screen you print is one of its templates, filled in. For PR review, Self-review, and Discussion, read the host file first (see **Host**). Local mode needs no host: skip host detection.

## Host

Find the host from the PR/MR URL if the user gave one, otherwise from `git remote get-url origin`:

| Remote contains | Read |
|---|---|
| `github.com` | `hosts/github.md` |
| `gitlab.com`, or a host where `glab auth status` succeeds | `hosts/gitlab.md` |
| `bitbucket.org` | `hosts/bitbucket.md` |

If it's still unclear, ask with a menu. Use the host's own words everywhere: "PR #12" on GitHub and Bitbucket, "MR !12" on GitLab. Every host file covers the same actions: read, post inline, suggestion format, verdict, read threads, reply/resolve, undo, and line link. If a host can't do something, leave it out of the menus.

## Pick the mode

Decide in this order. The first match wins.

1. **The user named the mode** ("reply to the comments", "review it as a reviewer", "just my local changes"): use it.
2. **No PR/MR mentioned:** **Local**. If the working tree is clean, review the branch against its base. If that's empty too, say there's nothing to review.
3. **A PR/MR is mentioned.** Get the user's username, the PR author, and the threads waiting on the user (host file, **Who am I**):
   - Any threads waiting on the user, on either side: **Discussion**.
   - Someone else's PR, nothing waiting: **PR review**.
   - The user's PR, nothing waiting: **Self-review**.

Show the pick and the reason on one line under the header, e.g. `🧭 Their PR · 2 replies waiting on you → Discussion`. If that's wrong, the user says so and you switch. If the host can't be reached to check the author, ask with a menu.

## Local

1. Collect: `git status --short`, `git diff HEAD`, untracked files (`git ls-files --others --exclude-standard`, read them). In a repo with no commits yet (`git rev-parse --verify HEAD` fails), use `git diff --cached` plus the untracked files instead. If the user asks for the whole repo, diff from the empty tree: `git diff $(git hash-object -t tree /dev/null) HEAD`. For a branch: `git diff <base>...HEAD`, where `<base>` is the one the user named, otherwise `git symbolic-ref --short refs/remotes/origin/HEAD`, otherwise `main`.
2. Review at the chosen tier (`review-tiers.md`, default 2).
3. Print the **Overview**, then the **Start menu**. If the user changes the tier there, re-review and show the new overview.
4. Walk through the findings: show one **Finding card** with its **Finding menu**, apply the fix if the user picks it, then move to the next card.
5. Show a **Done card**.

## PR review

1. Read the PR/MR and its diff (host file). Do not check out the branch unless the user asks.
2. Review at the chosen tier (`review-tiers.md`, default 2). Print the **PR header**, the **Overview**, and the **Start menu**. If the user changes the tier there, re-review and show the new overview.
3. Walk through the findings one **Finding card** at a time. Each pick in the **Finding menu** takes effect immediately: the comment is posted, copied, or dropped.
   - **Other reviewers:** read the existing threads and reviews first (host file, **Read threads** and **Reviews**). If a finding was already raised in an open thread, mark the card `👥 already raised by @x`, and the menu offers `👍 Agree in their thread` instead of `📤 Post`. Never post a duplicate.
4. Show the **Finish menu** to submit the overall verdict, then a **Done card**.

## Self-review

The user's own PR before anyone reviews it. Follow **PR review**, with these changes:
- The Finding menu is the Local one: 🔧 Fix · ⏭️ Skip · ⏹️ Stop here.
- Fixes go to the working tree, and only if the current branch is the PR's head branch. Otherwise, ask before checking it out.
- There is no Finish menu and nothing is posted. The Done card ends with: "Changes are uncommitted. Commit and push when ready."

## Discussion

Replying to PR/MR conversations that wait on the user, from either side. The role decides the take and the menu:

| | Author (the user's PR) | Reviewer (someone else's PR) |
|---|---|---|
| Waiting on the user | open threads from others where the last word isn't the user's | threads the user started or joined where someone replied after them, plus threads that @mention them |
| Take | ✅ valid · 🤔 partly · ❌ disagree | ✅ addressed · 🤔 partly · ❌ not addressed |
| Menu | see **Thread action bar** in `ui-kit.md` | see **Thread action bar** in `ui-kit.md` |

1. Fetch the threads (host file). Keep only the ones waiting on the user, unless they ask for all of them.
2. Print the **Inbox**. Blocking threads come first.
3. Walk through one **Thread card** at a time, then its **Thread action bar**. Before taking a side, check against the code as it is now, not against what anyone claims:
   - **Author:** check the comment against the code. Never agree because a reviewer said so, and never push back without a technical reason. If a comment is unclear, the draft reply asks a question.
   - **Reviewer:** check whether their reply or new commits actually address your point. Show what changed on that line since your comment.
4. **Author: commit per fix, send after the push.**
   - Before the first fix, check that the current branch is the PR's head branch. If it isn't, ask before switching. If the tree has uncommitted changes, ask whether to stash them, because they must never be mixed into fix commits. If the user agrees, run `git stash push -u -m "second-look: <PR>"` and remember it. At the end (after Push & send, Keep local, or Stop), run `git stash pop` for that entry and say so on the Done card. If the pop conflicts, stop and show the conflict without resolving it yourself.
   - 🔧 Fix applies the change and makes one commit with only the files that fix touched. The commit message is `fix: <short title>`, followed by a blank line and `Review: <thread url>`. If a commit hook fails, show its output and stop on that card.
   - Every pick queues the reply and resolve instead of posting them. This covers fixes, answers, and pushback alike.
   - "undo" before the push removes the last queued item. Only if that item made its own commit **and** `git rev-parse HEAD` is still that commit, also run `git reset --soft HEAD~1` (the change stays uncommitted). If the commit is shared with other queued replies, ask first, because those replies link to it. If HEAD moved, don't reset. Tell the user instead.
   - After the last card, show the **Send screen**. On `🚀 Push & send all`, run `git push` (never force). If the push is rejected, send nothing and explain why. Once it succeeds, post each queued reply with its commit link (host file, **Commit link**), then resolve the threads.
   - On `💾 Keep local`, save the queue to `$(git rev-parse --git-dir)/second-look/queue.json` (inside git's own folder, so never committed, and correct in worktrees). The next Discussion session on this PR starts by offering to send it.
5. **Reviewer:** replies and resolves post as soon as they're picked.
6. **Same ask, several threads** (author): fix it once with one commit, then queue a reply in each thread linking that commit. **Contradicting asks:** mark both threads ⚔️ and show both on one card, so the reply can address both reviewers.
7. **Reviewer, at the end:** if the author pushed commits after the user's last review, offer `🔍 Review new commits`: a **Re-review** (`review-tiers.md`) of just those commits (host file, **New commits since me**), with a verdict on each of the user's earlier findings, then the **Finish menu** to update the verdict.
8. Show a **Done card**: replied, resolved, fixed, committed, pushed, skipped.

## Rules

- Nothing reaches the host until the user picks Post or Reply on a card that shows the exact text. Posting is public, and "undo" removes the last post.
- Every finding has `file:line`, checked against the real file.
- Severity is one of: 🔴 Blocker, 🟠 Should fix, 🟡 Nit. Do not invent others.
- Posted and copied comments follow Conventional Comments (`issue (blocking): ...`, `nitpick (non-blocking): ...`). See **Comment labels** in `ui-kit.md`.
- Give fixes as `diff` blocks with the smallest change that works.
- Write replies in the user's voice: short and specific, with no "Great catch!" filler.
- You can't approve or request changes on your own PR/MR. Offer only 💬 Comment there.
