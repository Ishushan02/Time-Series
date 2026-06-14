# Heart Rate Prediction Model Notes

This file summarizes the modeling workflow built in
`SignalProcessing/heartRatePrediction.ipynb`.

The goal is to predict heart rate in BPM from wrist wearable data.

```text
Input  : wrist BVP / PPG + wrist ACC
Target : ECG-derived heart-rate label
Metric : MAE in BPM
```

Important rule:

```text
Do not use ECG as an input feature.
```

ECG is the ground-truth source. The model should learn from wrist PPG and
wrist motion data, then compare predictions against `data["label"]`.

---

# 1. Dataset Used

The notebook uses the synchronized pickle files:

```text
SignalProcessing/PPG_FieldStudy/S*/S*.pkl
```

Each subject pickle contains:

```python
data.keys()
```

```text
rpeaks
signal
label
activity
questionnaire
subject
```

The most important fields are:

```python
data["signal"]["wrist"]["BVP"]
data["signal"]["wrist"]["ACC"]
data["label"]
```

For subject `S1`, the inspected shapes were:

```text
BVP      : (589568, 1)
ACC      : (294784, 3)
label    : (4603,)
activity : (36848, 1)
rpeaks   : (11431,)
```

Sampling rates:

```text
BVP / PPG : 64 Hz
ACC       : 32 Hz
Activity  : 4 Hz
Chest ECG : 700 Hz
```

The pickle files are loaded with:

```python
pickle.load(f, encoding="latin1")
```

This avoids the Python pickle decoding error.

---

# 2. Prediction Task

The supervised learning task is:

```text
Given 8 seconds of wrist BVP and wrist accelerometer data,
predict the heart rate in BPM for that window.
```

The label is:

```python
data["label"]
```

This label was created from ECG R-peaks.

---

# 3. Windowing

The dataset labels are aligned to sliding windows.

Window settings:

```text
Window length : 8 seconds
Window shift  : 2 seconds
```

Because BVP is sampled at 64 Hz:

```text
8 seconds * 64 Hz = 512 BVP samples
```

Because ACC is sampled at 32 Hz:

```text
8 seconds * 32 Hz = 256 ACC samples
```

For label index `i`:

```text
BVP start = i * 2 sec * 64 Hz
ACC start = i * 2 sec * 32 Hz
```

So each training example becomes:

```text
X[i] = {
  BVP window with 512 samples,
  ACC window with 256 samples and 3 axes
}

y[i] = heart-rate label in BPM
```

---

# 4. Exploratory Plots

The notebook plots:

```text
1. Wrist BVP / PPG
2. Wrist ACC x, y, z
3. ECG-derived heart-rate labels
4. Activity labels
```

This is used to visually check:

```text
Motion-heavy areas
Noisy PPG regions
Heart-rate changes over time
Activity transitions
```

---

# 5. Feature Extraction Baseline

Before neural networks, the notebook creates engineered features from each
8-second window.

BVP features:

```text
mean
standard deviation
minimum
maximum
peak-to-peak range
mean absolute value
RMS energy
dominant frequency
```

ACC features are computed for:

```text
ACC x
ACC y
ACC z
ACC magnitude
```

For each ACC signal, the features are:

```text
mean
standard deviation
minimum
maximum
peak-to-peak range
RMS energy
```

These features are used by both classical machine learning models and the MLP.

---

# 6. S1 Baseline Models

The notebook first trains on subject `S1` only.

This is a simpler within-subject experiment.

Models:

```text
Ridge Regression
Random Forest Regressor
Gradient Boosting Regressor
```

Split:

```text
80% train
20% test
shuffle=False
```

The notebook prints exact:

```text
MAE
RMSE
R^2
```

---

# 7. Multi-Subject Dataset

After the S1 test, the notebook loads every available subject:

```text
S1 through S15
```

For each subject:

```text
load pickle
create BVP windows
create ACC windows
extract HR labels
store subject ID as group
```

All subjects are concatenated into:

```python
BVP_ALL
ACC_ALL
y_all
activity_all
groups_all
```

---

# 8. Subject-Level Train/Test Split

The multi-subject experiment uses a group split:

```python
GroupShuffleSplit
```

The group is the subject ID.

This means entire subjects are held out for testing.

This is more realistic than randomly mixing windows, because a real model should
generalize to people it did not train on.

---

# 9. Multi-Subject Classical ML Models

The notebook trains these feature-based models:

```text
Ridge Regression
Random Forest Regressor
Gradient Boosting Regressor
```

These produce the main non-neural baseline MAEs.

