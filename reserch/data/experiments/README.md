# Experiment Data

Folder untuk hasil simulasi, replay, what-if analysis, dan calibration experiments.

Recommended experiment types:

```text
quote ladder replay
size optimizer comparison
inventory allocation simulation
natural-netting simulation
bridge batch threshold simulation
failed-leg/unwind simulation
dynamic slippage calibration
route reliability scoring
min-cost rebalancing experiments
```

Each experiment should include:

```text
experiment_id
timestamp
input_dataset_refs
assumptions
parameters
method
result
limitations
reproducibility_notes
```

Rules:

- label assumptions explicitly;
- never mix simulated output with measured live data without a clear marker;
- preserve parameter sets so results are reproducible;
- rejected hypotheses are valuable evidence and should not be deleted.
