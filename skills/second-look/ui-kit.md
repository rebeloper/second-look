# Second Look UI kit

Every screen is one of these templates. Keep the order, headings, emoji, and separators exactly as shown and only fill in the slots marked `<...>`. Chat renders GitHub-flavored markdown, so headings, tables, blockquotes, and `diff` blocks are the visual system. The fences below show the raw markdown. Print each template as plain markdown, without the outer fence, so it renders.

**Tokens**

| Token | Use |
|---|---|
| 🔴 Blocker · 🟠 Should fix · 🟡 Nit | Finding severity |
| ⛔ Not ready · ⚠️ Fix first · ✅ Ready | Verdict |
| 💬 open · ✔️ resolved · 🕸️ outdated | Thread state |
| ✅ valid · 🤔 partly · ❌ disagree | Your take on a comment (author) |
| ✅ addressed · 🤔 partly · ❌ not addressed | Your take on a reply or fix (reviewer) |
| `---` | Separates every card |

## Comment labels (Conventional Comments)

Every comment and reply that gets posted or copied starts with a label line in the Conventional Comments format:

```
**<label> (<decoration>):** <subject>

<discussion>
```

| Finding | Posted label |
|---|---|
| 🔴 Blocker | `issue (blocking)`, or `todo (blocking)` for a small required chore |
| 🟠 Should fix | `issue (non-blocking)` for a defect, `suggestion (non-blocking)` for an improvement |
| 🟡 Nit | `nitpick (non-blocking)` |
| Something you need answered, or a deviation that may be intentional | `question` |
| Something done well | `praise`. Post at most one, and only if it's specific. |

Other labels can be used when they fit: `thought` for an idea that isn't a request, `chore` for process work such as a changelog entry, and `note` for information with no action needed. A small fix can take the `(if-minor)` decoration, meaning "fix it only if it's quick".

The emoji severity is for display in chat only. Posted text starts with the label line, not an emoji.

Replies use the same format. If the user's reply contains a question back to the reviewer, label it `question`. Otherwise, a plain reply needs no label.

In Discussion mode as the author, read the reviewer's label: `nitpick` and `(non-blocking)` mean the user may decline. `question` needs an answer, not a code change. `(blocking)` must be fixed or argued against before resolving.

---

## PR header

```
## 🔀 <PR title> · <#n on GitHub/Bitbucket, !n on GitLab>
🧭 <Their PR | Your PR> · <reason> → <PR review | Self-review | Discussion>
`<head>` → `<base>` · `@<author>` · <files> files · +<add> −<del> · <checks: ✅ passing | ❌ failing | ⏳ pending>

> <one-line summary of what the PR does, in your words>
```

## Overview

Local mode replaces the PR header with this header line:

```
## 🔍 Local changes · `<branch>`
🧭 <reason> → Local
<n> file(s) · +<add> −<del> · <uncommitted | staged | vs `<base>`> <· intent inferred, when there's no description>
```

Then:

````
> <⛔ **Not ready** | ⚠️ **Fix first** | ✅ **Ready**>: <n> blockers · <n> should fix · <n> nits
> <1–2 sentences: why this verdict>
🎚️ Tier <n> · <Diff Scan | Impact Review | Verified Review | Full Audit> · 🌡️ Risk <low | medium | high> <tier 2+: · ⏱️ Effort <1-5>/5>

<tier 3+: **What it does**
```diff
<smallest call / component / file tree sketch of the change>
```>

| | |
|---|---|
| 🚪 Merge danger | <one-way | two-way> door · blast radius: <1-2 words> |
| 🧭 Impact | <new symbols: `<symbol>` new · <n> uses. Changed symbols:> `<symbol>` <change> · <n> uses · <⚠️ n not updated | ✅ all updated | ⚠️ exported, outside callers unknown (when 0 uses in the repo)> |
| 🧪 Tests | <✅ covered | 🟡 partly: <gap> | ❌ not covered: <gap>> |
| 🗣️ Claims | <tier 2+, when the description, the user, or a code comment claims something: each claim → ✅ holds · ❌ contradicted by `<file>:<line>` · ⚠️ can't verify from the repo (e.g. manual testing)> |
| 👥 Reviewers | <PR modes, when others reviewed: `@a` ⛔ changes requested · `@b` ✅ approved · `@c` 💬 commented · <n> open threads> |
| 📋 Owner checks | <when a finding depends on a fact outside the repo: the fact → a safe check the owner can run> |
| 🎫 Ticket | <tier 3+, when a ticket, PR description, or requirements the user stated in chat exist: n met · n met but broken · n missing · n need a human> |
| 🗺️ Coverage | <tier 4: n files × n categories checked · n rejected · n need validation> |

