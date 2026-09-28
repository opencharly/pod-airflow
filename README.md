# airflow

Apache Airflow 3.x as an OpenCharly candy — single-node, LocalExecutor + SQLite.

This candy installs four wrapper scripts that drive Airflow under supervisord and
serves an authenticated REST API on `:8080`. Airflow runs single-node against a
SQLite database with zero external services, which makes it suitable for
single-node dev and R10 verification. Python dependencies live in the `marimo`
layer's pixi environment (this candy requires `pod-marimo`); the candy itself
installs only the wrapper scripts that drive the Airflow binary.

## What it provides

| Property | Value |
|---|---|
| Layer / candy | `airflow` |
| Services / port | `airflow-init` (25), `airflow-scheduler` (30), `airflow-dag-processor` (31), `airflow-webserver` (31) on `8080` |
| Volume | `~/airflow` |
| Requires | `pod-marimo`, `layer-supervisord` |
| Secrets | `airflow-fernet-key`, `airflow-webserver-secret`, `airflow-admin-password` |

`airflow-init` is a one-shot `db migrate` that also binds the injected
`AIRFLOW_ADMIN_PASSWORD` secret to the SimpleAuthManager admin user. DAGs live
under `AIRFLOW__CORE__DAGS_FOLDER` (`/workspace/dags`), so notebooks can
self-author DAGs at runtime.

## Cross-container env

The candy publishes three `env_provide` vars for a co-deployed notebook:

| Var | Meaning |
|---|---|
| `AIRFLOW_PUBLIC_URL` | browser-side UI links and OAuth callbacks |
| `AIRFLOW_API_INTERNAL_URL` | kernel-side REST calls (`http://<container>:8080`) |
| `AIRFLOW_DAGS_DIR` | the DAG folder path inside the workspace volume |

## How to use it

Compose the candy into a box (the `versa` pod composes it with the marimo
notebook layer):

```yaml
my-versa:
  candy:
    base: cachyos.cachyos
    candy:
      - '@github.com/opencharly/pod-airflow:<tag>'
```

Then build and deploy with the charly CLI:

```bash
charly box build my-versa
charly start my-versa
```

See the owning skill for the Airflow 3.x compatibility findings, the
SimpleAuthManager auth pattern, and the JWT-issuance + REST trigger flow.

## Layout

- `charly.yml` — the `airflow:` candy entity: the two requires, the env block,
  the port, the volume, the secrets, the `env_provide` block, the four services,
  and the plan that installs the wrapper scripts.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `CHANGELOG/` — per-CalVer history.
- `README.md` — this user overview.

## Related

- Owning skill: `/charly-versa:airflow-layer` — Airflow 3.x compatibility, the
  auth pattern, the dag-processor split, and the REST/JWT trigger flow.
- Companion layer: `/charly-versa:marimo-layer` (owns the Python environment).
- [`opencharly/charly`](https://github.com/opencharly/charly) — the charly CLI and image builder
- [`opencharly/opencharly`](https://github.com/opencharly/opencharly) — the umbrella
