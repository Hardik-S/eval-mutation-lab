# Release contract: Eval Mutation Lab 0.1

Purpose: give an agent-grader author a runnable tool that mutates a trajectory, compares configured grader reward with explicit independent intent invariants, minimizes a disagreement, and exports evidence another developer can reproduce. This is developer tooling, not a new benchmark score or a claim of research novelty.

## Product and release gates

- Install the standard-library-only core as `eval-mutation-lab`; `python -m eval_mutation_lab` also works on Windows/Linux, Python 3.10+.
- `run --adapter sqlite --out report.json [--suite custom.json] [--minimize]` works immediately, without model keys or a download. At least two reference task fixtures, clean and equivalent controls, meaningful wrong-state mutations, and policy/process gaps.
- `run --adapter tau --source PATH --out report.json [--minimize]` calls the unchanged pinned τ-bench evaluator in real local environments. Preserve the nine existing cases and no_knowledge/empty-NL/source-aware oracle limits. Optional dependencies and a separately verified source fetch are explicit.
- `validate REPORT`, `diff BEFORE AFTER`, and `minimize --adapter ... --case CASE_ID --out ...` are useful real operations. The CLI prints concise findings and returns documented nonzero exit statuses for invalid input/check failures; ordinary discovered coverage gaps are valid research output, not runtime crashes.
- Report schema v1, canonical semantic fingerprints, before/after event trace, configured grader scope, independent checks, source links, and reproduction information. No fake execution button or fabricated finding.
- Minimizer actually reruns the adapter in a fresh environment for each candidate. Preserve the original verdict and named failing oracle dimensions; bound work. Claim only deletion-based 1-minimality after a complete final single-deletion sweep; exhaustion must say incomplete.
- Browser app explores shipped actual reports and imports a local report JSON. Filters, task/case navigation, side-by-side trace changes, invariant/state evidence, minimized trace, JSON export, and getting-started instructions all work. Imports remain in browser memory; no telemetry or server upload. Keyboard use, mobile layout, useful empty/error states and readable code required.
- Public source has MIT license/third-party notices, reproducible commands, architecture/tradeoffs, CI, failure cases and accurate scope. Root independently tests clean installation, all report operations, upstream adapter and browser flows before adding it to the portfolio.

## Ownership

Root owns this contract, release docs/CI, integration, independent verification, public source and portfolio/deploy.
Pod A core lead owns `pyproject.toml`, `src/eval_mutation_lab/` EXCEPT `adapters/tau.py`, `examples/sqlite/`, and `tests/` EXCEPT `test_tau.py`.
Pod A benchmark specialist owns `src/eval_mutation_lab/adapters/tau.py`, `benchmarks/tau/`, and `tests/test_tau.py`.
Pod B product lead owns `web/` (including web tests and generated sample report copies).
All workers preserve others' edits. No worker commits, pushes, deploys, or changes the portfolio. Coordinate via this contract and concise messages.

## Adapter interface (plain JSON dictionaries)

Adapters expose `id`, `provenance` (JSON dict), `load_suite(path: Path | None = None) -> dict`, and `evaluate(task: dict, trace: list[dict]) -> dict`.

Suite:
```
{"id":"sqlite-demo", "title":"...", "tasks":[
 {"id":"exchange", "domain":"retail", "title":"...", "description":"...",
  "baseline":[EVENT,...], "mutations":[MUTATION,...],
  "initial_state":{}, "expected_state":{}, "policy":{}, "sources":[]}
]}
```
Adapters may use extra task fields. Tau task IDs/source paths are fixture references, not copies of the huge upstream database. Baseline and mutations must derive from documented task intent; expected validity is fixture-authored, not inferred from grader reward.

Event: `{"id":"e1", "role":"user|assistant|tool", "content":"...", "tool":"...", "arguments":{}, "result":{}, "error":false, "meta":{}}`. Only relevant fields are present. Tool events combine call/result. IDs are unique within a trace. Tool calls execute against a fresh local fixture; recorded `result` fields are observations, not trusted execution instructions.

Mutation: `{"id":"omit-confirmation", "family":"consent", "operator":"drop", "event_id":"e3", "expected_validity":"invalid", "description":"..."}`. Core supports `drop`, `replace` (event_id plus field/value), `move` (event_id plus after_id or before_id), `reorder` (event_ids plus order), and `duplicate` (event_id). No arbitrary code in a JSON suite. Unsupported operator/input must fail clearly. Core lead documents exact field selectors before extending them; benchmark specialist follows that API.

Evaluation:
```
{"official":{"passed":true,"reward":1.0,"scope":"DB only","checks":[]},
 "oracle":{"passed":false,"state_valid":true,"scope":"fixture-specific invariants",
           "checks":[{"id":"confirmation-before-write","passed":false,"description":"...","sources":[]}]},
 "state":{"expected":{},"observed":{}}, "trace":[EVENT,...], "limitations":[]}
```
`trace` is the enriched executed trace. Core computes verdict from official/oracle booleans: both pass=`conforms`; both fail=`caught`; official pass/oracle fail=`coverage_gap`; official fail/oracle pass=`false_rejection`. Expected-validity mismatch is surfaced explicitly, not hidden. No combined robustness score.

## Report schema v1

```
{"schema_version":"1.0", "tool":{"name":"Eval Mutation Lab","version":"0.1.0"},
 "run":{"id":"...","created_at":"ISO8601","adapter":{...},"suite_id":"..."},
 "summary":{...}, "tasks":[{"id":"...","domain":"...","title":"...","description":"...","sources":[]}],
 "cases":[{"id":"exchange/clean", "task_id":"exchange", "domain":"retail", "label":"clean",
 "family":"control", "is_control":true, "expected_validity":"valid", "mutation":null,
 "baseline_trace":[...], "trace":[...], "official":{}, "oracle":{}, "state":{},
 "verdict":"conforms", "fingerprint":"sha256", "sources":[], "limitations":[],
 "reproduction":{"adapter":"sqlite","task":{},"mutation":null,"command":"..."},
 "minimized":null}]}
```
Case fields `official`, `oracle`, `state` come from Evaluation. `minimized` when present contains `trace`, `evaluation`, `evaluations`, `complete`, `one_minimal`, `original_event_count`, `event_count`, and preserved failing-check IDs. Summary is derived from cases, with per-task/family counts and caveats; cases are not independent benchmark tasks.

Semantic fingerprints exclude only envelope event `id` and `meta`, and report creation time. Preserve event order and all content/tool arguments/results/errors, including business timestamps inside arguments. Do not recursively strip keys named timestamp. `run.id` derives from semantic case evidence; repeated equivalent runs should match despite envelope clocks. Imported reports must validate versions, types, unique case/event IDs, finite rewards and required fields before display; UI derives counts from cases.

## Visual direction

Use a polished engineering workbench: compact project identity, clear problem and quickstart, persistent task/case sidebar on desktop, readable matrix and selected trace details. Warm paper/ink or restrained dark surfaces, deliberate typography and one accent. The evidence is the main content, not a huge hero. Use native semantic controls, visible keyboard focus, real empty/error states. A static hosted inspector is appropriate: execution belongs to the local CLI. Make this distinction apparent where a user chooses an action.

Web files use relative asset/data links so the app works at `/` locally and `/eval-mutation-lab/` on the personal site. The report inspector must not import private operational data, embed credentials, call models, or imply its sample findings generalize beyond the two pinned upstream fixtures and reference examples.
