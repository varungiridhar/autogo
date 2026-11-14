# Workflow

Read the experiment's `README.md` before running anything. It defines the
question, baseline, editable files, commands, evaluation, and budget.
Use `experiments/TEMPLATE.md` when setting up a new experiment.

- `.claude/commands/experiment.md` — one bounded experiment.
- `.claude/commands/autoresearch.md` — a sequence of experiments toward a goal.

The commands are instructions for a coding agent, not executable scripts.
Use the same workflow with any model that can edit files and run commands.

# Code style guidelines

- Prioritize simplicity and compactness of code.
- Always be trying to reduce complexity of the codebase.
- Use existing project tools and libraries. Do not add an orchestration
  framework to run an experiment.
- Keep scripts, results, and reports easy to inspect and reproduce.
- Prefer one script until the experiment needs more.

# Experiment rules

- Resolve required `[HUMAN: ...]` fields before execution. Use information
  already supplied by the user; ask only for missing choices that affect
  scope, evaluation, or resources. Do not invent those choices.
- Edit only the files allowed by the experiment specification. Keep the
  baseline, evaluator, and held-out data outside the candidate's edit scope.
- Use the specified environment. Install packages only if the user has
  authorized them in the specification or conversation.
- Keep unrelated user changes intact. Commit only the files for this work.
- Record exact code revisions, commands, configuration, seeds, and input
  versions. A path alone is not a version if its contents can change.
- Preserve failed and discarded runs. Missing measurements are `null`,
  never zero. Separate observations from proposed explanations.
- Stay within the shared budget, including retries, evaluation, and any
  parallel workers. Stop active jobs when the deadline or run limit is met.
- Never treat an agent's own summary as stronger evidence than the saved
  measurements. Link conclusions to the runs that support them.

# Legacy files

`src/alpha_go/`, the existing dated Go experiments, `infra/`, cluster and
container configuration, and the original dependencies and tests are
historical reference. They are voided as defaults for the general harness.
Do not use their Go-specific conventions unless the experiment calls for
it. See `README.md` for the full list.

# Output paths

Use `experiments/<YYYY-MM-DD_HH-MM>-<slug>/` for new work. For the prefix:

```bash
TZ="America/Los_Angeles" date +"%Y-%m-%d_%H-%M"
```

Use a distinct slug if the folder already exists. Do not overwrite a run.