All results are appended into:

```python
all_results
```

---

# 10. Raw Sequence Preparation

For neural sequence models, the notebook uses raw windows.

BVP has 512 samples per 8-second window.

ACC has 256 samples per 8-second window.

To align them:

```text
BVP is downsampled from 512 points to 256 points
```

Then the signals are concatenated:

```text
BVP_downsampled + ACC_x + ACC_y + ACC_z
```

Final neural input shape:

```text
(number_of_windows, 256, 4)
```

Meaning:

```text
256 time steps
4 channels
```

The 4 channels are:

```text
1. BVP
2. ACC x
3. ACC y
4. ACC z
```

Inputs are normalized using training data only.

The HR target is also scaled for neural-network training, then converted back to
BPM for MAE reporting.

---

# 11. 1D CNN Model

The CNN reads the raw sequence:

```text
(batch, 256, 4)
```

Architecture:

```text
Conv1D
BatchNorm
ReLU
MaxPool
Conv1D
BatchNorm
ReLU
MaxPool
Conv1D
BatchNorm
ReLU
AdaptiveAvgPool
Dense head
```

Purpose:

```text
Learn local signal patterns from BVP and ACC.
```

The notebook reports:

```text
BEST CNN raw-window model MAE
```

---

# 12. LSTM Model

The LSTM also reads:

```text
(batch, 256, 4)
```

Architecture:

```text
Bidirectional LSTM
LayerNorm
Dense layers
Dropout
Output BPM prediction
```

Purpose:

```text
Learn temporal dependencies across the 8-second window.
```

The notebook reports:

```text
BEST LSTM raw-window model MAE
```

---

# 13. Attention / Transformer Model

The Transformer uses self-attention over the 256 time steps.

Input:

```text
BVP_downsampled, ACC_x, ACC_y, ACC_z
```

Architecture:

```text
Linear input projection
Positional encoding
Transformer encoder layers
Attention pooling
Dense regression head
```

Purpose:

```text
Let the model attend to important parts of the 8-second signal window.
```

The notebook reports:

```text
BEST Transformer attention model MAE
```

---

# 14. Normal Neural Network / MLP

The MLP is a standard feed-forward neural network.

Unlike CNN, LSTM, and Transformer, it does not use the raw sequence directly.

Input:

```text
engineered feature matrix
```

Architecture:

```text
Dense layer
BatchNorm
ReLU
Dropout
Dense layer
BatchNorm
ReLU
Dropout
Dense layer
Output BPM prediction
```

Purpose:

```text
Provide a neural-network version of the classical feature baseline.
```

The notebook reports:

```text
BEST MLP feature neural network MAE
```

---

# 15. Evaluation Metrics

Every model reports:

```text
MAE
RMSE
R^2
```

The main metric is:

```text
MAE in BPM
```

MAE means:

```text
average absolute difference between predicted heart rate and true heart rate
```

Example:

```text
MAE = 5.2 BPM
```

means the model is wrong by about 5.2 beats per minute on average.

---

# 16. Final Results Table

At the end of the notebook, all model results are combined:

```python
results_df = pd.DataFrame(all_results).sort_values("mae")
```

This table ranks models from lowest MAE to highest MAE.

The final comparison includes:

```text
Ridge feature baseline
Random Forest feature baseline
Gradient Boosting feature baseline
CNN raw-window model
LSTM raw-window model
Transformer attention model
MLP feature neural network
```

To get the exact MAEs, run the notebook from top to bottom in the Jupyter
kernel.

---

# 17. Recommended Reading of the Results

Use the final MAE table like this:

```text
Lowest MAE = best model for this split
```

But compare carefully:

```text
Feature models are faster and easier to explain.
CNN/LSTM/Transformer models learn directly from signal windows.
Subject-level test split is harder and more realistic.
```

If the Random Forest or Gradient Boosting model beats the neural models, that is
not necessarily bad. It means the engineered features are strong, or the neural
models need more tuning.

If the Transformer beats CNN and LSTM, then attention is helping identify useful
time regions in the PPG and ACC window.

---

# 18. Next Improvements

Useful next experiments:

```text
1. Leave-one-subject-out validation
2. Activity-specific MAE
3. Compare BVP-only vs BVP + ACC
4. Add signal-quality features
5. Tune Transformer depth, heads, and learning rate
6. Add learning-rate scheduling
7. Train longer with early stopping
8. Save the best model weights
```

Most important next comparison:

```text
BVP only
vs
BVP + ACC
```

This will show how much motion information helps heart-rate prediction.
