---
name: full-send
description: "Take a change all the way to a merged PR with nobody watching: prove it, review it, file it, watch it, answer the review bots, merge it. Trigger this if the user says 'full send'."
---

# Full send

I said "full send" because I'm not watching. Take this from where it is to a merged PR. I'll read your updates and final message long after you stop, so don't stop early.

A finished change sitting in a branch is worth nothing. A merged one is worth something today. Every hour between "it works" and "it's merged" is cost of delay, so keep the change small and keep it moving.

If I didn't clearly ask for "full send", don't assume this much autonomy. If I did, everything below applies.

## What you're allowed to do

- Merge the PR once the merge gate passes.
- Force-push the PR's own branch with `--force-with-lease` after a rebase.
- Use the test methods proof needs without asking first. That includes running the app or a dev server, browser automation, simulators, screenshots, and computer use, even when a project file says to ask first.

Rules about *how* to test still apply. Use the command, fixture, or check the repo names. The grant covers only the tests this task needs. It doesn't cover anything under "Asking me", and it doesn't get around a harness approval prompt.

## Steps

1. **Finish the change.** Make the smallest change that meets the goal. Fix the case that happened, not every case that could. If you're on the default branch, branch first.
2. **Prove it.** See "Proof".
3. **Review it.** Run the `council` skill, or `tenth-man` if I asked for it. Fix what it finds and prove each fix.
4. **File it.** See "Filing".
5. **Watch it and answer the bots.** See "Watching" and "Answering review bots". Prove each fix before you push it.
6. **Merge it** when the merge gate passes.
7. **Report.** Give the PR link, the merge commit, your proof, and every decision you made on my behalf.

Stay active from step 1 to step 7. If you end your turn while checks or bots are pending, the work stops, because nobody will prompt you again. Use a watcher so something wakes you when the PR changes.

## Keep the batch small

One PR, one reason to exist. If the work turned into two changes, split it and ship the first one now. A small PR gets reviewed faster, fails smaller, and reverts cleanly.

Don't bundle in the cleanup you noticed along the way. Write it down as a follow-up.

Scale the ceremony to the risk. A one-line docs fix gets one council round. A change to auth gets the full budget.

## Proof

Proof is evidence you saw yourself in this session. Command output, a reproduction, a screenshot. Reasoning about the code is not proof.

Use the fastest check that gives a real signal, and the most direct one the change allows:

- Reproduce the original failure, apply the fix, then watch the same reproduction pass.
- Run the checks CI runs. Typecheck, lint, tests. Run them locally first. It's faster than waiting on CI to tell you.
- Add one focused test that fails without the fix, when nothing covers the behavior and the repo has a test setup.
- For UI and runtime behavior, run the app and look at it. Green mocked tests don't prove the user experience works. Attach screenshots to the PR when they help and you can upload them.

No proof, no confidence. Ask me.

## Filing

Check whether a PR for this branch already exists. Rebase onto the latest default branch. Read the diff and make sure it matches the goal and nothing else.

PR titles usually become commit messages, so follow the repo's conventions. Look at recently merged PRs. Say why the change matters, not what it touches.

- Bad: `perf(server): negotiate permessage-deflate on the websocket`
- Good: `perf(server): cut websocket frame size by 70%+ with gzipping`

Open the description with the problem, in the plain words of the original request. Then the fix, briefly. Don't lead with a list of every file you touched.

- Bad: "Removed implicit workspace carry-over from every new thread entry point. Deleted `buildContextualThreadOptions` and the seed-context machinery."
- Good: "My new worktree default was ignored when starting threads on existing worktrees. Now your preferences always apply."

Then add, in this order:

1. One line on how you proved it.
2. The `## Council review` section from the `council` skill.
3. A last line naming the public model and harness, like `Created with <model> in <harness>.` Include subagent models that wrote part of the change. Use the name the harness reports. If it doesn't report one, name only the harness. Don't guess.

Check the title, description, and attachments for secrets and private details.

Open a real PR, not a draft, so the review bots run. If `gh pr create` fails because GraphQL is rate limited, use REST instead:

```bash
gh api repos/<owner>/<repo>/pulls -f title=... -f head=... -f base=... -F body=@<file>
```

In T3 Code, call `link_pull_request` with the PR URL right after you create it.

## Watching