| # | | Where | Issue | Conf. |
|---|---|---|---|---|
| 1 | 🔴 | `<file>:<line>` | <issue, under 60 chars> | 9 |
| 2 | 🟠 | `<file>:<line>` | <issue> | 6 ⚠️ |

<if any were hidden: 🙈 <n> more below the tier's confidence bar · a higher tier (`🎚️ Change tier`) shows them>

**Looks good:** <one specific line on what is solid>
````

Leave out the field table at tier 1, and any row that doesn't apply to the tier. At tier 3+, add after the table: **Failure modes** (a table: what · where · trigger · detected by), **Approach** when it applies, **Recommendations** (up to 3 bullets), **Declined to judge** (one line each, ending with `Name one to make it a finding.`), and **Split suggestion** when it applies. At tier 4, also add **Needs validation** (finding · the exact open question) and **Evidence** (what was run and the result). When verification was single-agent, say so on the tier line.

Number findings by severity: all 🔴 first, then 🟠, then 🟡. Within a severity, order by kind: security, then data loss, then crash or always-fails, then wrong result or a breaking API change, then a missed requirement, then everything else. Break ties by the earlier failure point (a crash on load comes before a crash on call), then by the larger blast radius. Mark unverified findings with ⚠️ in the Conf. column only, never in the Issue text. The overview has no finding details. Those come one at a time in the walkthrough.

Pluralize counts correctly ("1 file", "2 files"). Untracked files count as changed files, and all their lines count as additions.

Verdict rule: any 🔴 → ⛔ Not ready. Only 🟠 → ⚠️ Fix first. Only 🟡 or nothing → ✅ Ready.

If there are no findings, print only the header and `> ✅ **Ready**: nothing found`, followed by one line on what you checked. Skip the walkthrough. In PR review, go straight to the **Finish menu**.

## Menus

A menu is one single-choice question. Label each option with an emoji and a verb. Put the recommended option first and mark it "(Recommended)". Print `---` before the Start menu. Finding cards already open with one.

**Use the CLI's own question tool.** Most agent CLIs have a tool for asking the user a multiple-choice question, under different names. If one is available, use it for every menu. Fill in whatever it supports: a short title (`Start`, `<i>/<total>`, `Finish`), a description for each option, and a preview of what the option will do. Leave out what it doesn't support. If the tool limits the number of options, the menus here already fit within 4.

**If there is no such tool,** print a numbered list and end your turn. Accept the number, the label, or free text, and never move on without an answer:

```
**<title> · <question>**
1. 📤 Post (Recommended) · <preview, one line>
2. 📋 Copy · <preview, one line>
3. 🗑️ Drop
4. ⏹️ Stop here · Skips findings <i>–<total>
```

In the numbered form, a preview becomes one line after ` · `. Leave it out when the card above already shows the full text.

Free text always works: reword a comment, ask a question, or type "undo". Answer, then show the same menu again.

**Start menu** (after the overview). Title `Start`, question `How do you want to go through the <n> findings?`. `▶️ Walk through` is the recommended option. The `🔧 Fix all now` preview says how many fixes, which files, and which fixes will first ask for facts:
Local, Self-review: `▶️ Walk through` · `🔧 Fix all now` · `🎚️ Change tier` · `👋 Done`
PR review: `▶️ Walk through` · `🎚️ Change tier` · `🔒 Keep private`

The `🎚️ Change tier` preview shows the current tier. Picking it opens the **Tier menu**.

**Tier menu:** all four tiers as options, in order. Mark the current one "(current)" and recommend none. Each option's description says what the tier adds and its rough cost:

