# prototype/ — reference pipeline

A small, dependency-free Python script that runs the deterministic half of
the design (hard constraints, mandatory gate, band scoring, weighted sum,
ranking, audit) and wires in the two AI stages as swappable calls.

Not required by the brief. It exists to show the design is executable and
that the A–E result in `../ranking.json` is reproducible.

## Quick start

    python pipeline.py --selftest     # asserts A 1st/92, E 2nd/86, D 3rd/73, C flagged, B ineligible
    python pipeline.py                # prints the ranking JSON
    python pipeline.py --explain      # per-criterion trace
    python pipeline.py --check ../ranking.json

Python 3.10+. No third-party packages.

## Live mode (calls a model)

    cp .env.example .env        # put your key in .env, or export it
    python pipeline.py --live --docs samples/

`--docs` is a folder of `.txt`, `.md` or `.pdf` files (PDFs need
`pip install pypdf`). **Place the sample proposals there yourself; the folder
is git-ignored and never committed.** The script reads nothing outside the
folder you pass.

## What is committed vs not

- Committed: pipeline code, category config, a **structured fixture**
  (booleans, numbers and band inputs only — no proposal prose, quotes,
  licence numbers or company descriptions), `.env.example`.
- Not committed: `.env`, `samples/`, `audit.jsonl`, `__pycache__`.

## Where the AI is

`run_live()` is the only function that calls a model. Everything else —
`hard_constraints`, `mandatory_gate`, `scan_for_injection`, `score_numeric`,
`band_le`, `band_ge`, `rank`, `to_ranking_json`, `write_audit` — is pure
Python. Replace `run_live()` with a stub and the rest of the pipeline is
unchanged: that is the whole point of the AI/rules split.