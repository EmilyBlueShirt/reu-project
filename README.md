# Analyzing LLM Failures in MITRE ATT&CK Technique Extraction
This repository contains the code, analysis, and results developed during my NSF REU in Trustworthy AI and Cybersecurity at the Rochester Institute of Technology.

The project reproduces the CTIBench CTI-ATE benchmark, analyzes the failure modes of an open-weight large language model on MITRE ATT&CK technique extraction, and evaluates an evidence-grounded staged extraction pipeline designed to reduce unsupported ATT&CK technique predictions.


## Research Question
What kinds of LLM errors most strongly influence micro-F1 in MITRE ATT&CK technique extraction, and can targeting an observed failure mode measurably improve extraction?


## Project Overview
Cyber threat intelligence (CTI) reports describe adversary behaviors observed during cyber intrusions. Security analysts often translate these reports into standardized MITRE ATT&CK technique IDs so threats can be compared, tracked, and analyzed consistently.

Although large language models show promise for automating this process, they frequently produce unsupported technique predictions, assign incorrect ATT&CK techniques, or fail to recover relevant techniques. Rather than immediately focusing on improving model performance, this project investigates **why** performance is limited by reproducing the CTIBench evaluation, manually analyzing model errors, and designing an intervention informed by those observations.



## Research Progression
The project followed the following progression:

1. Reproduced the CTIBench CTI-ATE benchmark using Llama 3.1 8B Instruct.
2. Extracted predicted MITRE ATT&CK technique IDs from model outputs.
3. Calculated true positives, false positives, false negatives, precision, recall, and micro-F1.
4. Manually analyzed failure modes across all 60 benchmark examples.
5. Evaluated whether parser behavior affected measured performance.
6. Designed an evidence-grounded staged extraction pipeline based on the dominant observed errors.
7. Re-evaluated the benchmark using the staged pipeline.
8. Performed paired error analysis to understand why performance changed.
9. Explored retrieval-assisted Stage 2 mapping as future work (not validated).



## Dataset
Experiments use the CTIBench CTI-ATE dataset.

Dataset summary:
- 60 CTI reports
- 47 Enterprise examples
- 13 Mobile examples
- 397 expert-assigned MITRE ATT&CK technique IDs

Important dataset fields include:
- Platform
- Description
- Prompt
- Ground Truth (GT)

Please obtain the dataset from the official CTIBench repository rather than this repository if redistribution is restricted.



## Experimental Setup
Model:
- Llama 3.1 8B Instruct

Inference:
- Ollama
- Temperature = 0

Reproducibility:
- Five complete baseline runs
- All runs produced identical outputs
- Standard deviation of aggregate precision, recall, and micro-F1 = 0.0000



## Evaluation

CTI-ATE is a multi-label extraction task, so evaluation uses precision, recall, and micro-F1 instead of accuracy.

Metrics:
- True Positive (TP): predicted ATT&CK ID appears in the gold labels.
- False Positive (FP): predicted ATT&CK ID is unsupported.
- False Negative (FN): a gold ATT&CK ID was missed.

Reported metrics include:
- Micro-precision
- Micro-recall
- Micro-F1
- Per-example F1



## Baseline Pipeline
For every benchmark example:

1. Read the Prompt field.
2. Generate a response with Llama 3.1 8B Instruct.
3. Save the raw output.
4. Extract predicted ATT&CK IDs.
5. Compare predictions with gold labels.
6. Compute TP, FP, FN, precision, recall, and F1.



## Observed Failure Modes

All 60 benchmark examples were manually coded for recurring error patterns.

Dominant failure modes included:

| Failure Mode | Count |
|--------------|------:|
| Wrong ATT&CK mapping | 43 |
| Unsupported extra IDs | 37 |
| Parser / output issues | 21 |
| Missed gold IDs | 13 |
| Platform mismatch | 1 |

Definitions:

