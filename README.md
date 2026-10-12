# devin-judge-action

GitHub Action that runs the
[`poordjaevin`](https://pypi.org/project/poordjaevin/) advisory risk
gate — the calibrated decision layer from the
[devin-judge](https://github.com/Icaro0310/devin-judge) repo of the
devin-* ecosystem.

It answers concrete safety checks about an action description
(default: "moves money", "deletes data") and fails the job on a
`block` verdict. Read/verify only: it scores text, never mutates
anything. The gate **fails closed** — a backend error is reported as a
block, not silently passed.

## Usage

```yaml
- uses: Icaro0310/devin-judge-action@v0.1.0
  with:
    action: "Delete the sessions table and rotate all tokens"
```

Gate text from a file (e.g. a generated plan or PR body):

```yaml
- uses: Icaro0310/devin-judge-action@v0.1.0
  with:
    action-file: plan.md
    calibrator: ci/calibration.json      # optional, from `poordjaevin calibrate`
    version: "poordjaevin[local]==0.1.2"
```

For an advisory (never-failing) verdict, keep the gate but tolerate the
exit:

```yaml
- uses: Icaro0310/devin-judge-action@v0.1.0
  continue-on-error: true
  id: judge
  with:
    action: "..."
# then read steps.judge.outputs.verdict / .block / .triggered
```

## The three distinct outcomes

| Situation | Exit | `verdict` |
|---|---|---|
| no risk check triggered | 0 | `allow` |
| a check triggered, abstention, or backend error | 1 | `block, require explicit confirmation` |
| nothing to gate / unreadable file / usage error | 2 | `block` (precondition failure) |

## Inputs / outputs

- `action` — the action text to gate.
- `action-file` — read the action text from a workspace file.
- `calibrator` — path to `calibration.json` (default: `calibration.json`;
  absent file means raw confidence).
- `backend` — `nli` (default; offline local model, first run downloads
  ~400MB) or `acp` (Devin ACP session — only on self-hosted runners with
  Devin credentials).
- `version` — pip specifier (default `poordjaevin[local]>=0.1.2,<0.2`;
  drop the `[local]` extra only with `backend: acp`).
- outputs: `verdict`, `block` (`true`/`false`), `triggered` (JSON map of
  the triggered checks and their scores/`ABSTAIN`).

## Notes

- The `nli` backend needs the `local` extra (torch + transformers); the
  install is heavyweight on first run — cache pip/model dirs or use a
  self-hosted runner for speed.
- Read/verify only by design — the judge scores a description and emits
  a verdict; it never touches the repo or the environment.
- The satellite repo is only the Action distribution vehicle; all
  decision logic lives in
  [`devin-judge`](https://github.com/Icaro0310/devin-judge).
