# AGENTS.md — pod-airflow

Standalone candy repo for the `airflow` candy — Apache Airflow 3.x
single-node (LocalExecutor + SQLite) with four supervisord wrapper services. The
entire candy lives in `charly.yml` at the repo root: the `airflow:` entity with
its requires, env, port, volume, secrets, `env_provide`, services, and a `plan:`
that installs the wrapper scripts. There is no source tree — the Airflow binary
comes from the required `pod-marimo` layer's pixi environment.

Canonical files:

- `charly.yml` — the `airflow:` candy entity (description, `require`, `env`,
  port, volume, `secret`, `env_provide`, `service`, `plan`).
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `CHANGELOG/` — per-CalVer history.
- `README.md` — user overview only; never agent guidance.

## Load these skills first (R0)

- `/charly-versa:airflow-layer` — the owning skill: Airflow 3.x compatibility
  findings, the SimpleAuthManager auth-fix pattern, the dag-processor
  split-from-scheduler architecture, and the JWT-issuance + REST trigger flow.
  Load before editing, building, deploying, or troubleshooting this candy.
- `/charly-versa:marimo-layer` — the companion layer that owns the Python
  environment Airflow's deps live in.
- `/charly-image:layer` — the candy authoring reference (`charly.yml` schema,
  `plan:` step verbs incl. `check:`, package sections, service declarations).
  Load before editing any entity field or plan step.
- `/charly-internals:git-workflow` — before any git/PR action.

The candy carries no `skill:` entity of its own; the family skill
`/charly-versa:airflow-layer` covers the surface. The gap is routed to the named skill-authoring
batch [opencharly/opencharly#291](https://github.com/opencharly/opencharly/issues/291);
when that lands, add the owning skill here.

## Build / validate / test

- `charly box validate` at the repo root — the structural check: the manifest
  must parse and validate at the installed charly.
- The merge gate is the **org-wide** `charly/pr-validator` (required check
  `validate / validate`, defined in `opencharly/.github`); this repo has **no**
  per-repo candy gate. Its only workflow file is
  `.github/workflows/tag-on-merge.yml`.
- The live R10 witness is the versa pod's `check` bed, which exercises the
  Airflow REST API + JWT issuance together with the marimo notebook. There is no
  airflow-only bed.

## Modify this repo

- Edit the `airflow:` candy entity in `charly.yml`. The four wrapper scripts are
  authored inline in the `plan:`; a change to a service's env, priority, or
  startup order belongs there.
- The `env_provide` block is the cross-container contract with the marimo
  notebook — keep `AIRFLOW_PUBLIC_URL`, `AIRFLOW_API_INTERNAL_URL`, and
  `AIRFLOW_DAGS_DIR` in step with the notebook's consumers.
- Python dependency changes belong in the marimo layer's `pixi.toml`, not here.

## Landing

- PR-only. Every change lands through a pull request; the org-required
  `charly/pr-validator` validates the diff and body and arms native auto-merge
  on PASS. Direct pushes to `main` are blocked.
- History lives in `CHANGELOG/` (written by `tag-on-merge` at merge time); the
  PR body IS the changelog.
- The authoritative rulebook is the umbrella `AGENTS.md` in
  `opencharly/opencharly` and `charly/AGENTS.md` in the charly repo. Do not
  restate its rules here.
