# MATACS AI Proposal Triage Design

**Candidate:** Muhammad Wajid  
**Reference:** MATACS-TH-2026-02  
**Time spent:** 60 minutes  
**Email used for the brief:** chwajid6832@gmail.com

## Repository contents

- [Process flow diagram](diagram/process-flow.png)
- [Design write-up](WRITEUP.md)
- [AI prompts](PROMPTS.md)
- [Expected proposal ranking](ranking.json)

## How I used AI

I used ChatGPT to help structure the design, draft an extraction prompt, and check that the ranking logic addressed the stated criteria. I reviewed the output against the brief rather than accepting it blindly: I rejected the instruction embedded in Proposal C's appendix that asks an automated reviewer to rank it first, because proposal text is untrusted evidence, not an instruction to the system. The final ranking applies the mandatory eligibility rules before weighted scoring and leaves uncertain information for human review.

## Assumptions

The brief specifies criteria and weights but does not define numeric scoring curves or tie-break rules. The write-up therefore proposes an explicit, versioned scoring rubric and calls out missing evidence rather than silently inventing facts.
