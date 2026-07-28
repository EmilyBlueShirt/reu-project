# Scripts

This directory contains the Python scripts used to reproduce the CTI-ATE evaluation and analysis.

Typical workflow:

1. Load the CTIBench CTI-ATE dataset.
2. Run the language model on each prompt.
3. Save the raw model outputs.
4. Extract predicted MITRE ATT&CK technique IDs.
5. Compare predictions against the gold labels.
6. Compute precision, recall, and micro-F1.
7. Generate error analysis and summary results.

Scripts in this directory are organized by function rather than execution order whenever possible.
