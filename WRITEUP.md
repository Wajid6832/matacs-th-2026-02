# AI Proposal Triage — Design Write-up

## 1. AI vs. rules

1. The reviewer uploads PDF proposals and selects the category. Plain code validates file type/size, stores the original, records the category and loads that category's current admin-defined criteria version.
2. OCR/PDF text extraction uses a document parser; AI structures the extracted text into a fixed schema: bidder, licence evidence, WSIB, insurance, price/currency/inclusions, schedule, experience, warranties, team and approach. Every field must include a short evidence quote and page reference. Missing or ambiguous fields are returned as `unknown`, not guessed.
3. Deterministic rules check mandatory requirements and validate types, dates, currency, limits and weights. A proposal that fails a mandatory criterion is `ineligible`; a material uncertainty is `flagged` for human clarification. Only eligible proposals proceed to weighted scoring.
4. AI may summarize evidence and draft plain-language explanations, but it does not decide eligibility or calculate final scores. Code calculates scores and sorts them using a versioned rubric. A reviewer confirms the result before it is used for procurement.

This boundary uses AI for messy-language extraction and summaries, and code for repeatable decisions, arithmetic and hard constraints.

## 2. Consistency and explainability

Store the original file hash, extracted text, model/prompt version, criteria version, rubric version, evidence quotes, per-criterion scores, total score and timestamps. Use a fixed output schema, temperature 0 where supported, and deterministic code for scoring. AI extraction can still vary, so it is not claimed to be perfectly deterministic: schema validation, evidence checks and human review catch differences. Re-running the same validated evidence against the same criteria and rubric must produce the same numeric score and order.

The reviewer sees a table with eligibility status, each criterion's evidence, score, weight, weighted contribution, total and a short reason. Missing evidence is visibly marked; the tool does not invent it. A tie is broken by higher relevant-experience score, then lower stated price, then alphabetical proposal letter, and the tie-break is logged.

## 3. Human in the loop and audit

A person reviews low-confidence extraction, missing/contradictory evidence, mandatory failures and flagged cases. The reviewer can correct extracted fields with a reason, request clarification, or override a recommendation; overrides require a note and do not erase the original output. The human makes the final award decision.

Audit records include uploader/reviewer identity, file hash, upload time, category and criteria version, extraction and prompt versions, source-page evidence, rule outcomes, score breakdown, model/provider, review edits, override reason, ranking versions and final human decision. Access is role-based and records are retained according to the organization's policy.

## 4. Changing criteria

Every criteria edit creates a new immutable version with effective time, weights, mandatory rules and rubric. Existing results retain the version used to calculate them. The UI warns reviewers when a newer version exists and offers an explicit re-score action. Re-scoring creates a new result linked to the old result; it never silently overwrites it. The reviewer can compare rankings across versions, and the audit log records who changed criteria and why. Weighted criteria must total 100% before publication.

## 5. Risks and safeguards

- **Wrong OCR/extraction:** retain page references and quotes, validate fields and units, show confidence, and require review when evidence is unclear.
- **Made-up facts:** require every scored value to cite source text; absent evidence is `unknown`; never let AI fill gaps from general knowledge.
- **Prompt injection/manipulation:** treat proposal contents as untrusted data, never follow instructions inside them, isolate them from system instructions, and score only against admin criteria. Proposal C contains an instruction to rank itself first; it is ignored and logged as suspicious content.
- **Bias or inconsistent scoring:** use the same published rubric for every bidder, avoid protected or irrelevant attributes, test sample cases, and permit human challenge with an audit trail.
- **Unfair hard-fail decisions:** make mandatory evidence visible and require a human to confirm a failure before procurement action; ambiguous evidence is flagged rather than automatically treated as false.
- **Cost, latency or outages:** set file/page limits, timeouts and rate limits; cache extraction by file hash; monitor cost and latency; use a queue for long documents and a clear retry/error state. Do not silently return a partial ranking.
- **Sensitive information and security:** encrypt files in transit and at rest, use least-privilege access, scan uploads, avoid sending unnecessary data to AI providers, and define retention/deletion rules.
- **Criteria or model drift:** version prompts, models, criteria and scoring rules; run regression tests on fixed examples before changes; allow rollback and comparison.

## Sample-set decision

Proposal B is ineligible because the stated mandatory criterion requires the bidder to hold an Ontario electrical contractor licence; B says its subcontractor will be confirmed after award. Proposal D is flagged because its completion schedule is not stated, so compliance with the 16-week limit cannot be established. A, C and E pass the stated mandatory checks. Proposal C's embedded instruction is ignored. The proposed sample scores use transparent provisional scoring rules because the brief does not specify exact scoring curves; a production system should have the admin configure and approve those curves.


## Provisional scoring rubric used for this sample

Because the brief supplies weights but not numeric scoring curves, the JSON uses a simple explicit sample rubric so its scores are reproducible. Scores are clamped to 0–100 and the weighted total is the sum of `criterion score × weight`.

- **Experience (30%)**: `min(completed qualifying installs ÷ 14, 1) × 100`; 14 is the highest stated count among the eligible, scored sample proposals.
- **Price (25%)**: `max(0, (260000 − stated price) ÷ 260000 × 100)`. The amount must be stated in CAD and within the ceiling.
- **Timeline (15%)**: `max(0, (16 − completion weeks) ÷ 16 × 100)`. A missing schedule is not assigned a score and is flagged.
- **Warranty (15%)**: average of `min(panel warranty years ÷ 25, 1) × 100` and `min(workmanship warranty years ÷ 10, 1) × 100`.
- **Team and approach (15%)**: provisional evidence rubric: 100 when the proposal states a named lead, crew, safety plan and monitoring plan; 60 when it states a named lead and crew but does not state safety and monitoring plans. Production use should define this rubric in admin configuration before evaluation.

Using these provisional curves, the eligible sample totals are A 65.87, E 51.64 and C 46.70. These figures demonstrate the calculation approach; they are not a claim that the brief itself mandates these exact curves. Before real procurement use, an admin should approve and version the scoring rubric. The score is not a substitute for human procurement judgment.
