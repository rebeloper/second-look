# GitLab (glab CLI)

Works with gitlab.com and self-managed GitLab 17.2 or later. Earlier versions can't request changes.

## Setup

`glab` must be logged in (`glab auth status`). If it isn't, ask the user to run `glab auth login` in their terminal (add `--hostname <host>` for self-managed).
Inside the repo, `glab api` replaces `:id` with the current project. For an MR in another repo, use the URL-encoded path instead of `:id`, e.g. `group%2Fsub%2Frepo`, and add `--hostname <host>` for self-managed.
`<N>` is the MR IID, the number shown in the UI. Below, `$MR` stands for `projects/:id/merge_requests/<N>`.

Write every body to a temp file and pass it with `-F body=@file` (or `-F note=@file`), so the shell never interprets it. `-F` switches the method to POST, so always set `-X` explicitly.

## Who am I

- Me: `glab api user` and take `.username`.
- MR author: `.author.username` from **Read the MR**.
- Waiting on me:
  - **My PR:** discussions with `resolvable: true` and `resolved: false` started by others where the last comment isn't mine, plus conversation comments by others that I haven't answered.
  - **Their PR:** open threads I started or commented in where the last comment isn't mine, plus any thread whose latest comment @mentions me.
  Use **Read threads** for the data.

## Read the MR

```bash
glab api "$MR"              # title, description, author.username, source_branch, target_branch, web_url, diff_refs{base_sha,start_sha,head_sha}
glab mr diff <N> --raw      # unified diff
glab api "projects/:id/repository/files/<url-encoded path>/raw?ref=<head_sha>"   # full file
```

Keep `diff_refs`, because every inline comment needs all three SHAs.

## Post an inline comment

Each inline comment opens a new discussion:

```bash
glab api -X POST "$MR/discussions" -F body=@comment.md \
  -f "position[position_type]=text" -f "position[base_sha]=<base_sha>" \
  -f "position[start_sha]=<start_sha>" -f "position[head_sha]=<head_sha>" \
  -f "position[old_path]=<path>" -f "position[new_path]=<path>" \
  -f "position[new_line]=<line>"
```

- Added line: send `position[new_line]` only. Removed line: send `position[old_line]` only. Unchanged line: send both.
- Range: add `position[line_range][start][line_code]` and `position[line_range][end][line_code]` (format `<sha1 of path>_<old_line>_<new_line>`), each with `[type]` = `new` or `old`. If that fails, pin the comment to the last line and mention the range in the text.
- The response holds `id` (the discussion id) and `notes[0].id`. Keep both for undo.
- **Suggestion:** ```` ```suggestion:-<above>+<below> ```` followed by the replacement lines. The offsets count lines above and below the commented line, and `-0+0` replaces just that line. GitLab shows an "Apply suggestion" button.

## Submit the verdict

| Pick | Call |
|---|---|
| ✅ Approve | `glab api -X POST "$MR/approve"` |
| ⛔ Request changes | `glab api -X POST "$MR/draft_notes" -F note=@summary.md`, then `glab api -X POST "$MR/draft_notes/bulk_publish" -f reviewer_state=requested_changes` |
| 💬 Comment | `glab api -X POST "$MR/notes" -F body=@summary.md` |

- `bulk_publish` publishes **all** of the user's pending drafts on this MR, including ones started in the web UI. Before Request changes, run `glab api "$MR/draft_notes"`. If any drafts exist besides the summary, list them and ask: publish them too, or post the summary as 💬 Comment and let the user request changes in the web UI.
- If the user lacks approval rights, GitLab records `reviewed` instead of `requested_changes`. Tell them.
- The MR author can't approve their own MR unless the project allows it.

## Read threads

```bash
glab api --paginate "$MR/discussions?per_page=100"
```

- Each discussion has `id` and `notes[]`.
- Each note has `id`, `body`, `author.username`, `created_at`, `resolvable`, `resolved`, `system`, and `position` (`new_path`, `old_path`, `new_line`, `old_line`, `head_sha`).
- Skip notes with `system: true`.
- A discussion with no `position` is a conversation comment.
- The response has no diff hunk. Build the Thread card's code view from `glab mr diff <N> --raw` around the note's line. If that line is no longer in the diff, mark the thread 🕸️ outdated.

## Reply, resolve

```bash
glab api -X POST "$MR/discussions/<discussion_id>/notes" -F body=@reply.md      # reply (keep note id)
glab api -X PUT  "$MR/discussions/<discussion_id>" -f resolved=true              # resolve
glab api -X PUT  "$MR/discussions/<discussion_id>" -f resolved=false             # unresolve
```

## Undo

```bash
glab api -X DELETE "$MR/discussions/<discussion_id>/notes/<note_id>"
```

Deleting a discussion's only note removes the whole discussion. If the undone action also resolved a thread, unresolve it.

## New commits since me

```bash
glab api "$MR/versions"     # each version: head_commit_sha, created_at (newest first)
```

`OLD` is the `head_commit_sha` of the newest version created before my last note. Then:

```bash
glab api "projects/:id/repository/compare?from=$OLD&to=<head_sha>"   # .diffs[] per file, .commits[]
```

If I never commented, offer a full review instead.

## Reviews

```bash
glab api "$MR/reviewers"    # [].user.username, [].state: unreviewed | review_started | reviewed | requested_changes | approved
glab api "$MR/approvals"    # .approved_by[].user.username
```

## Commit link

`<project web_url>/-/commit/<sha>`

## Line link (Copy)

`<project web_url>/-/blob/<head_sha>/<path>#L<line>`, using `base_sha` for the old side. Also give the MR's changes tab: `<MR web_url>/diffs`.
