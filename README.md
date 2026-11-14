# AutoGo — general experiment harness

A small harness for running experiments with a coding agent. Adapted from
[Eric Jang's AutoGo](https://github.com/ericjang/autogo). The workflow is
plain Markdown; it does not depend on a particular model or agent SDK.

- **`experiment`**: answer one question, save the evidence, and write a report.
- **`autoresearch`**: pursue a research goal by running experiments, keeping
  improvements, and investigating failures.

Both commands use the same experiment format. A quick experiment can run
on its own or contribute to an ongoing autoresearch campaign.

## Start here — human setup

1. Read [the experiment template](experiments/TEMPLATE.md). Copy it to
   `experiments/<YYYY-MM-DD_HH-MM>-<slug>/README.md`.
2. Fill in every `[HUMAN: ...]` field. Write `none` or `not applicable`
   where appropriate. You can ask the agent to help draft these choices.
   Set the goal, edit scope, evaluation, and budget before execution.
3. Point your agent at that file:
   ```text
   /experiment experiments/<folder>/README.md
   /autoresearch experiments/<folder>/README.md
   ```
4. Read `report.md` for findings and `results.tsv` for the run history.

The slash commands live in `.claude/commands/`. With another coding agent,
ask it to read `AGENTS.md` and the relevant command file, using your prompt
in place of `$ARGUMENTS`. There is no separate runner to install. The agent
executes the command you specify using your project's environment.

## Read the harness in this order

| File | Purpose |
|---|---|
| [experiments/TEMPLATE.md](experiments/TEMPLATE.md) | The fields you fill in for each experiment or campaign. |
| [.claude/commands/experiment.md](.claude/commands/experiment.md) | How one experiment is prepared, run, and reported. |
| [.claude/commands/autoresearch.md](.claude/commands/autoresearch.md) | How the agent chooses and compares successive experiments. |
| [AGENTS.md](AGENTS.md) | Shared rules for either command. |
| [CLAUDE.md](CLAUDE.md) | Points Claude to the shared rules. |

## What changes between experiments?

The research question, baseline, editable files, run command, evaluation,
and resource limits. The template puts these choices in one place.

For example, a sample-efficiency study might screen ideas on a small
simulator, send promising candidates to a full task suite, then confirm
with fresh seeds and longer training. A debugging experiment might only
need one diagnostic and a report. Use only the stages you need.

Metrics and promotion rules are fixed within a comparison. If the
question or evaluator changes, record a new comparison and rerun the
baseline. A useful negative result is still worth saving.

## Model, compute, and knowledge access

Use the coding agent and job scheduler you already have. Keep experiment
files and results independent of the model's conversation history.

The template includes relevant papers, earlier runs, code, and optional
search tools. Read the relevant sources as needed; keep links to evidence
in reports. Parallel agents are optional and share the campaign budget.
The default is one agent running experiments sequentially.

## Legacy AutoGo files — skip these when reviewing the harness

**The following files are voided as defaults for this harness.** They are
preserved as historical reference, not deleted or generalized. They do not
supply the run command, dependencies, evaluator, or infrastructure for new
experiments unless you explicitly choose to reuse them.

| Legacy files | Original purpose |
|---|---|
| `src/alpha_go/` | Go rules, networks, MCTS, self-play, and Go data handling. |
| Existing dated folders under `experiments/2026-*` | Go experiments and their results. |
| `infra/`, `cluster.toml` | Original SSH/Docker/NFS cluster setup and GPU leases. |
| `.devcontainer/`, `Dockerfile.worker`, `scripts/build_cpp.sh` | Original Go development and worker images. |
| `pyproject.toml`, `uv.lock`, `tests/` | Dependencies and tests for the original Go project and infrastructure. |

Do not run the old setup or install its dependencies to use the harness.
Use your experiment's environment instead. The original documentation is
available in [upstream AutoGo at the fork point](https://github.com/ericjang/autogo/tree/54ac8d4191b006cec627d6c6b5db7ac07826ffdd).
