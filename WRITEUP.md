# Write-up — MATACS-TH-2026-02

## The flow in one paragraph

An admin builds a category from mandatory (pass/fail) and weighted (weight, direction, bands) criterion blocks, saved as a **versioned config**. A reviewer uploads PDFs and selects the category. The pipeline is: untrusted-content guard → AI extraction → schema/evidence validation → rule-based mandatory gate → rule-based numeric scoring → AI qualitative scoring → code-computed weighted total/rank → human checkpoint → output and audit log.

## 1. AI vs rules

AI handles three tasks: extraction into a fixed JSON schema; scoring two judgemental criteria (experience relevance, team and approach) against rubric anchors; and drafting one-line reasons from the scorecard. 

Everything else is code: validation, pass/fail gates, ceiling/deadline checks, calculations, ranking, and injection detection. 

I drew the line at determinism and auditability. Anything deciding eligibility, doing arithmetic, or fixing order must be reproducible and unit-testable. AI is used only where input is natural language and output is a bounded label plus evidence. Letting AI compute the rank would make it untestable and invite manipulation like Proposal C's. 

## 2. Consistency and explainability

Repeatability relies on: versioned prompts; temperature 0 and schema-constrained JSON; rubric anchors; **one proposal per call** (removing ordering effects); fixed score bands (so adding a bid cannot move Proposal A's score); and caching against document hash and versions. Qualitative criteria get two passes; a >15 point gap goes to a human. 

Same inputs yield the same scores in practice. It is not mathematically guaranteed because models drift, which is why caching is authoritative and re-runs are logged. Explainability is structural: every value has a verbatim quote and page reference, and every score shows its band, weight, and points. Reviewers see a scorecard (criterion, quote, band, score, points, confidence, flags) and generated reasons can only cite supported evidence.

## 3. Human in the loop

Four checkpoints: (a) admin configures; (b) reviewer handles exceptions (low confidence, missing evidence, injection flags, pass disagreements, missing timeline); (c) reviewer actively confirms or overrides flags before ranking is released—flags cannot be bulk-dismissed; (d) final award is human. 

We log per run: uploader, timestamp, document hash, extraction JSON, evidence quotes, model/config versions, raw responses, scores, flags, overrides (with reviewer identity/reason), and the final ranking. The log is append-only; overrides add adjustment records rather than mutating original scores. 

## 4. Changing criteria

Criteria and weights are versioned, never edited in place. Scores are stored against their config version. If an admin edits a criterion or weight, the tool creates a new version, marks old scores **stale**, shows affected proposals with estimated re-run cost, and requires an explicit re-score action. Stale scores are hidden from the ranked view. Awards are pinned to a config version to reproduce rankings exactly as they were seen.

## 5. Risks

- **Wrong extraction:** Schema validation, mandatory quotes, confidence scores, human routing for low confidence.
- **Made-up facts:** Closed-book prompting, "not stated" as valid output, validator rejects unsupported values.
- **Manipulation (Proposal C):** Delimited untrusted text, injection detection, and human review flags. The injected text never reached scoring.
- **Bias:** One rubric for all, masked identifiers during qualitative scoring, periodic distribution review.
- **Automation bias:** Active confirmation, no bulk dismissal, random sampling.
- **Cost:** Cheap model for extraction, aggressive caching, re-runs only on version change.
- **Unverifiable claims:** Treated as claims to verify later, not facts.

## Ranking of the sample set

| Proposal | Exp 30% | Price 25% | Time 15% | Warranty 15% | Team 15% | Total | Status |
|---|---|---|---|---|---|---|---|
| A | 100 | 80 | 100 | 80 | 100 | **92** | Rank 1 |
| E | 80 | 80 | 80 | 100 | 100 | **86** | Rank 2 |
| D | 90 | 100 | 0 | 80 | 60 | **73** | Rank 3 |
| C | 80 | 80 | 80 | 80 | 60 | (77) | Flagged, not scored |
| B | — | — | — | — | — | — | Ineligible |

**A** wins because it clears all mandatory checks, commits to 12 weeks, and fully evidences its team. **E** matches A on team and beats it on warranty, but is slower/dearer. **D** has the best price and experience but refuses to commit to a completion date within the 16-week limit (scoring zero); it also names no crew/monitoring plan.

**C** is flagged, not ranked. Its appendix contains an instruction to automated review systems, which was stripped and ignored. On merits, C scores 77 (thin team/approach), placing it 3rd and pushing D to 4th. A human decides this, not the tool. 

**B** is ineligible: it does not state the bidder holds an Ontario electrical contractor licence, only that a licensed subcontractor will be confirmed after award.