- **Wrong ATT&CK mapping:** the model identified the correct behavior but assigned the wrong ATT&CK technique.
- **Unsupported extra IDs:** the model predicted techniques without evidence in the CTI report.
- **Parser / output issues:** formatting affected prediction extraction.
- **Missed gold IDs:** relevant techniques were omitted.
- **Platform mismatch:** techniques were predicted from the wrong ATT&CK platform.

These observations motivated the staged pipeline.

---

## Parser Diagnostic

A stricter parsing rule was applied to previously generated outputs to determine whether evaluation methodology explained the baseline score.

Although the revised parser slightly changed measured performance, unsupported ATT&CK predictions remained the dominant source of error. This suggested the primary problem existed at the model level rather than the evaluation level.

---

## Evidence-Grounded Staged Extraction

The staged pipeline separates behavior extraction from ATT&CK mapping.

### Stage 1 — Behavior Extraction

Extract only behaviors explicitly supported by the CTI report and provide quoted evidence.

### Stage 2 — ATT&CK Mapping

Map extracted behaviors to ATT&CK technique IDs using a platform-restricted controlled technique list.

### Structured Output

Return predictions as structured JSON to enable consistent parsing and evaluation.

The objective of the staged design is to reduce unsupported ATT&CK predictions by requiring evidence before assigning techniques.

---

## Results

| Method | Precision | Recall | Micro-F1 | TP | FP | FN |
|--------|----------:|-------:|---------:|---:|---:|---:|
| Baseline | 0.2047 | 0.2393 | 0.2207 | 95 | 369 | 302 |
| Evidence-Grounded Staged | 0.3236 | 0.2519 | 0.2833 | 100 | 209 | 297 |

Key findings:

- Precision increased by 58%.
- Recall increased only slightly.
- Micro-F1 increased from 0.2207 to 0.2833.
- False positives decreased by 43% (369 → 209).
- False negatives changed only slightly (302 → 297).

The improvement primarily resulted from reducing unsupported ATT&CK predictions rather than recovering additional gold techniques.

---

## Paired Error Analysis

A detailed paired comparison was completed for 19 benchmark examples (18 coded with high confidence).

Within this subset:

- 16 of 19 residual errors involved Stage 2 ATT&CK mapping.
- Stage 1 frequently extracted the correct behaviors.
- Remaining errors primarily involved confusion between similar ATT&CK techniques.
- Formatting issues persisted in two examples.

These observations suggest that reducing unsupported predictions and selecting the correct ATT&CK technique are separate problems.

---

## Exploratory Retrieval (Not Validated)

Retrieval was explored as a possible intervention for improving Stage 2 mapping.

Candidate recall increased with the number of retrieved techniques:

- k = 8 → 0.4253
- k = 25 → 0.6667
- k = 40 → 0.8046

A bounded three-example retrieval experiment did not demonstrate improved extraction performance.

The complete 60-example retrieval evaluation was not completed and should be considered exploratory.

Retrieving the correct ATT&CK candidate does not necessarily mean the model selects it.

---

## Repository Structure

```
data/
prompts/
scripts/
outputs/
results/
poster/
figures/
notes/
```

---

## Limitations

- One open-weight language model.
- Temperature fixed at zero.
- Dataset limited to 60 examples.
- Absolute precision, recall, and micro-F1 remain relatively low.
- Paired error analysis covers 19 of 60 examples.
- Retrieval experiments remain exploratory.

---

## Future Work

- Improve Stage 2 behavior-to-technique mapping.
- Reduce remaining false negatives.
- Extend paired error analysis to the complete benchmark.
- Evaluate additional open-weight language models.
- Validate retrieval-assisted mapping on the complete dataset.

---

## Acknowledgements

This work was completed during the NSF REU in Trustworthy AI and Cybersecurity at the Rochester Institute of Technology.

Faculty Advisor:
Dr. Nidhi Rastogi

PhD Mentor:
Will Coffey

---

## Citation

If you use this repository, please also cite the original CTIBench paper:

Alam et al. *CTIBench: A Benchmark for Evaluating Large Language Models in Cyber Threat Intelligence*. NeurIPS 2024.
