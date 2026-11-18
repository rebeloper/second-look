# Bitbucket Cloud (REST API + curl)

Bitbucket has no official CLI, so these calls go through `curl`. This file covers Bitbucket Cloud (bitbucket.org) only. Bitbucket Data Center has a different API and isn't supported.

## Setup

- **Credentials:** the user needs an Atlassian API token with scopes for reading repositories and for reading and writing pull requests. Ask them to export the email and token in their own shell, never to paste them into the chat:
  ```bash
  export BITBUCKET_EMAIL=<atlassian account email>
  export BITBUCKET_API_TOKEN=<token>
  ```
  If either variable is missing, stop and ask for it.
- **Never print, log, or echo the token.** The helper passes it to curl on stdin (`-K -`) through the shell's built-in `printf`, so it never appears in the process list.
- **Base URL:** `BB=https://api.bitbucket.org/2.0/repositories/<workspace>/<repo>/pullrequests/<N>`. Read `<workspace>/<repo>` from the remote URL or the PR URL.
- **Request helper:**
  ```bash
  bb() { printf 'user = "%s:%s"\n' "$BITBUCKET_EMAIL" "$BITBUCKET_API_TOKEN" \
           | curl -sS --fail-with-body -K - -H 'Content-Type: application/json' "$@"; }
  ```
- **Bodies:** build each JSON body in a temp file with `python3 -c 'import json,sys; ...'` or `jq`, and send it with `--data @file`. Never splice text into JSON by hand.

## Who am I

- Me: `bb https://api.bitbucket.org/2.0/user` and take `.account_id`.
- PR author: `.author.account_id` from **Read the PR**. Compare account ids, not display names.
- Waiting on me:
  - **My PR:** threads (root comment with no `resolution`) started by others where the last comment isn't mine, plus conversation comments by others that I haven't answered.
  - **Their PR:** open threads I started or commented in where the last comment isn't mine, plus any thread whose latest comment @mentions me.
  Use **Read threads** for the data. Mentions appear in `content.raw` as `@{<account_id>}`.

## Read the PR

```bash
bb "$BB"                 # title, description, author.display_name, source.branch.name, destination.branch.name,
                         # source.commit.hash, destination.commit.hash, links.html.href
bb -L "$BB/diff"         # unified diff (redirects, so -L is needed)
bb -L "https://api.bitbucket.org/2.0/repositories/<workspace>/<repo>/src/<source hash>/<path>"   # full file
```

Bitbucket shows usernames as display names. Use `author.nickname` if it's there, otherwise `display_name`.

## Post an inline comment

```json
{ "content": { "raw": "<comment markdown>" },
  "inline": { "path": "<path>", "to": <new line> } }
```

```bash
bb -X POST "$BB/comments" --data @comment.json     # keep .id and .links.html.href
```

- Old side: use `"from": <old line>` instead of `"to"`. Don't send both, because Bitbucket then ignores `to`.
- Range: add `"start_to": <first new line>` (or `"start_from"` on the old side). The `to`/`from` value is the last line.
- **Suggestion:** Bitbucket has no one-click suggestion block. Post the fix as a normal fenced code block under the line `Suggested change:`.

## Submit the verdict

| Pick | Call |
|---|---|
| ✅ Approve | `bb -X POST "$BB/approve"` |
| ⛔ Request changes | `bb -X POST "$BB/request-changes"`, then post the summary as a comment |
| 💬 Comment | `bb -X POST "$BB/comments" --data @summary.json` (`{"content":{"raw":"..."}}`) |

The PR author can't approve or request changes on their own PR. Offer only 💬 Comment.

## Read threads

```bash
bb "$BB/comments?pagelen=100"       # follow .next until it is absent
```

- Each comment has `id`, `content.raw`, `user`, `created_on`, `deleted`, `inline` (`path`, `to`, `from`, `start_to`, `start_from`), `parent.id` for replies, and `resolution` (present when resolved).
- Group comments into threads by following `parent.id` up to the root.
- Skip comments with `deleted: true`.
- A root without `inline` is a conversation comment.
- No diff hunk comes back. Build the Thread card's code view from `$BB/diff` around the line. If the line is no longer in the diff, mark the thread 🕸️ outdated.

## Reply, resolve

```bash
bb -X POST   "$BB/comments" --data @reply.json        # {"content":{"raw":"..."},"parent":{"id":<root id>}}
bb -X POST   "$BB/comments/<root id>/resolve"         # resolve thread
bb -X DELETE "$BB/comments/<root id>/resolve"         # reopen thread
```

## Undo

```bash
bb -X DELETE "$BB/comments/<id>"
```

If the undone action also resolved a thread, reopen it.

## New commits since me

```bash
bb "$BB/commits"            # newest first, each with .hash and .date
```

`OLD` is the newest commit dated before my last comment. Then get the diff. Bitbucket orders the range as **new first**, the reverse of `git diff`:

```bash
bb -L "https://api.bitbucket.org/2.0/repositories/<workspace>/<repo>/diff/<source hash>..$OLD?topic=false"
```

If I never commented, offer a full review instead.

## Reviews

From **Read the PR**: `.participants[]` has `user`, `role` (`REVIEWER` | `PARTICIPANT`), `approved`, and `state` (`approved` | `changes_requested` | null).

## Commit link

`https://bitbucket.org/<workspace>/<repo>/commits/<sha>`

## Line link (Copy)

`https://bitbucket.org/<workspace>/<repo>/src/<source hash>/<path>#lines-<line>`, using the destination hash for the old side. Also give the PR's diff tab: `<links.html.href>/diff`.
