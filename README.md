# GateKeeper: Detecting Semantic Weakening in Detection-as-Code Rules using a CI/CD Integrity Gate

GateKeeper is a proof-of-concept pre-deployment CI/CD integrity gate for Detection-as-Code workflows using Sigma rules. It compares a modified Sigma rule with its baseline and flags changes that may weaken detection coverage, even when the modified rule still passes `sigma check`.

Developed as an MSc Cyber Security dissertation project. Author details withheld for peer review.

## Weakening patterns evaluated
- Narrowed match
- Threshold inflation
- Log-source drift
- Added exclusion
- Weakened condition

Non-weakening changes (unchanged, cosmetic edit, and strengthened rules) are also included as negative cases so that false positives can be measured.

## Approaches
- **Rule-based:** deterministic checks for the five known weakening patterns.
- **LLM-assisted:** reviews the semantic difference between the baseline and modified rule.
- **Hybrid:** rule-based checks screen every pair, and only flagged pairs are escalated to LLM review.

## Datasets
- **Hand-built:** 10 baseline-modified pairs.
- **Generated:** 116 pairs created from 20 sampled SigmaHQ Windows process-creation rules, using a pinned SigmaHQ commit and `random.seed(42)`.
- **Independent (PowerShell):** 45 pairs created from SigmaHQ Windows PowerShell rules using `random.seed(123)`, to test whether the approach generalises to a rule type not used in development.

## Results (precision / recall / F1)

| Approach | Hand-built | Generated | Independent (PowerShell) |
| --- | --- | --- | --- |
| Rule-based | 88% / 100% / 93% | 80% / 100% / 89% | 92% / 100% / 96% |
| LLM-assisted | 100% / 86% / 92% | 100% / 95% / 97% | 100% / 100% / 100% |
| Hybrid | 100% / 100% / 100% | 98% / 93% / 96% | 100% / 100% / 100% |

## Key findings
- The rule-based detector reached 100% recall on all three datasets. Its false positives were all strengthened rules.
- The LLM-assisted reviewer is non-deterministic on weakened-condition cases; it missed three such cases on the generated set, and its verdict on these cases can change between runs.
- The hybrid detector is therefore valuable for efficiency rather than higher accuracy. On the generated set it used LLM review for 76 of 116 pairs (about a third fewer LLM calls) while reaching comparable results.
- In a SIEM testbed, the original `whoami` rule detected 4 of 4 attacker events, while the weakened version detected only 1 of 4.
- These are proof-of-concept results, not general performance claims.

## Requirements
Ubuntu under VMware, Python 3.12, Sigma CLI, PyYAML, Elasticsearch 8.15 (testbed only), and Anthropic API access (LLM and hybrid evaluation only). The rule-based detectors run without an API key.

## Reproduction
1. Unzip `evaluation_generated.zip` and `evaluation_independent.zip`.
2. Rule-based results: run `evaluate.py` (hand-built), `evaluate_generated.py` (generated), and `evaluate_independent.py` (PowerShell).
3. LLM-assisted and hybrid results need `ANTHROPIC_API_KEY` set in the environment.
4. The generated and independent datasets can be rebuilt with `generate_pairs.py` and `generate_powershell.py`.
5. The LLM configuration is `claude-sonnet-4-5` with a 600-token maximum output and default temperature.

## Limitations
This is a research prototype, not a production platform. Results depend on the included datasets. The full methodology and limitations are described in the dissertation.
