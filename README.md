# super-eureka: a PLE harness

An entry for the [Physical-Engineering’s Last Exam](https://ple-bench.com)
harness competition. It starts from the official
[submission template](https://github.com/physical-engineerings-last-exam/submission-template):
the [Pi](https://github.com/badlogic/pi-mono) coding agent, with its file and
shell tools redirected to the task container, now pointed at
`google-cloud/gemini-3.8-flash` (cheap, so it sits well on the score-vs-cost
frontier).

`artifacts/` is not committed yet: it must come from a dev-set run of this
exact harness and model (`eval/run.py`), so the template's own Claude Opus 5.5
run was left out.

## Run it

From a clone of [google-research/ple](https://github.com/google-research/ple)
next to this repository:

```bash
# google-cloud models run locally through a Vertex AI proxy in your own GCP project
gcloud auth application-default login
export VERTEXAI_PROJECT=<your-gcp-project> MODEL_PROXY_KEY=$(openssl rand -hex 24)
kit/model_proxy/start.sh &
export MODEL_PROXY_URL=http://localhost:4000

uv run python -m kit check ../super-eureka            # build, model, listen, smoke, mcp
uv run python eval/run.py ../super-eureka             # 29 dev tasks, writes artifacts/
uv run python -m kit artifacts ../super-eureka        # validate artifacts/
```

The dev run also needs Harbor (`uv tool install harbor==0.20.0 --with websockets`)
and an `HF_TOKEN` with access to the gated
[dataset](https://huggingface.co/datasets/google/physical-engineerings-last-exam).
Commit the harness and `artifacts/` together: `manifest.json` hashes every
harness file, so any change after the run needs a fresh run.

## Changing the model

Set `model_api` in `submission.yaml` to another `google-cloud/<id>` model and
`THINKING` under `env:` to its top reasoning level (`high` for Gemini, `max`
for Claude and GLM). Claude and Gemini use their native APIs; every other model
uses the OpenAI-compatible endpoint with the `UNKNOWN_MODEL` limits in
`harness.py`.

## Files

| File | Purpose |
|---|---|
| `submission.yaml` | The model, and the install and start commands |
| `serve.py` | The WebSocket endpoint: hands each task to `harness.solve` |
| `harness.py` | Runs Pi on each task. Edit `NOTE` (prompt) or `UNKNOWN_MODEL` (limits for a model Pi doesn't know) |
| `task-tools.ts` | Points Pi's `read`, `write`, `edit` and `bash` tools at the task container |
| `taskbridge.py` | The competition protocol |
| `package.json`, `package-lock.json` | Pi 1.1.0 and every package it uses, pinned |
| `AGENTS.md` | The full competition rules, for a coding agent working on this entry |

Licensed under Apache 2.0 (see `LICENSE`), like the template it comes from.
