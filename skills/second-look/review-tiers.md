# Review tiers

How findings are produced. `ui-kit.md` decides how they look. Each tier includes everything in the tiers below it.

## Pick the tier

| User says | Tier |
|---|---|
| "quick look", "skim", "scan", "tier 1" | **1 · Diff Scan**: sure bugs in the changed lines |
| nothing specific, "review", "impact", "tier 2" | **2 · Impact Review** (default): + intent, callers, tests |
| "thorough", "deep", "careful", "verify", "tier 3" | **3 · Verified Review**: + focused passes, failure modes, an independent check of each finding |
| "audit", "security review", "before release", "tier 4" | **4 · Full Audit**: + module map, coverage checklist, parallel helper agents, a test run |

Show the tier in the overview header. The Start menu's `🎚️ Change tier` lets the user switch to any tier (see **Tier menu** in `ui-kit.md`).

## What each tier does

| Step | 1 | 2 | 3 | 4 |
|---|:-:|:-:|:-:|:-:|
| A. Scope: diff, PR description, linked ticket | ✔ | ✔ | ✔ | ✔ |
| B. Context: read around every hunk | ±20 lines | whole functions and types | + their callers | + whole touched modules |
| C. Bug hunt (core checklist) | ✔ | ✔ | ✔ | ✔ |
| D. Intent: does it do what the PR says? | | ✔ | ✔ | ✔ |
| E. Impact: callers, conformers, missing pieces | | ✔ | ✔ | ✔ |
| F. Tests: is the new behavior covered? | | ✔ | ✔ | ✔ |
| G. Self-check each finding | ✔ | ✔ | | |
| H. Category passes | | | ✔ | ✔ |
| I. Failure modes | | | ✔ | ✔ |
| J. Fresh verifier per finding | | | ✔ | ✔ |
| K. Recon of touched modules | | | | ✔ |
| L. Coverage ledger + critic pass | | | | ✔ |
| M. Parallel hunters per area | | | | ✔ |
| N. Second verifier on final claims | | | | ✔ |
| O. Evidence run: build and tests (with permission) | | | | ✔ |

**Which findings are shown** (confidence scale below):

| | 1 | 2 | 3 | 4 |
|---|---|---|---|---|
| Show | ≥ 8 | ≥ 7 | ≥ 5 | all confirmed |
| As `⚠️ Unverified` | none | 5–6 | 3–4 | needs-validation list |
| Nits | none | up to 3 | all | all |
| Cap | 5 findings | none | none | none |

**Overview adds:**

| | 1 | 2 | 3 | 4 |
|---|---|---|---|---|
| Verdict, risk | ✔ | ✔ | ✔ | ✔ |
| Effort, merge danger, impact, tests | | ✔ | ✔ | ✔ |
| What it does (visual), ticket check, failure modes, declined to judge, split suggestion | | | ✔ | ✔ |
| Coverage, needs validation, evidence | | | | ✔ |

## The steps

**A. Scope.** Read the diff, the PR/MR description, and any linked issue or ticket. These state the intent. In Local mode there is usually no description: infer the intent from the branch name, recent commit messages, and the diff, and write "intent inferred" in the overview. If the diff is over ~1000 changed lines, review it in passes by area and say so in the overview.

**B. Context.** Never judge a hunk you only saw in the diff. Read the surrounding code at the depth the tier sets. Beyond that, read code outside the diff only to check a risk you can name (e.g. "does any caller still pass a sync handler?"). Do one focused check per named risk, and record it as `risk → what you read (file:line)`. Findings, the Looks good line, and Declined to judge cite these checks.

**C. Bug hunt.** Look for things that break in practice:
- Logic: off-by-one, inverted condition, wrong operator, unreachable branch.
- Nil and optionals: force unwraps, missing guards, empty collections.
- Errors: swallowed errors, `try?` hiding failures, missing failure path.
- Concurrency: races, wrong thread or actor for UI, retain cycles, missing cancellation.
- Resources: leaks, unclosed handles, timers and observers never removed.
- Security basics: injection, secrets in code, missing auth check, unsafe deserialization.
- Data: loss, corruption, irreversible writes, migrations without a path back.
- Wiring (cheap, so every tier checks it): new imports, requires, and asset or resource references resolve to files that exist.

**D. Intent.** Compare the code with what the PR says it does:
- **Missing:** a requirement skipped, or claimed but not built. If the description or ticket lists files or changes, each one must show up in the diff. A listed item with no hunk is Missing.
- **Extra:** features nobody asked for, over-engineering, unrequested "nice to haves".
- **Misunderstood:** the right feature built the wrong way, or the wrong problem solved.

