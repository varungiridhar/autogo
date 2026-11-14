---
description: Run sequential experiments toward a research goal
argument-hint: <research-goal or path to experiment SPEC.md>
---

# Autonomous research loop

Run experiments toward a research goal. Keep improvements, investigate
failures, and use what you learn to choose the next experiment.

User's research goal: $ARGUMENTS

## Setup

For a new campaign, follow the steps below. To resume an existing campaign,
read its specification and `state.md`, reconcile active jobs and the budget,
and continue the loop. Do not recreate its branch or overwrite its run log.

1. Read `AGENTS.md` and `.claude/commands/experiment.md`. Use the experiment
   workflow for every run; the folder layout and result format are shared.
2. Read the supplied experiment specification, or copy
   `experiments/TEMPLATE.md` to `SPEC.md` in a new experiment folder. Fill in information
   already supplied and ask the human for unresolved fields before execution.
3. Create a branch such as `autoresearch/<MMMDD>-<goal>`. It must not already
   exist. Use a separate worktree if another experiment is in progress.
4. Initialize `results.tsv` using the header in `experiment.md`.
5. Create `state.md` with the fields below. This is the place to resume after
   an interruption or a new context window.

```markdown
# Research state

## Current question and hypothesis
## Baseline and candidate commits
## Best validated candidate at each stage
## Findings and links to supporting runs
## Ideas to try next
## Active runs, job IDs, and output paths
## Budget used and remaining
## Next action
```

A campaign uses one specification and one total budget. Do not create
independent budgets for its experiments, retries, or workers.

## What you edit

**Only the files allowed by the specification.** Prefer one experiment
script until the task needs more. Keep baseline code and evaluation fixed.

Forbidden:

- Editing files outside the allowed scope
- Installing packages outside the permitted dependencies
- Changing the evaluator, metric definition, or comparison rules mid-search
- Changing resources within a comparison without recording a new comparison
- Exceeding the shared budget

If the evaluator is wrong or the research question changes, record the
problem and ask the human to revise the specification. Start a new
comparison and rerun the baseline under the revised rules. Preserve the old
results, but do not rank them against the new ones.

## The loop

Repeat while the stopping conditions allow:

1. Read `state.md`, `results.tsv`, and relevant reports. Check existing jobs
   before launching anything; resuming a session must not duplicate a run.
2. **First run = baseline.** Run the unmodified starting point under the
   specified evaluation. Establish a valid baseline before ranking changes.
3. Pick one question or change to try. State why it is worth testing and
   what result would change the next decision. A diagnostic experiment is
   allowed even when it is not expected to improve the metric.
4. Choose the parent candidate and evaluation stage. Confirm the run and
   its evaluation fit within the remaining budget.
5. Follow `experiment.md`: prepare the change, commit the candidate code,
   run it, save evidence, and log the result. Record the exact candidate
   commit; later commits containing reports are not candidate revisions.
6. Apply the stage's comparison rules. Mark the result `keep`, `revisit`,
   `discard`, or `inconclusive`. Failed runs cannot be promoted. Do not
   treat a noisy improvement as confirmed.
7. A regression can still be promising: it may expose a useful mechanism,
   improve stability, or plausibly help at a larger scale. Mark it `revisit`
   with a concrete reason and a follow-up experiment. Keep its code commit
   and run records; it can be the parent of later experiments. Keep the
   best validated candidate separately. Follow-ups must fit the budget;
   moving to another stage must follow the specified promotion rules.
   If you decide to return to the parent, restore only the candidate's
   editable code from the recorded parent commit. Do not assume `HEAD~1`
   is the parent; diagnostic and report commits may have intervened.
8. Update `report.md` and `state.md`, including budget use, active jobs,
   evidence, and the next action. Commit the small records.
9. Decide whether to refine the current idea, test a different hypothesis,
   run a diagnostic, or promote a candidate to a broader evaluation.

## Evaluation stages

Use only the stages in the specification. A common sequence is:

1. **Screen:** cheap experiments to identify promising ideas.
2. **Scale:** compare selected candidates and the baseline on the full suite.
3. **Confirm:** repeat with fresh seeds, held-out tasks, or longer training.

`keep` means keep the candidate at its current stage. Only a candidate that
passes the final required stage is a validated improvement. If the budget
ends sooner, report it as promising but unconfirmed.

Use the specified promotion rules, including any top-k rule and treatment
of uncertainty. Compare baseline and candidates under matching conditions.
Do not repeatedly tune against the final held-out evaluation; if its
results guide new changes, disclose that reuse and obtain fresh confirmation
data before claiming held-out performance.

## Quick experiments during a campaign

The human can request `/experiment` at any point. Use the same experiment
format, linking its parent run and campaign. Give it a separate run/output
directory and count its work against the campaign budget when it belongs
to this campaign. Use an isolated worktree if it changes code while other
runs are active.

Read the findings before the next research decision. A diagnostic does
not replace the best candidate or change the evaluator by itself.

## Strategy

- **Simpler beats complex at equal evidence.** Removing code with no
  meaningful regression is a win.
- **One variable at a time** when possible.
- Keep failed ideas and the conditions under which they failed. Do not
  repeat them without a reason.
- Read the specified references and saved experiment files when they can
  answer the current question. Link findings to the relevant runs and reports.
- **Stuck?** Re-read the code, inspect raw outputs, check the baseline,
  revisit an earlier candidate, or try a different hypothesis. Step back:
  are we investigating the right bottleneck?

## Optional parallel work

Default to sequential work. If the user allows multiple workers, delegate
independent questions with explicit edit scope and a share of the budget.
Use separate worktrees and run directories. Workers return evidence and
reports; the lead agent alone updates campaign state and selects candidates.
Use existing agent and scheduler tools. Do not build a new orchestration
framework or let workers recursively expand the resource budget.

## Stopping and summary

Do not pause to ask "should I keep going?" between authorized experiments.
Continue until the specified success, budget, or stopping condition is met,
or the human interrupts. Cancel active jobs when required and save state.

Write the final findings to `report.md`:

- Baseline vs best candidate under comparable conditions
- Stage reached and whether the improvement is confirmed
- Important findings, including negative and inconclusive results
- Run counts and total resources used
- Reproduction commands and links to evidence
- What is still unknown and the most useful next experiment

Plot progress if it helps explain the result. Keep different evaluation
stages separate; a single running-best curve must not mix incompatible
metrics, tasks, or budgets.
