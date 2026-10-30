# MATACS-TH-2026-02 — AI-Assisted Proposal Triage Design

**Name:** Ali Talha
**Reference:** MATACS-TH-2026-02
**Time actually spent:** ~2.5 hours (design, diagram, write-up)

A design (not an implementation) for a tool that lets an admin define
evaluation criteria per category, lets a reviewer upload contractor
proposals, and returns a ranked list with a reason for each position,
with a human making the final award decision.

## Files

| File | Contents |
|---|---|
| [diagram/process-flow.png](diagram/process-flow.png) | Process flow, upload → ranked output, marking AI steps, rule-based steps and human checkpoints |
| [WRITEUP.md](WRITEUP.md) | Answers to the five questions (under 1,000 words) |
| [PROMPTS.md](PROMPTS.md) | The extraction, scoring and explanation prompts, exactly as sent |
| [ranking.json](ranking.json) | Expected ranking of proposals A–E |
| [prototype/README.md](prototype/README.md) | Optional reference pipeline: how to run it |
| [prototype/pipeline.py](prototype/pipeline.py) | Rule engine, band scoring, ranking, audit; AI calls behind a single swappable function |
| [prototype/config/commercial_solar.json](prototype/config/commercial_solar.json) | Versioned category config (weights, bands, mandatory rules) |
| [prototype/fixtures/extracted_demo.json](prototype/fixtures/extracted_demo.json) | Structured fixture (numbers and booleans only) used by `--selftest` |
| [prototype/.env.example](prototype/.env.example) | Environment template for live mode; no keys are committed |

## How I used AI tools

I used an AI assistant as a design reviewer: to pressure-test my AI
versus-rules split and help refine the system instructions for the
PROMPTS.md. I designed the process flow, made the scoring judgements and
determined the edge-case handling myself.

One place I disagreed with the AI: it initially proposed scoring price
by min-max normalising across the uploaded proposals (cheapest bid gets
100, dearest gets 0). I rejected that and replaced it with fixed
percentage-of-ceiling bands, because min-max means a proposal's price
score changes depending on who else happened to bid. Fixed bands give
the same score for the same price every time. I also rejected its
suggestion that the model compute the weighted total and rank; that is
now plain code.

## Assumptions

- `ranking.json` is a top-level JSON array of the five objects, the only
  reading of the format in the brief that parses as valid JSON.
- Proposal B is treated as failing the mandatory licence criterion as
  written ("held by the bidder and stated in the proposal"). If an admin
  later allows licensed subcontractors, B is re-run, not re-interpreted.
- Proposal C is flagged rather than auto-scored; its provisional merits
  score is recorded in the write-up.
- No proposal text, and no text from the brief, is reproduced in this
  repository. Proposals are referred to by letter only; the prototype
  fixture contains structured values only.

## scope

No deployed app and no API keys. The prototype runs offline against a
structured fixture; live mode reads documents from a git-ignored folder
and uses `.env` (template provided).