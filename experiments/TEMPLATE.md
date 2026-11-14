# Experiment specification

Copy this file to `experiments/<YYYY-MM-DD_HH-MM>-<slug>/SPEC.md`.
The human fills in each `[HUMAN: ...]` field, with help from the agent if
useful. Write `none` or `not applicable` for unused fields. Remove unused
evaluation stages. This file is a specification, not a configuration parser.

## Goal

- Question: [HUMAN: What should we improve or understand?]
- Success: [HUMAN: What evidence would count as success?]
- Initial hypothesis: [HUMAN: Your starting idea, or ask the agent to propose one.]

A diagnostic can succeed by answering a question; it need not beat a score.

## Starting point and edit scope

- Project directory and baseline revision: [HUMAN: Path and Git commit.]
- Starting configuration, data, and checkpoints: [HUMAN: Paths plus versions or hashes.]
- Editable files: [HUMAN: Exact files or directories the agent may change.]
- Fixed files and inputs: [HUMAN: Evaluator, baseline, held-out data, and other controls.]
- Dependencies: [HUMAN: Existing environment and any permitted package additions.]

The agent records resolved versions before running. The baseline stays
available for comparison throughout the campaign.

## Running an experiment

- Working directory: [HUMAN: Directory from which to run commands.]
- Run command: [HUMAN: Command, or ask the agent to implement a runner in the editable scope.]
- Run inputs: [HUMAN: How to select config, seed, task, budget, and output directory.]
- Results: [HUMAN: Where metrics, curves, logs, and other evidence are written.]
- Resources: [HUMAN: CPU/GPU/memory requirements; local or existing scheduler.]
- Remote jobs: [HUMAN: Submit, status, and cancel commands, or not applicable.]

The agent may add a small wrapper to save the shared `result.json` format
in `experiment.md`. Keep the existing runner if one already works.

## Evaluation

- Measures: [HUMAN: Metric names and units; higher/lower/target, or qualitative checks.]
- Comparison: [HUMAN: Baseline, fixed resource budget, and how to handle noise.]
- Failure conditions: [HUMAN: What counts as divergence, invalid output, or an unacceptable regression?]

Fill in the stages you need. Fix these rules before comparing candidates.
Stage names below are examples; a one-off experiment can use only `screen`.
Promotion rules can allow promising regressions to receive larger tests.
State when that is allowed and how much of the shared budget it may use.

| Stage | Tasks/data, seeds, and per-run budget | Promotion or completion rule |
|---|---|---|
| screen | [HUMAN: Cheap experiments and baseline runs.] | [HUMAN: Threshold/top k/diagnostic evidence; how ties or uncertainty are handled.] |
| scale | [HUMAN: Broader suite and baseline runs.] | [HUMAN: What warrants confirmation?] |
| confirm | [HUMAN: Fresh seeds, held-out tasks, longer runs, and baseline runs.] | [HUMAN: What establishes a validated improvement?] |

For a sample-efficiency study, specify environment interactions separately
from wall-clock time and compute. Define stability using measurable
failures, collapses, or variation across seeds. Compare candidates and
baseline under matching conditions within each stage.

## Campaign budget and stopping

- Total budget: [HUMAN: Maximum runs, compute/cost, or wall-clock deadline; include evaluation and retries.]
- Per-run timeout: [HUMAN: Wall-clock limit and how to cancel the process or remote job.]
- Maximum concurrency: [HUMAN: 1 for sequential work, or a larger resource limit.]
- Stop when: [HUMAN: Success criterion, lack-of-progress rule, or budget exhaustion.]

Use the same limits for a one-off experiment. Autoresearch shares them
across all child experiments. Do not start work that cannot fit in the
remaining budget. If cost cannot be measured, use an enforceable run/time
limit as well.

## References and prior findings

- Relevant code, papers, and earlier experiments: [HUMAN: Paths/links, or none.]
- Known failures or constraints: [HUMAN: Things already tried or established, or none.]

Read earlier experiment files and reports as needed. Link findings to the
saved results, including contradictory evidence and the conditions under
which a finding holds. No external knowledge-base setup is needed.
