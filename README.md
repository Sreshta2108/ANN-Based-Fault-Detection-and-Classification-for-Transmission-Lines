# ANN-Based Fault Detection and Classification for Transmission Lines

A complete end-to-end pipeline for detecting and classifying faults on a
three-phase transmission line using MATLAB Simulink for fault simulation
and a Python-based Artificial Neural Network (ANN) for classification.
Achieves 99.76% test accuracy across 6 fault categories.

---

## Project Overview

Transmission line faults — caused by lightning strikes, equipment failure,
or environmental factors — need to be detected and classified quickly to
prevent equipment damage and cascading grid failures. Traditional protection
relays detect faults but don't always directly classify the fault type.

This project builds a data-driven fault classifier that identifies the fault
type purely from instantaneous three-phase voltage and current readings
(Va, Vb, Vc, Ia, Ib, Ic), without any manual feature engineering.

### Fault Types Classified

| Label | Fault Type | Description |
|-------|------------|-------------|
| 0 | No Fault | Healthy system, balanced three-phase operation |
| 1 | AG | Single line-to-ground (Phase A to Ground) |
| 2 | BC | Line-to-line (Phase B to Phase C) |
| 3 | ABG | Two-line-to-ground (Phase A, B to Ground) |
| 4 | ABC | Three-phase fault (all phases shorted) |
| 5 | ABCG | Three-phase-to-ground (most severe fault) |

---

## Pipeline

**Step 1 — Simulate (MATLAB Simulink)**
Model a three-phase transmission line system and inject 6 fault types
at non-overlapping time windows using Three-Phase Fault blocks.
Capture Va, Vb, Vc, Ia, Ib, Ic continuously via measurement blocks.

**Step 2 — Extract and Label (MATLAB Script)**
Segment the captured signals by time window, assign fault type labels
(0–5), and export as fault_dataset.csv (6205 rows x 7 columns).

**Step 3 — Train ANN (Python / scikit-learn)**
Normalize features with StandardScaler, train an MLPClassifier
(32 -> 16 -> 6 neurons) on 80% of data, evaluate on remaining 20%.

**Step 4 — Classify (Inference)**
Load saved model and scaler, feed any new Va/Vb/Vc/Ia/Ib/Ic reading,
get predicted fault type in real time.

---

---

## Simulink Model

The transmission line system consists of:
- Two three-phase voltage sources feeding the line from both ends
- Two three-phase transformer banks (step-up/step-down)
- A distributed parameters transmission line (33 km, 50 Hz)
- A fault subsystem containing 6 pre-configured Three-Phase Fault blocks
- Three-phase voltage (Vabc) and current (Iabc) measurement blocks
- Scope and To Workspace block for data capture

### Fault Timing (Simulation Duration: 0.7s)

| Fault | Switching Times (s) |
|-------|---------------------|
| No Fault | 0.02 – 0.08 |
| AG | 0.10 – 0.15 |
| BC | 0.20 – 0.25 |
| ABG | 0.30 – 0.35 |
| ABC | 0.40 – 0.45 |
| ABCG | 0.50 – 0.55 |

Each fault block is active only during its assigned window. All 6 windows
are non-overlapping so there is no interference between fault types in
the captured data.

---

## Dataset

- **Total samples:** 6,205 rows
- **Features:** Va, Vb, Vc (phase voltages), Ia, Ib, Ic (phase currents)
- **Labels:** 0–5 (fault type, as above)
- **Time step:** 5x10^-5 s (Discrete solver, 20 kHz sampling)
- **Source:** MATLAB Simulink simulation

### Sample Distribution

| Fault Type | Samples |
|------------|---------|
| No Fault | 1201 |
| AG | 1001 |
| BC | 1001 |
| ABG | 1000 |
| ABC | 1001 |
| ABCG | 1001 |

---

## ANN Architecture

| Layer | Details |
|-------|---------|
| Input | 6 features (Va, Vb, Vc, Ia, Ib, Ic) |
| Hidden Layer 1 | 32 neurons, ReLU activation |
| Hidden Layer 2 | 16 neurons, ReLU activation |
| Output Layer | 6 neurons (one per fault class) |
| Optimizer | Adam |
| Max Iterations | 500 |

Input features are normalized using StandardScaler before training.

---

## Results

| Metric | Value |
|--------|-------|
| Test Accuracy | **99.76%** |
| Precision (macro avg) | 1.00 |
| Recall (macro avg) | 1.00 |
| F1-Score (macro avg) | 1.00 |

The only minor imperfection is a small overlap between ABC and ABCG
classifications, which is physically expected since both involve all
three phases — the ground component produces only a subtle difference
in the instantaneous Va/Vb/Vc/Ia/Ib/Ic signature.

---

## How to Run

### 1. Simulink — Generate Data

- Open `simulink/faultdetection.slx` in MATLAB
- Set simulation stop time to `0.7`
- Run the simulation
- Open MATLAB and run `matlab/extract_fault_data.m`
- This produces `fault_dataset.csv` in your working directory

### 2. Python — Train Classifier

Install dependencies:

```bash
pip install pandas numpy scikit-learn matplotlib seaborn joblib
```

Run the training script:

```bash
python python/fault_classifier.py
```

Or open `fault_classifier.py` in Google Colab and upload
`fault_dataset.csv` when prompted.

### 3. Test on New Data

```python
import joblib
import numpy as np

model  = joblib.load('models/fault_ann_model.pkl')
scaler = joblib.load('models/fault_scaler.pkl')

# Provide Va, Vb, Vc, Ia, Ib, Ic values
sample = np.array([[0.18, -0.59, 0.41, -55.12, 0.39, 54.74]])
prediction = model.predict(scaler.transform(sample))

labels = {0:'No_Fault', 1:'AG', 2:'BC', 3:'ABG', 4:'ABC', 5:'ABCG'}
print(f"Predicted fault type: {labels[prediction[0]]}")
```

---

## Limitations and Honest Caveats

- Trained on simulated data from one specific network topology — real
  transmission lines have noise, harmonics, and measurement error not
  present in this dataset.
- Train/test split is row-level, not event-level. Adjacent time samples
  within the same fault event are highly correlated, which inflates
  apparent accuracy slightly compared to a strict event-level split.
- Not validated against real field measurements or alternative grid
  configurations.
- Not intended for direct deployment as a protection relay — this is a
  proof-of-concept demonstrating the feasibility of ML-based fault
  classification as a potential decision-support layer alongside
  traditional protection schemes.

---

## Tools and Technologies

- MATLAB R2024a, Simulink, Simscape Electrical
- Python 3.12, scikit-learn, Pandas, NumPy, Matplotlib, Seaborn
- Google Colab

---

## Author

**kakanooru Sai Sreshta**
B.Tech, Electrical and Electronics Engineering
National Institute of Technology, Warangal
