# GitHub (gh CLI)

## Setup

`gh` must be logged in (`gh auth status`). If it isn't, ask the user to run `gh auth login` in their terminal.
Get the repo with `gh repo view --json nameWithOwner -q .nameWithOwner` (or from the PR URL) and split it into `O`/`R`. `<N>` is the PR number.

Write every comment, reply, and summary body to a temp file and pass it with `-F body=@file`, so the shell never interprets it.

## Who am I

- Me: `gh api user -q .login`
- PR author: `.author.login` from **Read the PR**.
- Waiting on me:
  - **My PR:** unresolved `reviewThreads` started by others where the last comment isn't mine, plus conversation comments by others that I haven't answered.
  - **Their PR:** open threads I started or commented in where the last comment isn't mine, plus any thread whose latest comment @mentions me.
  Use **Read threads** for the data.

## Read the PR

```bash
gh pr view <N> -R "$O/$R" --json number,title,body,author,baseRefName,headRefName,headRefOid,additions,deletions,changedFiles,statusCheckRollup,url
gh pr diff <N> -R "$O/$R"
gh api "repos/$O/$R/contents/<path>?ref=<headRefOid>" -q .content | base64 -d   # full file, no checkout
```

## Post an inline comment

```bash
gh api "repos/$O/$R/pulls/<N>/comments" -F body=@comment.md -f commit_id=<headRefOid> \
  -f path=<path> -F line=<line> -f side=RIGHT -q '"\(.id) \(.html_url)"'
```

- Use `side=LEFT` for the old side. For a range, add `-F start_line=<first>`.
- `line` must be inside a diff hunk, or GitHub returns 422. Put any finding outside the hunks into the verdict summary instead.
- Keep the returned id for undo.
- **Suggestion:** a ```` ```suggestion ```` block holding the replacement for lines `start_line`–`line`. GitHub shows it as a one-click "Commit suggestion".

## Submit the verdict

```bash
gh api "repos/$O/$R/pulls/<N>/reviews" -f commit_id=<headRefOid> \
  -f event=<REQUEST_CHANGES|COMMENT|APPROVE> -F body=@summary.md -q .html_url
```

`REQUEST_CHANGES` and `COMMENT` need a body. On your own PR, only `COMMENT` works.

## Read threads

```bash
gh api graphql -F o="$O" -F r="$R" -F n=<N> -f query='
query($o:String!,$r:String!,$n:Int!){repository(owner:$o,name:$r){pullRequest(number:$n){
  reviewThreads(first:100){nodes{id isResolved isOutdated path line startLine diffSide
    comments(first:50){nodes{author{login} body url createdAt diffHunk originalCommit{abbreviatedOid}}}}}
  comments(first:100){nodes{author{login} body url}}}}}'
```

`reviewThreads` are line comments and can be resolved. `diffHunk` on the first comment feeds the Thread card's code view. `comments` are PR conversation comments and can only be replied to.

## Reply, resolve

```bash
# reply in a thread
gh api graphql -F t=<threadId> -F b=@reply.md -f query='
mutation($t:ID!,$b:String!){addPullRequestReviewThreadReply(input:{pullRequestReviewThreadId:$t,body:$b}){comment{url fullDatabaseId}}}'

# resolve / unresolve
gh api graphql -F t=<threadId> -f query='mutation($t:ID!){resolveReviewThread(input:{threadId:$t}){thread{isResolved}}}'
gh api graphql -F t=<threadId> -f query='mutation($t:ID!){unresolveReviewThread(input:{threadId:$t}){thread{isResolved}}}'

# reply to a PR conversation comment
gh api "repos/$O/$R/issues/<N>/comments" -F body=@reply.md -q '"\(.id) \(.html_url)"'
```

## Undo

```bash
gh api -X DELETE "repos/$O/$R/pulls/comments/<id>"     # inline comment or thread reply (fullDatabaseId)
gh api -X DELETE "repos/$O/$R/issues/comments/<id>"    # PR conversation comment
```

If the undone action also resolved a thread, unresolve it.

## New commits since me

```bash
OLD=$(gh api "repos/$O/$R/pulls/<N>/reviews" -q '[.[]|select(.user.login=="<me>")]|last|.commit_id')
gh api -H "Accept: application/vnd.github.diff" "repos/$O/$R/compare/$OLD...<headRefOid>"
```

If I never submitted a review, use the PR commit just before my first comment (`gh pr view <N> --json commits`). If `OLD` is no longer reachable because of a force-push, say so and offer a full review instead.

## Reviews

```bash
gh pr view <N> -R "$O/$R" --json latestReviews,reviewRequests \
  -q '.latestReviews[]|"\(.author.login) \(.state)"'      # APPROVED | CHANGES_REQUESTED | COMMENTED
```

## Commit link

`https://github.com/<O>/<R>/pull/<N>/commits/<sha>`

## Line link (Copy)

`https://github.com/<O>/<R>/pull/<N>/files#diff-<sha256 of path>R<line>`, using `L<line>` for the old side. Get the hash with `printf %s "<path>" | shasum -a 256`.