```
**Tier · Which depth?**
1. 🎚️ 1 · Diff Scan · sure bugs in the changed lines · ~1 min
2. 🎚️ 2 · Impact Review (current) · + intent and claims, callers and other paths, tests, security checklists · a few min
3. 🎚️ 3 · Verified Review · + focused passes, failure modes, an independent check of each finding, variant search · longer, up to 3 agents
4. 🎚️ 4 · Full Audit · + module map, coverage checklist, parallel helper agents, a test run · longest, most expensive, up to 8 agents, asks first
```

- **Higher tier:** re-review, keeping the findings already made. Only the steps the new tier adds run again.
- **Lower tier:** don't re-review. Re-filter the existing findings with that tier's confidence bar, nit rule, and cap.
- **Current tier:** close the menu.

In all three cases, print the new overview and the Start menu again.

`Fix all now` skips the walkthrough. If any fix needs facts only the user has (e.g. where a missing module lives), ask for all of them in one question first. Fixes the user can't answer for are skipped and listed on the Done card.

## Code view

Code is shown the way a host's diff view shows it, with line numbers and inline pinned comments. The fence is ```` ```diff ```` so the terminal colors `+` lines green and `-` lines red. The color comes from each line's first character, so the sign always comes first.

````
```diff
@@ -38,9 +38,15 @@ final class ProfileViewModel
    38   38 ┃     func load() {
-   39      ┃         api.fetchProfile(id: userID) { result in
+        39 ┃         isLoading = true
+        40 ┃         api.fetchProfile(id: userID) { result in
+        41 ┃             self.timer = Timer.scheduledTimer(withTimeInterval: 30, repeats: true) { _ in
            ┃  ╭─ 🔴 @you · L41 · new side · Blocker · inline · a1b2c3d
            ┃  │ issue (blocking): Retain cycle in the refresh timer
            ┃  │
            ┃  │ The timer's closure captures self strongly.
            ┃  │
            ┃  │ Suggested change
            ┃  │ -            self.timer = ... { _ in
            ┃  │ +            self.timer = ... { [weak self] _ in
            ┃  ╰─
+        42 ┃                 self.refresh()
```
````

- **Columns:** each line is the sign (`+`, `-`, or a space), a space, the old line number right-aligned in 4 characters, a space, the new line number right-aligned in 4, then ` ┃ ` (heavy bar) and the code exactly as written. Removed lines leave the new number blank, and added lines leave the old number blank.
- **Hunk header:** the first line is the raw `@@ ... @@ <enclosing type or func>` header, starting in column 0.
- **Range:** show 5-15 lines around the finding. Keep every `+`/`-` line near it. Where you cut far context, add a trim row: `     ⋯    ⋯ ┃`, with `⋯` in both number columns. A blank code line ends at `┃`, with no trailing space.
- **Pinned comments:** a comment box goes directly under the line it is pinned to. For a range, that is the last line of the range, as all three hosts do. A comment on the old side goes under the `-` line.
- **Comment box lines:** these start with 12 spaces and then `┃  `, so the `┃` lines up with the code column and the box stays uncolored. The box has a left border only: `╭─` header, `│ ` body, `╰─` close. Never add a right border, because emoji widths would misalign it.
  - The header line is `<severity> @<who> · L<start>[–<end>] · <new | old> side · <severity word> · <inline> · <sha7>`. In Thread cards, use `@<who> · <relative time>` instead.
  - Wrap body text at 72 characters. Markdown does not render inside the fence, so write plain text with no `**` or backticks.
  - A suggestion is the line `Suggested change`, followed by `-` and `+` lines.
- **Usernames:** outside a fence, write every username as inline code, `` `@<name>` ``, so it shows in the theme's code color. Inside a fence, write it as plain `@<name>`.
- **Existing threads:** in every code view, show open threads by others pinned to their lines, in the same box format, so nobody repeats a point already made. Collapse a long thread to its first and last comment plus `│ … <n> more`.
- **Threads:** put every comment in one box, oldest first. Separate replies with `├─ @<who> · <time>`. The draft reply goes last, after `├┄ ✏️ draft · @<you> · <✔️ will resolve | stays open>`, and its body uses `┆ ` instead of `│ `.

## Finding card

Show one card at a time, then the finding menu. Wait for the answer before showing the next card.

````
---

