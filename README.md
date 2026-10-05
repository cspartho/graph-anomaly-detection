# CIDDS-001 Network Anomaly Detection with a Heterogeneous GNN

A memory-conscious experiment that classifies network flows from the CIDDS-001 internal week-1 capture into five traffic classes using a heterogeneous graph neural network.

Each graph contains `host` and `flow` nodes. Source and destination hosts are connected to their flow, and a multi-layer GraphSAGE model passes information between the two node types before predicting the flow class.

## Results

Saved test-set report from the notebook run:

| Class | Test examples | Precision | Recall | F1-score |
| --- | ---: | ---: | ---: | ---: |
| Benign | 700,801 | 1.0000 | 1.0000 | 1.0000 |
| Brute force | 163 | 0.9360 | 0.9877 | 0.9612 |
| DoS | 125,158 | 1.0000 | 1.0000 | 1.0000 |
| Ping scan | 336 | 0.8791 | 0.9524 | 0.9143 |
| Port scan | 18,342 | 0.9990 | 0.9974 | 0.9982 |
| **Overall** | **844,800** | **0.9999 weighted** | **0.9999 weighted** | **0.9747 macro** |

These are the metrics retained in the notebook outputs, not an independently rerun benchmark. The split is a stratified random split of rows. Flows from the same hosts or time periods can therefore appear in both training and test sets; these scores do not establish performance on unseen hosts, future traffic, or another network. The classes are also highly imbalanced, so accuracy alone is not a useful summary.

## Pipeline

1. Download and load only the required columns from the CIDDS-001 archive.
2. Convert labels, byte counts, timestamps, protocol, TCP flags, and host addresses into model features.
3. Split rows into train, validation, and test sets; fit the continuous-feature transformer on training data only.
4. Build small heterogeneous host-flow subgraphs to keep memory use manageable.
5. Train a GraphSAGE model and report classification metrics and a normalized confusion matrix.

## Run

Requires Python 3.10 or newer and enough memory for the full dataset (the notebook targets a Colab runtime with roughly 12 GB RAM). From the repository root:

```bash
python -m pip install -r requirements.txt
jupyter notebook anomaly_detection_graph_CIDDS_001.ipynb
```

Run the notebook cells from top to bottom. It downloads and extracts the dataset on the first run; later runs reuse the extracted CSV. A CUDA-capable PyTorch installation can be used when available, but CPU execution is supported and may be slow.

## Dataset

The notebook downloads `CIDDS-001.zip` from the [CIDDS-001 dataset source](https://www.hs-coburg.de/wp-content/uploads/2024/11/CIDDS-001.zip). The archive is not included in this repository. Please consult the dataset provider's terms and citation guidance before redistribution or reuse.

## Limitations

- Evaluation uses a random row split rather than a chronological or host-disjoint split.
- Rare classes have relatively few test examples, so their scores have higher uncertainty.
- The notebook is an offline experiment; it does not measure inference latency, operational false-alarm rates, or robustness to distribution shift.
- The benchmark outputs were produced in Colab, and runtime/hardware details were not retained.