Treat the PR description, commit messages, and the author's claims ("tested on device", "no behavior change", "kept it simple on purpose") as unverified. Check them against the code. A stated reason never lowers a severity. If the ticket or description asks for something this checklist calls a defect, it's still a finding, labeled `as requested`, and the user decides. Where the description says nothing, judge by what a reasonable user of the software would expect: silence is not permission. If a requirement can't be checked from the diff (it lives in unchanged code), list it as `⚠️ can't verify from the diff` instead of widening the search. That marker is only for requirements and claims. A finding that rests on something missing or unseen keeps a confidence score and uses `⚠️ Unverified`. Report every claim's result in the overview's 🗣️ Claims row.

**E. Impact.** For every changed signature, type, protocol, public API, config key, or schema, search the codebase for its uses. Flag uses the PR didn't update, files it depends on that don't exist, and broken backward compatibility. For iOS, also check protocol conformers, `public` and `open` API, `@MainActor` and `Sendable` changes, `Codable` keys, and SwiftUI initializers.

**F. Tests.** Is each new or changed behavior covered by a test that checks real behavior rather than a mock? Name the untested path. Never claim a test passed unless you ran it. Test and CI output must be clean: new warnings, flaky retries, and newly skipped tests are findings. If a log is truncated or missing, say "couldn't see it". Never read a missing log as a pass.

**G. Self-check.** Before keeping a finding, re-read the code and try once to prove it wrong. Drop it if you can.

**H. Category passes.** One focused pass per category that applies to the touched code: security (pick the classes that match, e.g. auth, input handling, storage, network, IPC and deep links), performance, API and backward compatibility, error handling, maintainability (duplication, premature abstraction, YAGNI), structure (each file has one clear job; did this change create a very large new file or grow an existing one a lot? Judge only what this change added), and docs.

**I. Failure modes.** For each risky change: what fails, where (`file:function`), the realistic trigger, and what would detect it (a named test or signal, or "nothing"). Note whether this PR covers it.

**J. Fresh verifier.** Give each finding to a fresh agent that tries to disprove it. Pass only the diff, the file paths, and the finding, never your reasoning. The verdict is one of:
- **confirmed:** the trace is complete.
- **needs validation:** state the exact unresolved fact.
- **rejected:** dropped, and counted in the overview.

The agent that found a finding never verifies it.

**K. Recon.** Before hunting, map the touched modules: what they own, where outside input enters, and which trust boundaries the change crosses.

**L. Coverage ledger.** List every changed file against every category from H, and mark each cell checked, not applicable, or not done. A critic pass then looks for gaps. Fill the gaps before reporting.

**M. Parallel hunters.** Split the ledger by area and give each area to its own fresh agent.

**N. Second verifier.** A different fresh agent re-checks every final `file:line` claim and quoted line against the files.

**O. Evidence run.** Build, then run the focused tests that answer a specific doubt from the review, and show the before and after. Don't run whole suites, race detectors, or repeat loops. Recommend those in the overview instead. Running PR code is running someone else's code: ask first, and prefer a sandbox with no network. For a PR you didn't write, never run it without an explicit yes.

## Rules for every tier

- **Read-only.** Never change the checkout while reviewing. To see another revision, use a separate worktree in a temp directory.
- **Fresh agents:** if the CLI can start subagents, use them for J, M, and N. If it can't, do those steps as separate passes: re-read the code from scratch, argue the opposite case, and say in the overview that verification was single-agent.
- **Don't review your own code.** If this session wrote or edited the code under review (e.g. you implemented it earlier in the conversation), run steps C–I in a fresh agent that gets only the review package. You then only verify, filter, and present. This applies at every tier. Without subagents, do it as a separate pass and put `self-authored · single-agent` on the tier line.
- **Review package** (whenever a helper agent is used): write it once to a temp file. It holds the PR description or inferred intent, the commit list, a stat summary, and the full diff with generous context (`git diff -U25`, or the host diff). Every helper gets the same file and doesn't re-run git or host fetches.
- **Helpers don't start helpers.** A helper agent does its whole job itself and never starts more agents. If its share is too large, it works in passes and says so. Its report is only verdicts, findings with `file:line`, and checks it ran, with no narration.
- **Confidence (0–10) on every finding:**
  - 9–10: verified in the code, with a concrete trigger.
  - 7–8: very likely, and relies only on reasonable assumptions about code outside the diff.
  - 5–6: plausible but unverified.
  - 3–4: weak. Shown only at tier 3, as `⚠️ Unverified`, after a fresh verifier failed to reject it.
  - Below 3: never report.
- **Severity is likelihood × impact,** not "breaks a rule". A missing second safeguard is 🟡 hardening when the first one holds. Not everything is a blocker.
- **Severity anchors:** 🟠 should fix includes tests that assert nothing, copy-pasted logic blocks, swallowed errors, and a missed requirement. "Coverage could be broader" and polish are 🟡.
- **A certain failure is 🔴,** even if it looks like unfinished work, such as a missing file that fails on load or code that throws on every call. Say "may be unfinished" in the text.
- **Fixes:** if a change of 10 lines or less fixes it, give the exact current lines and their replacement.
- **Praise:** one specific "Looks good" line. Accurate praise makes the rest easier to trust.
- **Declined to judge** (tier 3+): list anything you considered and set aside, one line each with the reason, so nothing is dropped silently.

## Re-review

Used for `🔍 Review new commits` in Discussion, and whenever earlier findings meet new code. It is not a fresh review:
1. **Each earlier finding gets a verdict**, in order: ✅ addressed or ❌ not addressed, with `file:line` evidence. "Attempted" is not addressed: the specific defect must no longer be reachable. A partial fix is ❌, with what is still reachable.
2. **New breakage:** problems the fix itself introduced, at the usual severities.
3. **Out of scope:** anything noticed outside the fix diff is listed as non-blocking, never as a new finding round.

## Overview fields

- **Verdict reasoning:** one or two sentences under the verdict saying why, e.g. "Core logic is solid; two leaks need fixing before merge."

- **Risk:** `low`, `medium`, or `high`. High only for a clear bug, a security problem, or a major logic risk.
- **Effort:** 1–5 for an experienced reviewer of this PR.
- **Merge danger:** `one-way door` (hard to roll back: data, migrations, public API, releases) or `two-way door`, plus the blast radius in one or two words (e.g. "checkout flow").
- **Impact:** changed symbols, their use count, and the uses not updated.
- **Tests:** covered, partly covered, or not covered, naming the gap.
- **What it does:** the smallest visual that shows the change: a call tree, component tree, or file tree, drawn as a `diff` sketch.
- **Ticket check:** requirements met, not met, and needing a human (e.g. "check on device").
- **Split suggestion:** only when the PR mixes unrelated changes. Name the parts.