### 🔴 <i>/<total> · <short title>
`<file>:<line>` · <added | changed | removed> in this <PR | change> · confidence <n>/10 <tier 3+: · <✅ verified | ⚠️ needs validation: <open question>>>

<Code view of the hunk. In PR review mode, the comment box sits inside it, under the pinned line.>

<1-3 sentences: what breaks, for which input, and why.>
<If the finding or its fix rests on something you could not check, end with `⚠️ Unverified: <assumption>` (e.g. the DB driver's placeholder syntax).>

<Local mode:>
**Fix**
```diff
-<current line>
+<fixed line>
```

<PR review mode: if the finding is outside every diff hunk, it can't be pinned. Mark it 📝 PR-level and show the comment after the code view as a blockquote with the same metadata line. 📤 Post posts it right away as a PR-level comment (host file, conversation comment).>
````

The box shows a suggestion as `Suggested change`. The posted body uses the host's suggestion format (see the host file). Include a suggestion only when the fix replaces exactly the pinned lines. Otherwise describe the fix in the comment text.

If no local diff fits (a missing file, or a change needed in callers elsewhere), write `**Fix:** <one sentence>` instead of the Fix diff. The `🔧 Fix` preview then shows that sentence. If the fix needs facts only the user has, such as where a module lives, picking Fix asks for them first.

**Finding menu.** Each option's preview shows exactly what will happen:
Local: `🔧 Fix` (preview: the diff) · `⏭️ Skip` · `⏹️ Stop here`
PR review: `📤 Post` (preview: the exact comment, with the suggestion as it will post) · `📋 Copy` · `🗑️ Drop` · `⏹️ Stop here`

Recommend Fix or Post for 🔴 and 🟠. Don't recommend anything for 🟡. The `⏹️ Stop here` preview reads `Skips findings <i>–<total>`.

Act as soon as the user picks, confirm with one line, then show the next card:
- 🔧 Fix → apply it, then `✔️ Fixed <file>:<line>`
- 📤 Post → post this one comment now (host file), then `✔️ Posted <file>:<line> · <comment url>`
- 📋 Copy → see **Copy line** below.
- `Stop here` ends the walkthrough. The current finding and the rest count as skipped.

**Undo:** undo only reaches back one action at a time.
- After 📤 Post: delete that comment (host file). If the action also resolved a thread, unresolve it. Confirm with `↩️ Removed <file>:<line>`.
- After 🔧 Fix (Local, Self-review): reverse exactly the lines that fix changed, and leave every other line in the file alone. Never run `git checkout`, `git restore`, or `git stash` on the file, because it holds the user's own uncommitted work. Confirm with `↩️ Reverted <file>:<line>`.
- For the Discussion author flow, see `SKILL.md`.

**Finish menu** (PR review only: after the last card, or right after the overview when there are no findings): `⛔ Request changes` · `💬 Comment` · `✅ Approve` · `👋 Leave as is`. Recommend `Request changes` if a 🔴 was posted, `Approve` if nothing was posted, otherwise `Comment`. Before the menu, print the review summary that will post with it, in full, as a blockquote under `**Review summary**`, so the user sees the exact text. This submits the overall verdict. The inline comments are already posted.

## Copy line

Copy the exact text that would have been posted, including any suggestion in the host's format, then print:

```
📋 Copied · paste at `<file>` line <L>[–<end>] · <new | old> side
🔗 <link to that line in the PR>
```

- Clipboard command, picked by OS: macOS `pbcopy`, Windows `clip.exe`, Linux `wl-copy`, or `xclip -selection clipboard` if that's missing. Write the text to a temp file first and pipe the file in (`pbcopy < file`), so the shell never interprets it.
- If no clipboard command works, print the text in a fenced block for manual copying and say so.
- Link: from the host file's **Line link** section.
- In Discussion mode, copy the reply and link to the thread's comment URL instead.

## Inbox

```
## 💬 Discussion · <#n | !n> <PR title>
🧭 <Your PR · you're the author | Their PR · you're the reviewer> · <n> waiting on you <reviewer: · <n> new commits since your review>

| # | | Who | Where | Latest | State |
|---|---|---|---|---|---|
| 1 | 🔴 | `@<who>` | `<file>:<line>` | "<latest message, under 60 chars>" | <author: open | reviewer: 🔧 pushed a fix | 💬 answered | ↩️ pushed back> |
| 2 | 💬 | `@<who>` | `<file>:<line>` | "..." | ... |
| 3 | 💬 | `@<who>` | PR thread | "..." | ... |
```

Keep "Latest" under 60 chars, and start it with the label if one was used (e.g. `question: why not…`). The second column is 🔴 for `(blocking)` threads, 🕸️ for outdated ones, and 💬 otherwise. Order: blocking first, then the rest by age, then PR-level comments.

## Thread card

````
---

### 💬 <i>/<total> · `@<who>` on `<file>:<line>`
<💬 open | 🕸️ outdated> · <n> comment(s) · 🔗 `<commit the thread is on, 7 chars>`

<Code view of the comment's diffHunk, with the whole thread and the draft reply inside it, under the pinned line.>

<Reviewer, if the code at that line changed since your comment, add:>
**Since your comment** · `<old sha7>` → `<head sha7>`
```diff
<the same lines then vs now, in the code view format>
```

**Take:** <author: ✅ valid | 🤔 partly | ❌ disagree · reviewer: ✅ addressed | 🤔 partly | ❌ not addressed>: <one sentence on why. Mention the label if it changes the answer, e.g. "non-blocking nit, fine to decline".>

<Author, if the take is valid or partly, add:>
**Fix**
```diff
-<current line>
+<fixed line>
```
````

If the thread is outdated, say so on the status line, and diff the old hunk against the current file before taking a position.

## Thread action bar

A menu (see **Menus**). Each preview shows exactly what will happen (the reply text, or the fix diff plus the reply):

Author: `🔧 Fix + reply + resolve` · `💬 Reply` · `📋 Copy reply` · `⏭️ Skip`
Reviewer: `✔️ Accept + resolve` · `💬 Reply` · `📋 Copy reply` · `⏭️ Skip`

Recommend Fix when the author's take is valid, and Accept when the reviewer's take is addressed. Otherwise recommend Reply. As a reviewer, ✅ addressed means the defect can no longer happen. "Attempted" doesn't count, and 🤔 partly is treated as not addressed: never recommend Accept for it. As a reviewer, Accept posts a short reply (e.g. "Looks good now, thanks.") only if the user keeps the draft. They can clear it to resolve silently.

Reviewer: act on the pick right away. Replies and resolves post immediately, followed by `✔️ Replied <file>:<line> · <url>`.
Author: a fix commits right away, followed by `✔️ Committed <sha7> · fix: <title>`. The reply is queued (`🕓 Queued reply · <file>:<line>`) and posts after the push, from the **Send screen**. Copy reply follows **Copy line**. With free text, the user can rewrite the reply (use their wording as given), say "resolve only", or say "undo".

## Send screen

Author side of Discussion, after the last card:

```
## 🚀 Ready to push · <#n | !n>
<n> commits · <n> replies · <n> resolves · branch `<head>` → `origin`

| # | Commit | Thread | Reply |
|---|---|---|---|
| 1 | `a1b2c3d` fix: guard bad avatar URL | `ImageLoader.swift:22` · `@maria` | "Restored the guard; throws .badURL now." |
| 2 | (none) | `ProfileView.swift:8` · `@tom` | "question: would a subview help here? …" |
```

Below the table, show each reply in full as a blockquote, including its `Fixed in <sha7>` line, so the user sees the exact text before sending. The menu is `🚀 Push & send all` (Recommended) · `✏️ Edit queue` · `💾 Keep local`. `✏️ Edit queue` reopens any card the user names. Replies post as `<reply text>` followed by a blank line and `Fixed in [<sha7>](<commit link>)` for fix replies.

## Done card

If nothing was fixed, posted, or copied, skip the card and print one line: `👋 Nothing changed.`

```
---
## ✅ Done
<🔧 n fixed> · <📦 n committed> · <🚀 pushed> · <📤 n posted> · <📋 n copied> · <💬 n replied> · <✔️ n resolved> · <⏭️ n skipped>
<Leave out any count that is 0 or doesn't apply to the mode.>
<URL(s) if posted>

<If files changed: "Changes are uncommitted: `git diff` to see them.">
```
