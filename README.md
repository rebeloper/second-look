# Second Look

Code review in your AI coding CLI that feels like a GUI.

Second Look is an agent skill. It reviews your local changes before you commit. It reviews other people's pull requests, and you can post or copy each finding one card at a time. It also works through PR discussions with you, from either side of the review. Everything renders as cards, tables, and colored diffs right in the chat, with a menu under every step.

Works with **GitHub**, **GitLab**, and **Bitbucket Cloud**, and with any CLI that supports agent skills (Claude Code, GitHub Copilot CLI, and others).

## What it looks like

````
## 🔀 Cache user avatars on disk · #231
`feat/avatar-disk-cache` → `main` · `@dana` · 4 files · +96 −11 · ✅ passing
🧭 Their PR · no replies waiting → PR review

> ⛔ Not ready: 1 blocker · 2 should fix · 1 nit
🎚️ Tier 2 · Impact Review · 🌡️ Risk high · ⏱️ Effort 2/5

| # |    | Where                  | Issue                                     | Conf. |
|---|----|------------------------|-------------------------------------------|-------|
| 1 | 🔴 | `AvatarCache.swift:31` | Cached avatars collide across users       | 9     |
| 2 | 🟠 | `UserSession.swift:58` | Logout leaves previous user's avatars     | 8     |
| 3 | 🟠 | `AvatarCache.swift:30` | No size limit or expiry                   | 6 ⚠️  |
| 4 | 🟡 | `AvatarCache.swift:12` | Hard-coded `7` days                       | 8     |
````

Then one finding at a time, with the comment pinned to its line the way your Git host shows it:

```diff
@@ -28,6 +28,14 @@ final class AvatarCache
    28   28 ┃     private let directory: URL
+        30 ┃     func store(_ data: Data, for url: URL) throws {
+        31 ┃         let file = directory.appendingPathComponent(url.lastPathComponent)
+        32 ┃         try data.write(to: file, options: .atomic)
            ┃  ╭─ 🔴 @you · L31 · new side · Blocker · inline · 4e5f6a7
            ┃  │ issue (blocking): Cached avatars collide across users
            ┃  │
            ┃  │ Suggested change
            ┃  │ -        let file = directory.appendingPathComponent(url.lastPathComponent)
            ┃  │ +        let file = directory.appendingPathComponent(url.cacheFileName)
            ┃  ╰─
```

```
1/4 · What do you want to do with this finding?
  ❯ 📤 Post (Recommended)
    📋 Copy
    🗑️ Drop
    ⏹️ Stop here
```

## Modes

Second Look picks the mode from what you say and who wrote the PR. It shows its pick and the reason, so you can redirect it.

| Situation | Mode | A finding becomes |
|---|---|---|
| No PR mentioned | **Local review** | a fix applied to your files |
| Someone else's PR | **PR review** | a comment you post, copy, or drop |
| Your PR, no comments yet | **Self-review** | a fix in your branch, nothing posted |
| Any PR with threads waiting on you | **Discussion** | a reply, plus a fix commit if it's your PR |

**Discussion** works from both sides:

- **Your PR:** it checks each comment against the code before agreeing or pushing back, and makes one commit per fix. It queues the replies, pushes at the end, then posts every reply with a link to its commit.
- **Their PR:** it checks whether the author's reply or new commits actually fix what you asked. It shows that line then and now, and offers a review of only the new commits.

## Review tiers

| Tier | Name | Adds |
|---|---|---|
| 1 | **Diff Scan** | sure bugs in the changed lines |
| 2 | **Impact Review** (default) | intent vs the PR description (claims checked, not trusted), callers and other paths the PR didn't update, test coverage, security checklists for touched areas |
| 3 | **Verified Review** | focused passes (security, performance, compatibility…), failure modes, an independent check of each finding, variant search · up to 3 helper agents |
| 4 | **Full Audit** | module map, coverage checklist, parallel reviewers, focused tests in a clean environment (only with your OK) · up to 8 helper agents, asks first |

Say "quick look", "thorough review", or "audit", or switch tiers from the start menu. Every finding carries a confidence score. Comments follow [Conventional Comments](https://conventionalcomments.org) (`issue (blocking):`, `nitpick (non-blocking):`, …).

## Install

```bash
npx skills add rebeloper/second-look -g
```

This uses the open [skills](https://skills.sh) installer, which supports Claude Code, Codex, Cursor, and many more. It asks which agents to install for. Pick them up front with `-a`, e.g. `-a claude-code`. Leave out `-g` to install into the current project only.

Update with `npx skills update second-look`, and remove with `npx skills remove second-look`.

<details>
<summary>Manual install</summary>

```bash
git clone https://github.com/rebeloper/second-look.git
cd second-look
```

Link the skill folder into your CLI's skills directory:

| CLI | Command |
|---|---|
| Claude Code | `ln -s "$PWD/skills/second-look" ~/.claude/skills/second-look` |
| GitHub Copilot CLI | `ln -s "$PWD/skills/second-look" ~/.copilot/skills/second-look` |
| CLIs that read `~/.agents/skills` | `ln -s "$PWD/skills/second-look" ~/.agents/skills/second-look` |

Pull the repo to update.

</details>

### Requirements

- `git`
- **GitHub:** [`gh`](https://cli.github.com), logged in with `gh auth login`
- **GitLab:** [`glab`](https://gitlab.com/gitlab-org/cli), logged in with `glab auth login` (GitLab 17.2+)
- **Bitbucket Cloud:** `curl`, plus an Atlassian API token set in your shell as `BITBUCKET_EMAIL` and `BITBUCKET_API_TOKEN`
- `python3` (used for a few helper steps)

## Usage

```
review my local changes
quick look at my changes before I commit
review PR 231
thorough review of https://gitlab.com/acme/app/-/merge_requests/88
go through the comments on my PR 231
```

## Safety

- Nothing reaches your Git host until you pick **Post** or **Reply** on a card that shows the exact text. Type `undo` to remove your last post.
- Reviews never change your checkout. Fixes stay local unless you ask to commit, or, in Discussion mode on your own PR, until you choose **Push & send**.
- It never force-pushes. It runs someone else's PR code only with your explicit OK.

## License

[MIT](LICENSE)