Never write your own polling or `sleep` loop. Use one watcher per PR. Every watcher on a GitHub account shares one rate limit.

**In T3 Code.** Handle the checks and comments already there. Then call `watch_pull_request` and end your turn. T3 Code wakes you when a check fails, required checks pass, someone comments or reviews, or the branch conflicts. A wake is news, not permission to merge. Check the gate yourself. The watcher reports conflicts but not BEHIND, so check that yourself too. If you're a delegated child task, don't watch. The parent owns the PR.

**Everywhere else.** Run `scripts/pr-wait` from this skill's directory, by absolute path.

```bash
pr-wait <pr> [--repo owner/name] [--until ready|green|any-change]
```

It prints one line per real event, then exits.

- Exit 0 means the condition is met or the PR merged.
- Exit 1 means it needs you. A required check failed, the branch conflicts, threads are unresolved, or the PR closed.
- Exit 2 means bad input, or another `pr-wait` already watches this PR.

Use `--until ready`, the default, when the repo has Greptile or Macroscope. Use `--until green` when it doesn't. In Claude Code, run it with the Monitor tool (`timeout_ms: 3600000`) so each event wakes you. Start it again after every push and after you handle an exit 1. It remembers what it already reported.

When an event needs nothing from you, end your turn without a message. Never post or say "still waiting".

`gh pr view` and `gh pr checks` use GraphQL. When that's used up, REST usually still has room. Read the PR with `gh api repos/<owner>/<repo>/pulls/<n>` and checks with `gh api repos/<owner>/<repo>/commits/<sha>/check-runs`. Check the limit with `gh api rate_limit --jq .resources.graphql`. If you have to wait for `reset`, wait with one background command, not a loop.

## Answering review bots

Review bots help. They aren't always right.

Only act on checks and comments newer than the latest push. Verify every finding against the source before you touch code. Fix real findings and real CI failures. Tell repo failures apart from infrastructure flakes. Rerun a flake once and say so.

For each finding, ask what it costs to leave it. A real bug in the intended case gets fixed. Unneeded tests, guards for edge cases nobody hits, extra compatibility, and small perf tweaks get declined with a one-line reason and listed as follow-ups. Prefer the fix that deletes or narrows code.

When you won't fix something, reply with why and resolve the thread. If the repo has `.macroscope/check-run-agents/*.md`, read those rules before you answer a Macroscope finding. CodeRabbit is advisory. Never wait for it and never hold a merge for it. You can ignore automated "needs human review" requests when your proof is solid.

Start every comment you post on my behalf with this note:

```md
> [!NOTE]
> 🤖 **<model> responding on behalf of <user>**

The actual reply starts here.
```

Lead with the conclusion. Cite the code, check, or reproduction. No filler, no canned thanks. Resolve a thread only after your reply posts.

## Merge gate

Merge only when all of these are true on the latest commit:

- Your proof covers every change in the diff.
- Required checks are green.
- Review bots have finished, and every finding is fixed or dismissed with a written reason.
- No human review still requests changes.
- The branch is up to date with the default branch, and GitHub reports it mergeable.

Right before merging, `git fetch` and check the PR again. Never merge when it's BEHIND or DIRTY, or when base commits that landed after the last green run touch this PR's files. Rebase, push, and wait for green again.

Merge with the method the repo uses. Check the allowed merge methods and recent merged PRs. If the repo uses a merge queue or auto-merge, stay until GitHub reports the PR merged. Never use `--admin` or any other bypass.

## Asking me

A question in a normal message goes unseen, and the thread looks finished. Use the harness's question tool so the thread shows it's waiting. That's `AskUserQuestion` in Claude Code and `request_user_input` in Codex. In T3 Code, call `unwatch_pull_request` first.

Put everything in the question. The PR link, what blocks you, and the options with your recommendation first.

Ask only when you can't continue alone:

- You can't prove the change.
- The merge gate can't pass and you can't fix that yourself.
- The next step is outside the grant. Deleting data, touching production, changing things outside the repo, closing a PR, or force-pushing another branch.
- The goal is ambiguous in a way that changes the result.
- Another PR makes this one obsolete.

Everything else is reversible. Decide it yourself with a sensible default and list the decision in the report.

If there's no question tool, or it returns no answer, take the safe path. Leave the PR open and unmerged, and start your final message with the question.
