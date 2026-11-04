---
name: council
description: "Get review or feedback from a council of models from different providers, and loop until they agree. Trigger this if the user says 'council'."
---

# Council

Get a second, third, and fourth opinion from models that don't share your blind spots. Then ship.

The council exists to catch real bugs cheaply. It doesn't exist to make the change perfect. Every round costs wall-clock time while the change sits unshipped, so the job is to buy the most signal for the fewest rounds and get out.

## Before it sits

Do the cheap work first. Fix the issues you already know about, answer open PR comments, and run the checks. Don't hold back a known fix waiting for reviews. That spends a round finding something you already found.

If the user wants to throw the implementation away and explore new ideas, skip fixes to code that's being replaced.

## Seats

Use the latest flagship model from each provider at high reasoning effort. If the user names models, in the request or in `CLAUDE.md` / `AGENTS.md`, use those.

| Seat | Model | Sits when |
| --- | --- | --- |
| Opus | Claude Opus | Always |
| Codex | GPT Sol, through Codex | Always |
| Muse | Muse Code | Installed and authenticated |
| Grok | Grok | Installed and authenticated |
| Jester | Gemini Pro | Installed and authenticated |

Opus and Codex are required. If either can't run, stop and report it. Don't substitute another model.

Skip the optional seats without asking when they aren't available. Name each skipped seat and why in the report.

In T3 Code, call `orchestrator_capabilities`. A seat is available when its provider has `canRunChildTask: true`, a non-empty `models` list, and no constraint saying it's disabled, not installed, or not authenticated. Take model IDs from that list. Don't hardcode them. Gemini may show up under Antigravity, OpenCode, or Pi. Use whichever available provider lists a Gemini Pro model.

Outside T3 Code, use the harness's subagent tools or the provider's CLI. If you can't confirm a seat is logged in, it's unavailable. Never start an interactive login.

## The jester

Gemini sits as the jester. Everyone else reviews the change. The jester reviews the decision to make it.

Its job is to ask the questions the other seats won't:

- Is this the right problem? Is there a smaller change that gets the same result, or no change at all?
- What did everyone assume without checking?
- Which fixes the other seats asked for aren't worth the time they cost?

Answer every jester objection with evidence. Its objections don't block agreement, though. When the regular seats agree, the council is done, and any open jester objection goes in the report as "jester dissent", one line each.

The jester is often wrong. That's fine. One good "you don't need this" pays for a lot of bad ones.

## Budget

Three rounds by default. Round 1 is a full review. Later rounds check only the changed code and the open findings.

Go to five only for security, auth, permissions, data loss, or concurrency, while a blocker or should is still open with evidence behind it. Or when the user asks.

Stop early when a round returns only nits. Declined nits and comment, doc, or format-only fixes don't earn another round.

## Keep the seats blind

Each seat reviews alone, as if it were the only reviewer. A seat never learns:

- The score, verdict, or bar you're aiming for. If the user said "get this to 9/10" or "keep going until everyone approves", that stays with you. A reviewer who knows the target grades toward it, and then the score tells you nothing.
- That other reviewers exist. No "council", no "panel", no "the other models said". Never pass one seat's findings, scores, or seat name to another seat.

Don't point a seat at the PR. Its description and comments may hold earlier scores and other reviews. Give it the diff command instead.

A seat does get the intent of the change. That's what it reviews against. What it doesn't get is your goal for the review.

## Running a round

In T3 Code, call `delegate_task` once per seat, in parallel, with `role: "review"` and `mode: "async"`. Save each `taskId`. Use a fresh `clientRequestId` per seat per round. Each round is a new `delegate_task`. Never continue a review with `t3_thread_send` on a `childThreadId`.

Child tasks get no history. Every brief carries everything it needs:

```
Review only. Do not edit, commit, push, comment on PRs, or start subagents.
Repo: <abs path>  Base: <sha>  Intent: <intent>  Out of scope: <scope>
Checks: <results>
See: git diff <base> [-- <paths>]; git status --short (read untracked files).
Find bugs, broken callers, and mismatches with the intent. Skip style unless
it hides a bug. Hold the change to the smallest one that meets the intent.
Flag code, tests, or guards the intent doesn't need.
blocker = the intended case is wrong.
should = a real bug callers will hit.
nit = everything else, including rare paths and small perf costs.
Reply:
SCORE: <n>/10
VERDICT: APPROVE or CHANGES
C1 [blocker|should|nit] <file>:<line> <claim>. Fix: <suggestion>
```

For the jester, add one line. It still doesn't learn about the other seats. `You are the jester. Challenge the premise, the unchecked assumptions, and any scope the change doesn't need.`

Don't edit the reviewed files while a round runs. Wait for the completion notices. Don't poll. A timeout, an error, or silence is not approval. Cancel that seat and run it again.

## Between rounds

Check every finding yourself. A reviewer's label is a claim, not a fact. A rare path or a small cost is a nit, whatever it was labeled.

For each finding, ask what it costs if you don't fix it. Who hits it? How often? How bad is it when they do? Then:

- Fix it if it's a real bug in the intended case. Make the smallest fix. Prefer deleting or narrowing code over adding a lock, cache, helper, or option.
- Disagree with evidence. Give a file:line, a test, or output.
- Decline it in one line if it's out of scope or a nit, and list it as a follow-up.

Keep a ledger of every finding and its outcome. Rerun the checks and `git diff --shortstat <base>`. If review grew the diff, cut before the next round.

Start the next round only if budget remains and a blocker or should is open, or behavior changed since the last approval. Send each seat the original brief, its own findings with their outcomes (`fixed:` / `disagree:` / `declined:` with evidence), and the change since last round. Ask it to mark each of its old findings `resolved` or `still open: <reason>`. Fixes that came from other seats show up in the diff with no credit attached. Don't explain where they came from.

## Stopping

Done when every regular seat approves the current diff and no blocker or should is open.

Stuck when an item is still disputed after two exchanges, or the budget runs out. Show both sides in two or three lines each and let the user decide. Running out of budget is not approval.

Don't agree with a finding just to end the loop. Don't let the loop grow the change either. Review loops grow a change one reasonable-looking fix at a time. Your job is to keep it small and get it out.

## Reporting

For a code review, every seat scores out of 10. Each full pass is one round. If a seat gives no score, write `no score`. Never invent one.

If the work ends in a pull request, put the results in the PR description, above the model and harness line at the end:

```markdown
## Council review

Rounds: 2

| Seat | Model | Round 1 | Round 2 |
| --- | --- | --- | --- |
| Opus | Claude Opus 5.5 | 7/10 | 9/10 |
| Codex | GPT-6.1-Sol | 8/10 | 10/10 |
| Jester | Gemini Pro | 5/10 | 6/10 |

Skipped: Muse (not authenticated), Grok (not installed).
Jester dissent: none.
```

Edit the description of an open PR. Don't post a comment for this. With no PR, put the same results in the final reply, along with what changed, what you declined and why, and the follow-ups.
