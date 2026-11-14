---
description: Create and run one experiment, saving results and findings
argument-hint: <experiment-description or path to experiment SPEC.md>
---

# Experiment Runner

Create a one-off experiment based on: $ARGUMENTS

Answer one question. Save the evidence, draw conclusions, and state what
is still unknown. An experiment may be a training run, a small sweep, a
benchmark, or an analysis of existing results.

## Setup

1. Read `AGENTS.md` and the experiment specification supplied by the user.
2. If there is no specification, copy `experiments/TEMPLATE.md` to `SPEC.md`
   in a new experiment folder. Fill in information already supplied. Ask the human
   for unresolved fields before running; do not invent an evaluation or
   compute budget.
3. Record the hypothesis and the observation that would support or reject
   it. For analysis-only work, record the question and source runs instead.
4. Use the baseline, edit scope, environment, and commands in the specification.

If called by autoresearch, use the campaign specification, assigned stage,
parent candidate, and remaining budget. Do not create a second budget.

## Folder layout

Keep the experiment small. Create scripts only when needed.

```
experiments/<YYYY-MM-DD_HH-MM>-<slug>/
    SPEC.md               # Filled-in specification
    run.py                # Optional experiment script or wrapper
    analyze.py            # Optional analysis of saved results
    results.tsv           # One row per run
    report.md             # Findings and next steps
    runs/0001/
        notes.md          # Hypothesis, exact command/config, input versions
        run.log           # Captured output, or durable link to remote logs
        result.json       # Measurements and artifact paths
```

Use `TZ="America/Los_Angeles" date +"%Y-%m-%d_%H-%M"` for the prefix.
Each execution gets a new run directory, including retries. Add only the
artifact folders the experiment needs. Large datasets and checkpoints may
live elsewhere; record durable locations and versions.

## Running one experiment

1. Prepare the candidate in its checkout. If code changed, commit only the
   allowed files before running. Include any new scripts/configs the run
   uses. Check `git status --short` and commit any remaining changes to
   those files. Use an isolated branch/worktree when needed to preserve user work.
2. Run `git rev-parse HEAD` in that checkout and save the full SHA as
   `code_commit` in `runs/<id>/result.json`, initially with `status: running`.
   Save the exact command, config paths/overrides, seeds, input versions,
   stage, limits, and evaluation rules in `runs/<id>/notes.md`. Reference
   committed configs by path at that SHA; do not duplicate them. External
   data/checkpoints need an immutable version or hash because Git does not
   track them. Ensure remote jobs execute the same code revision and inputs.
3. Run the specified command. A diagnostic may only read existing artifacts.
4. Monitor the log and resource limits. For remote work, record the job ID
   as soon as it is submitted. Enforce the timeout and cancel active work
   if the budget is exhausted; do not leave a remote job running after
   stopping its local monitor.
5. Save the result. A nonzero exit, missing required measurement, non-finite
   value, or failed validity check is a failed run, not an improvement.
   A simple fix may be retried within budget using a new run ID.
6. Run analysis against saved artifacts. Write findings with links to the
   measurements. Do not infer results from an unfinished job.

For a sweep, use one run record per configuration/seed/task execution and
summarize the comparison in `report.md`. Keep raw measurements as well as
aggregates. Use an existing scheduler for parallel execution when specified.

## Result format

Save `runs/<id>/result.json`. Use this shape; add domain-specific evidence
under `metrics` and `artifacts`:

```json
{
  "run_id": "0001",
  "parent_run": null,
  "code_commit": "<candidate commit>",
  "stage": "screen",
  "status": "completed",
  "metrics": {},
  "elapsed_seconds": 12.5,
  "artifacts": {}
}
```

- `parent_run`: run this candidate came from, or `null` for the baseline.
  Use a path when the parent belongs to another experiment folder.
- `code_commit`: revision of the code actually executed. Record the harness
  revision separately in the run notes if it is in another repository.
- `status`: `running`, `completed`, `failed`, or `cancelled`. Update the
  initial record when the run ends. This describes execution, not whether
  the candidate should be kept.
- `metrics`: named measurements; use `null` for missing values. An
  analysis-only experiment may leave this empty and link its evidence.
- `artifacts`: names mapped to paths or durable URLs for curves, tables,
  checkpoints, videos, or other evidence. Relative paths are relative to
  the run directory. Record versions or hashes in the run notes.

Use the existing runner's output to create this file. Do not rewrite a
working training loop just to change its output format.

Append one row per execution to `results.tsv`:

```text
run_id	parent_run	code_commit	stage	status	decision	result_path	description
```

`result_path` is relative to the experiment folder. `decision` is
`baseline`, `pending`, `keep`, `revisit`, `discard`, or `inconclusive`.
Use `revisit` for a candidate that does not pass the comparison but has a
specific reason for further investigation. Record that reason and the next
test; do not label the run an improvement. A completed run can still be
inconclusive. Update its decision when comparison is
finished, and explain the decision and supporting runs in `report.md`.
Metric names and directions come from the specification, never from guesses
about a column name. Compare only matching evaluation stages and controls.

## Report

Write `report.md` with these sections:

```markdown
# <slug>

## Goal
## Setup
## Results
## Key findings
## What is still unknown
## Next steps
```

Separate measured observations from explanations. State which conditions
a finding applies to and whether fresh seeds or larger tasks have confirmed
it. If later evidence changes a conclusion, mark the earlier conclusion
as superseded and link the new evidence.

Commit small run records, configurations, analysis, and reports so another
session can resume. Keep large artifacts and sensitive inputs out of Git;
record their locations instead. Finish with a short summary for the user.

## Constraints

- Stay within the specification's editable files and shared budget.
- Do not change the evaluator or comparison rules to favor a candidate.
- Preserve raw results, including failures and discarded candidates.
- A one-off experiment ends when its question is answered or its budget
  is exhausted. Return the findings to the caller; do not start an
  autoresearch campaign unless requested.
