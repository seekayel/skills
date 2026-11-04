---
name: tenth-man
description: "Run a council review plus a rebel seat that reads every other review and argues the council is wrong. Trigger this if the user says '10th man' or 'tenth man'."
---

# Tenth man

After the 1973 Yom Kippur War caught Israel by surprise, its military intelligence added a rule. When everyone agrees, one person's job is to assume they're all wrong and make the case for it. That person is the tenth man.

This skill runs the `council` skill and adds that person. I call the seat the rebel.

## How it differs from council

Everything in `council` applies: the seats, the budget, the blind briefs, how you handle findings, and the report. This skill adds one seat that runs after the others every round.

The regular seats stay blind. They never learn the rebel exists. The rebel is the only seat that sees other reviews.

## The rebel

Run the rebel on Claude Opus 5.5 or GPT 6.1 Sol. If the user names one, use it. Otherwise use the one that isn't running you, so the rebel doesn't share your blind spots. If you're neither, use GPT 6.1 Sol. In T3 Code, confirm the model is listed in `orchestrator_capabilities` and take its ID from there. Use high reasoning effort.

If neither model can run, stop and report it. Don't substitute.

## Running a round

1. Run the council round exactly as `council` says. Wait for every seat.
2. Start the rebel with a fresh `delegate_task` (`role: "review"`, `mode: "async"`). Give it the same brief the seats got, plus every seat's full reply from this round.
3. Wait for the rebel. Then handle all findings together, the seats' and the rebel's.

Label the reviews `Reviewer A`, `Reviewer B`, and so on. Don't tell the rebel which model wrote which review. It should judge the arguments, not the brand.

The rebel is blind to the target too. It never sees a score the user asked for or your bar for approval. It does see the scores the seats gave, because a score is part of a review.

Add this to the rebel's brief:

```
You are the tenth man. Below are independent reviews of this change.
Assume they are wrong. Your job is the case they missed, not one more review.
Find:
- Bugs or risks none of them caught.
- Findings they agreed on without evidence. Check each one against the code.
- Findings that are wrong, or labeled more or less severe than they are.
- Assumptions every review shares. Is the change solving the right problem?
- Places a reviewer approved code it didn't read.
Every claim needs a file:line, a command, or output. No evidence, no claim.
Reply:
R1 [blocker|should|nit] <file>:<line> <claim>. Evidence: <...>. Fix: <suggestion>
R2 [challenge] Reviewer <X> C<n> <why it's wrong or mislabeled>. Evidence: <...>
```

The rebel doesn't score and doesn't approve. Its job is dissent, not a verdict.

## Handling the rebel

The rebel argues against the room on purpose. Take it seriously anyway. A tenth man is useful because you have to answer it, not because it's usually right.

- Investigate every rebel claim yourself. Don't dismiss one because the rebel is paid to disagree. Don't accept one for that reason either.
- A confirmed gap becomes a normal finding. Give it the severity it deserves and handle it like any other. A confirmed blocker or should blocks agreement.
- A confirmed challenge to a seat's finding changes how you handle that finding. If the rebel shows a fix you planned is wrong or unneeded, drop it.
- An unconfirmed claim gets a one-line reply with your evidence, in the ledger.

The rebel's arguments alone never block agreement. Its confirmed findings do. The council is done when the regular seats approve the current diff, no blocker or should is open, and you've answered every rebel claim from the last round.

Don't let the rebel grow the change. "You missed a case" is worth something only if a user would actually hit that case. Apply the same cost test as for any finding.

Later rounds give the rebel the brief, its own claims with outcomes, the change since last round, and this round's reviews.

## Reporting

Use the `council` report and add a row for the rebel. Show claims raised and how many held up, not a score:

```markdown
| Rebel | GPT-6.1-Sol | 4 claims, 1 confirmed | 1 claim, 0 confirmed |
```

Then add one line per confirmed rebel finding, and one line for the strongest claim you rejected and why.
