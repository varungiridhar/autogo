# Workflow

Read `AGENTS.md` for the shared experiment rules and code style.

- `/experiment` runs one bounded experiment.
- `/autoresearch` runs a sequence of experiments toward a research goal.

Both commands use `experiments/TEMPLATE.md`. The human fills in the
experiment-specific choices; the agent prepares scripts, runs experiments,
and writes reports.

The original Go code, experiments, dependencies, and cluster configuration
are historical reference. They are voided as defaults for this harness.
See `README.md` for the files you can skip.
