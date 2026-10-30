# AI Proposal Triage Design

## Design Summary

AI extracts information from proposals and returns structured facts supported by evidence. Deterministic code applies the admin's criteria, performs mandatory checks, calculates weighted scores and ranks eligible proposals. Humans review exceptions, validate results and make the final award decision.

## 1. AI vs. Rules

**AI steps:**

- Extracting facts from proposal text, including licence details, insurance, installation count and dates, price, completion time, warranties and team information.
- Identifying whether team and approach elements are present, such as a named project lead, crew, safety plan and monitoring plan.
- Detecting suspicious instructions embedded in proposal documents that attempt to influence the reviewing system.

These tasks require language understanding because bidders may describe the same information differently.

**Rule-based or code steps:**

- Loading admin-defined criteria, mandatory requirements, budget ceiling and completion deadline.
- Validating extracted values against mandatory requirements.
- Calculating weighted scores, applying eligibility rules, ranking and generating reason statements.

The boundary is drawn where consistency and defensibility matter most. AI can interpret language, but it should not decide how many points a particular price or warranty receives. Deterministic code produces repeatable, testable results.

## 2. Consistency and Explainability

- Extraction uses a strict JSON schema and controlled model settings. Results are cached using the file hash, prompt version and model version.
- Scoring is performed through deterministic code. Given the same extracted facts and criteria version, the ranking will remain the same.
- Temperature 0 reduces output variability but does not guarantee identical model responses. If extraction is repeated with a different model or prompt, the system compares the new facts against the previous extraction and displays the differences for review.
- Ties are broken by higher experience score, then lower price, followed by proposal letter as a stable final fallback.

**Scoring formulas:**

Each normalized criterion score is multiplied by its configured weight to calculate the weighted contribution.

- **Experience (30%):** Qualifying installations in the last five years, capped at 12, divided by 12.
- **Price (25%):** Lowest eligible bid divided by the proposal's bid.
- **Timeline (15%):** Fastest eligible completion time divided by the proposal's completion time.
- **Warranty (15%):** Half the panel warranty score (years capped at 25) plus half the workmanship warranty score (years capped at 12).
- **Team and approach (15%):** Proportion of four elements present: named project lead, crew, safety plan and monitoring plan.

Price and timeline scores are relative to the eligible bid set. Adding or removing an eligible proposal can change these scores, so the system rescales and scores the complete set together.

Reviewers see a breakdown for each proposal, including the extracted value, supporting evidence quote and page reference, normalized score and weighted contribution. The final reason statement is generated from this breakdown using predefined templates, not invented by AI.

## 3. Human in the Loop

- **Extraction review:** Missing, low-confidence or unverifiable values are sent to a reviewer for confirmation or correction before scoring.
- **Eligibility and exception review:** Flagged and ineligible proposals are displayed with their specific reasons. Reviewers can investigate, request clarification or confirm an exclusion.
- **Ranking review:** Reviewers can override a rank but must provide a written justification.
- **Final award:** The system never makes the award decision. A person must also verify licence validity against the official registry before awarding the contract.

**Audit log:** The system records file hashes, model and prompt versions, criteria version, extracted values and evidence, scores, ranking changes, and every human action with the reviewer, timestamp, previous value, updated value and reason.

## 4. Changing Criteria

- Every scoring run stores a frozen snapshot of the criteria and its version number. Previous results are never overwritten.
- When an admin changes criteria or weights, existing results remain available and are labelled with the version under which they were scored.
- Reviewers can initiate a new scoring run. Previously extracted facts are reused where applicable, so only the scoring calculations need to run again. If a new criterion requires information that was not previously extracted, the relevant field is extracted.
- The reviewer sees a before-and-after ranking comparison so changes cannot silently reshuffle the results.
- Weights must total 100% before an admin can save the criteria. Criteria are locked once an award is recorded.

## 5. Risks and Safeguards

- **Incorrect extraction:** Evidence quotes must match the extracted source text. Missing, unverifiable or low-confidence values require human review.
- **Hallucinated facts:** The extraction prompt requires null when information is not explicitly stated. Missing weighted information receives zero for that criterion and is flagged for review.
- **Prompt injection:** Proposal content is treated as untrusted data. The model is instructed to ignore embedded instructions and return only the required schema. Suspicious content triggers a human review flag, but does not automatically disqualify or penalize the bidder. Its legitimate proposal information can still be evaluated.
- **Misleading claims:** Statements such as a licensed subcontractor being "confirmed after award" do not satisfy the requirement that the bidder holds and states the licence.
- **Bias:** Company names and other unnecessary identifying details are excluded from the scoring input. Fixed formulas prevent writing style and document presentation from influencing scores. Reviewers can inspect every calculation.
- **Bad PDFs:** Scanned, unreadable or garbled documents are detected and flagged for manual review rather than being treated as proposals with zero scores.
- **Date ambiguity:** Statements such as installations completed "since 2020" may not establish whether every installation falls within the last five years. Only verifiable qualifying installations are counted; unclear dates are flagged.
- **Cost and performance:** Extraction is performed once per file and cached. A small model is sufficient for structured extraction, input length is capped, and rescoring after criteria changes does not require new AI calls unless additional facts are needed.
- **Over-reliance on automation:** The interface clearly labels results as decision support. Flagged proposals and indicative scores are visibly distinguished from confirmed rankings.

**Assumptions:** A proposal failing either mandatory requirement or exceeding the $260,000 CAD budget ceiling is ineligible. A proposal with missing or unverifiable information about a hard project constraint, such as the 16-week completion deadline, is flagged and withheld from ranking until reviewed. The human reviewer retains final authority over eligibility and award decisions.