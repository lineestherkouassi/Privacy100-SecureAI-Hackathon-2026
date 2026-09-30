# Privacy100 — Secure AI Hackathon 2026

Privacy100 is our submission for the **Intermediate Track** of the Secure AI Hackathon 2026.

The project addresses federated intrusion detection under strongly non-IID client distributions. Five banks collaboratively train an intrusion detection system without sharing their raw network traffic.

Our final approach combines a high-recall neural detector with a high-precision federated tree ensemble.

## Challenge

In the Intermediate Track, the main difficulty is that each bank observes a different distribution of normal traffic and attack families.

Under this setting, naive FedAvg became unstable and achieved:

**F1 = 0.7035**

The objective was to improve detection performance while preserving the federated-learning constraint:

> Raw client data never leaves the bank.

## Privacy100 Approach

Our final system combines two complementary federated experts.

### 1. Neural Expert

The neural branch uses:

- Residual MLP
- SCAFFOLD
- Hard-Negative Focal Loss
- Equal server aggregation

This model prioritizes attack detection and achieves very high recall.

Neural-only result:

- Precision: **0.7858**
- Recall: **0.9681**
- F1: **0.8675**

### 2. Federated ExtraTrees Expert

Each bank independently trains an ExtraTrees classifier using only its local data.

No raw samples are pooled.

The five local predictions are combined using robust trimmed aggregation:

1. sort the five attack probabilities;
2. remove the highest prediction;
3. remove the lowest prediction;
4. average the remaining three.

The ExtraTrees expert achieves lower recall but very high precision.

ExtraTrees-only result:

- Precision: **0.9666**
- Recall: **0.6366**
- F1: **0.7676**

### 3. Dual-Expert Hybrid

The final Privacy100 model exploits the complementary behavior of both experts.

The neural model acts as the primary high-recall detector, while the ExtraTrees ensemble acts as a high-precision correction mechanism.

The hybrid decision layer uses:

- neural attack prediction;
- normal-traffic veto when tree consensus strongly indicates normal traffic;
- attack rescue when the tree ensemble has very high attack confidence.

This significantly reduces false positives while maintaining high recall.

## Final Results

| Model | Precision | Recall | F1 |
|---|---:|---:|---:|
| Naive FedAvg | — | — | **0.7035** |
| Residual SCAFFOLD Neural | 0.7858 | 0.9681 | **0.8675** |
| Federated ExtraTrees | 0.9666 | 0.6366 | **0.7676** |
| Privacy100 Hybrid | **0.8710** | **0.9626** | **0.9145** |

Final confusion matrix:

| Metric | Count |
|---|---:|
| True Positives | 12,353 |
| False Positives | 1,829 |
| False Negatives | 480 |
| True Negatives | 7,882 |

Privacy100 therefore improves F1 from:

**0.7035 → 0.9145**

Absolute improvement:

**+0.2110 F1**

## Repository Structure

```text
Privacy100-SecureAI-Hackathon-2026/
│
├── README.md
├── model_scripted.pt
├── submission.json
├── requirements.txt
├── PrivacyTeam_Intermediate_Day2.ipynb

   
```

## Files

### `model_scripted.pt`

TorchScript export of the final Privacy100 hybrid model.

It contains:

- the neural Residual MLP;
- the converted federated ExtraTrees models;
- trimmed tree aggregation;
- veto and rescue decision logic.

The model expects:

```text
41 input features
```

and returns one binary classification logit per sample.

### `submission.json`

Contains the submission metadata and self-reported metrics.

Example:

```json
{
  "team_name": "Privacy100",
  "track": "intermediate",
  "self_reported_metrics": {
    "precision": 0.871,
    "recall": 0.9626,
    "f1": 0.9145
  },
  "model_file": "model_scripted.pt",
  "n_input_features": 41
}
```

### `PrivacyTeam_Intermediate_Day2.ipynb`

The notebook contains the complete experimental pipeline:

1. NSL-KDD preprocessing;
2. deterministic non-IID partitioning across five banks;
3. naive FedAvg baseline;
4. residual neural architecture;
5. SCAFFOLD federated training;
6. Hard-Negative Focal Loss;
7. local ExtraTrees training;
8. robust trimmed aggregation;
9. dual-expert hybrid decision logic;
10. final evaluation;
11. TorchScript export.

## Setup

Recommended environment:

- Python 3.x
- PyTorch
- NumPy
- Pandas
- scikit-learn

Install the main dependencies with:

```bash
pip install torch numpy pandas scikit-learn
```

## Reproducing the Results

Open:

```text
notebooks/Privacy100_Intermediate.ipynb
```

Run the notebook from top to bottom.

The preprocessing and non-IID client split must be executed before the custom Privacy100 training cell.

The final model should produce approximately:

```text
Precision: 0.8710
Recall:    0.9626
F1:        0.9145
```

The final submission cell exports:

```text
model_scripted.pt
submission.json
```

## Federated Learning Constraint

Privacy100 does not pool raw client traffic.

Each bank trains locally on its own data.

The server only operates on model updates, model outputs, or aggregated predictions depending on the component of the system.

This preserves the central privacy constraint of the federated-learning scenario.

## Key Takeaway

Our main finding is that non-IID federated intrusion detection was not solved by simply increasing model complexity.

The strongest improvement came from combining two models with complementary behaviors:

- the neural detector provides high sensitivity;
- the federated ExtraTrees ensemble provides conservative high-precision confirmation.

The disagreement between both experts is therefore treated as useful information rather than noise.

## Team

**Team:** Privacy100  
**Country:** Côte d’Ivoire  
**Track:** Intermediate  
**Competition:** Secure AI Hackathon 2026
