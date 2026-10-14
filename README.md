# Eval Mutation Lab

Mutation testing for stateful agent graders. Apply a deliberate trajectory change, rerun a grader and a separately specified intent oracle, inspect disagreements, and minimize a reproducible counterexample.

Version 0.1 is under development. The first release will include a standard-library Python CLI, a local SQLite reference adapter, a pinned official τ-bench adapter for two documented fixtures, and a browser report inspector. No benchmark-wide robustness or model-performance claim is intended.

See [SPEC.md](SPEC.md) for the implementation contract and release gates. Source inspiration and provenance come from the October 14 supervised research sprint; the official benchmark retains its original MIT license and attribution.
