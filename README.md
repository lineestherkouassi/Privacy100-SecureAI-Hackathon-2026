# Privacy100 — Secure AI Hackathon 2026

Privacy100 is our federated intrusion-detection submission for the **Secure AI Hackathon 2026**.

Our work covers both the **Intermediate** and **Advanced** Day 2 tracks, with **Advanced** as our primary submitted track.

The project studies two problems in federated learning across five banks:

1. strongly non-IID client distributions;
2. malicious clients sending poisoned model updates.

Raw client traffic is never centrally pooled.

## Main Results

### Intermediate — Non-IID Federated Learning

Naive FedAvg on the non-IID split:

**F1 = 0.7264 ± 0.0216**

Our normalized server aggregation alone:

**F1 = 0.7612 ± 0.0195**

Full skew-aware pipeline with normalized aggregation and locally balanced loss:

**F1 = 0.7836 ± 0.0067**

IID FedAvg reference:

**F1 = 0.7654 ± 0.0121**

### Advanced — Poisoning Defense

Main attack scenario:

- 5 non-IID banks;
- client 1 malicious;
- local label flipping;
- update amplification ×15;
- 8 federated rounds;
- evaluation over seeds `[42, 43, 44, 45, 46]`.

Naive FedAvg under attack:

**F1 = 0.3092 ± 0.1781**

Privacy100 KrumShield:

**F1 = 0.7629 ± 0.0171**

Overall F1 recovery:

**+0.4536**

For a controlled comparison using the same locally balanced training:

**FedAvg = 0.5202 ± 0.0910**

**KrumShield = 0.7629 ± 0.0171**

Controlled improvement:

**+0.2426 F1**

## KrumShield

KrumShield is a robust federated aggregation and malicious-client attribution mechanism.

It uses four main stages:

1. **Median-norm clipping**  
   Large updates cannot dominate the federation simply through magnitude.

2. **Directional consistency**  
   Client updates are compared using cosine similarity after clipping.

3. **Temporal reputation**  
   An exponential moving average tracks whether each client remains consistent with the federation across rounds.

4. **Selective aggregation**  
   Persistently suspicious clients are excluded and the remaining updates are normalized before aggregation.

This allows KrumShield to both resist poisoning and identify suspicious participants.

## Malicious-Client Detection

For the primary one-attacker ×15 scenario:

- round-level detection precision: **1.00**
- round-level detection recall: **0.575**
- attacker identified: **5/5 sessions**
- sessions with false accusations: **0/5**

## Comparison with Robust Baselines

| Defense | F1 with balanced local training |
|---|---:|
| FedAvg | 0.5202 ± 0.0910 |
| Norm clipping | 0.7372 ± 0.0254 |
| Krum | 0.7214 ± 0.0163 |
| Multi-Krum | 0.7307 ± 0.0196 |
| Coordinate median | 0.7354 ± 0.0258 |
| Trimmed mean | 0.7397 ± 0.0216 |
| Median clipping only | 0.7623 ± 0.0198 |
| **KrumShield** | **0.7629 ± 0.0171** |

KrumShield is approximately tied with median clipping on F1.

Its main additional contribution is **attack attribution**, not simply classification accuracy.

## Robustness Tests

KrumShield was also evaluated under:

- ×50 update amplification;
- label-flip without amplification;
- two malicious clients.

For one malicious client, KrumShield maintained approximately **0.7629 F1** under both amplified and non-amplified attacks and identified the attacker in all five seeds without false accusations.

With two colluding malicious clients, classification performance remained strong, but the pairwise attribution mechanism failed to identify the attackers.

## KrumShield-R Extension

We explored a second version, **KrumShield-R**, to address colluding attackers.

Instead of measuring consistency only between clients, KrumShield-R compares each client update with a server-side reference direction generated from approximately 200 labeled samples.

This restored two-attacker attribution in our experiments.

However, this extension requires an additional trusted-data assumption. Experiments with biased reference sets showed that the choice of the reference data can also introduce false accusations.

KrumShield-R is therefore included as an experimental extension rather than the primary submitted model.

## Exported Model

The repository includes:

`model_scripted.pt`

This is the final TorchScript model produced by the notebook for the Advanced track.

The exported seed-42 model achieves:

- Precision: **0.9363**
- Recall: **0.6338**
- F1: **0.7559**

The five-seed average of the KrumShield defense is:

**0.7629 ± 0.0171**

The difference exists because the exported artifact is a single deterministic seed-42 model, while the primary experimental result reports the mean across five seeds.

## Repository Contents

```text
Privacy100-SecureAI-Hackathon-2026/
├── README.md
├── Day2_Advanced_Privacy100.ipynb
├── model_scripted.pt
├── submission.json
├── cover.png
└── requirements.txt
```

## Setup

Python 3 is required.

Install dependencies with:

```bash
pip install torch scikit-learn pandas numpy scipy matplotlib
```

## Reproducing the Results

Open:

```text
Day2_Advanced_Privacy100.ipynb
```

Run every cell from top to bottom.

The notebook:

- downloads and preprocesses NSL-KDD;
- creates IID and non-IID five-bank partitions;
- evaluates the Intermediate aggregation strategies;
- simulates malicious clients;
- evaluates Byzantine-robust aggregation baselines;
- trains and evaluates KrumShield over five seeds;
- performs robustness and sensitivity tests;
- exports `model_scripted.pt`;
- generates `submission.json`.

## Submission Artifacts

### `model_scripted.pt`

TorchScript export of the final Advanced model.

Input:

```text
41 standardized NSL-KDD features
```

Output:

```text
one binary classification logit per connection
```

### `submission.json`

Contains:

- team name;
- selected track;
- exported model metrics;
- main poisoning scenario;
- five-seed baseline and defense results;
- recovered F1;
- malicious-client detection metrics;
- KrumShield-R extension results.

## Privacy Model

No raw client traffic is pooled during federated training.

Each bank performs local optimization and sends only its trained model/update to the aggregation process.

## Limitations

KrumShield's original pairwise-consistency mechanism is designed primarily for the one-malicious-client scenario.

With two colluding malicious clients it maintains classification performance but can fail to attribute the attackers.

KrumShield-R addresses this limitation at the cost of requiring a small labeled server-side reference set.

## Team

**Privacy100**  
**Côte d’Ivoire**  
**Primary Track: Advanced**  
**Additional Track Attempted: Intermediate**
