# PPG Field Study Dataset - Complete Notes

# 1. Introduction

## What is this Dataset?

The **PPG Field Study Dataset** is a publicly available physiological signal dataset designed for **Heart Rate (HR) Estimation** using **Photoplethysmography (PPG)** signals collected from a wrist-worn wearable device.

The primary goal is to build Machine Learning (ML) and Deep Learning (DL) models that can estimate a person's heart rate from wearable sensor data, especially under real-world conditions involving motion and physical activity.

Unlike many existing datasets collected in laboratory environments, this dataset contains data recorded during realistic daily-life activities such as:

* Sitting
* Walking
* Climbing stairs
* Cycling
* Driving
* Working
* Eating lunch
* Playing table soccer

This makes the dataset significantly more challenging and useful for real-world applications.

---

# 2. Full Forms

| Abbreviation | Full Form                        |
| ------------ | -------------------------------- |
| PPG          | Photoplethysmography             |
| ECG          | Electrocardiogram                |
| HR           | Heart Rate                       |
| BPM          | Beats Per Minute                 |
| ACC          | Accelerometer                    |
| BVP          | Blood Volume Pulse               |
| EDA          | Electrodermal Activity           |
| RESP         | Respiration Signal               |
| EMG          | Electromyography                 |
| TEMP         | Temperature                      |
| RR Interval  | Time between consecutive R-peaks |
| Hz           | Hertz (samples per second)       |

---

# 3. Dataset Statistics

## Subjects

* Total Subjects: 15
* Male Subjects: 7
* Female Subjects: 8

Average age:

30.60 ± 9.59 years

Each recording lasts approximately:

2.5 hours

---

# 4. Dataset Directory Structure

Each subject has a separate folder.

Example:

```text
S1/
│
├── S1_quest.csv
├── S1_activity.csv
├── S1_RespiBAN.h5
├── S1_E4.zip
└── S1.pkl
```

Where:

### S1_quest.csv

Contains subject metadata:

```text
Age
Gender
Height
Weight
Skin Type
Fitness Level
```

### S1_activity.csv

Contains activity labels and timestamps.

### S1_RespiBAN.h5

Contains chest sensor data.

### S1_E4.zip

Contains wrist sensor data.

### S1.pkl

Contains synchronized signals and labels.

This is the most important file for ML training.

---

# 5. Data Collection Devices

Two devices were used simultaneously.

## Device 1: RespiBAN Professional

Location:Chest
Purpose: Ground-truth physiological measurements.
Sampling Rate: 700 Hz
Meaning: 700 samples recorded every second.

### Signals Recorded

#### ECG

Electrocardiogram
Measures electrical activity of the heart.
Used to generate heart-rate labels.

#### RESP
Respiration Signal
Measures breathing pattern.

#### ACC
Accelerometer
Measures body movement.

#### EDA
Electrodermal Activit
Measures skin conductivity.
Not useful in this dataset.

#### EMG

Electromyography
Measures muscle activity.
Dummy signal in this dataset.

#### TEMP

Temperature
Dummy signal in this dataset.

---

## Device 2: Empatica E4 Wristband

Location: Non-dominant wrist

### Signals Recorded

#### ACC

Accelerometer 
Sampling Rate: 32 Hz

Measures wrist movement along:
* X-axis
* Y-axis
* Z-axis

---

#### BVP

Blood Volume Pulse

Sampling Rate: 64 Hz
This is the raw PPG signal.
Most important signal for heart-rate estimation.

---

#### EDA

Electrodermal Activity

Sampling Rate: 4 Hz
Measures sweat gland activity.

---

#### TEMP

Temperature
Sampling Rate: 4 Hz
Measures skin temperature.

---

# 6. Understanding PPG

## What is PPG?

PPG stands for:

Photoplethysmography: It measures blood volume changes using light.

Process:

```text
Heart Beats
      ↓
Blood Flows
      ↓
Blood Volume Changes
      ↓
Light Reflection Changes
      ↓
PPG Sensor Measures Signal
```

The PPG signal is stored as:

```python
data['signal']['wrist']['BVP']
```

---

# 7. Understanding ECG

## What is ECG?

ECG stands for:

Electrocardiogram: It measures electrical activity of the heart.
Unlike PPG, ECG directly observes heart electrical signals.
ECG is considered the ground truth.

Stored in:

```python
data['signal']['chest']['ECG']
```

---

# 8. ECG Waveform

A heartbeat produces a characteristic waveform.

```text
               R
              /\
             /  \
       P    /    \    T
      /\   /      \  /\
     /  \_/        \/  \
         Q          S
```

---

## P Wave

P = Atrial Depolarization

Meaning:

Electrical activation of the atria.

Result:

Atria contract.

---

## Q Wave

First negative deflection.

Beginning of ventricular activation.

---

## R Wave

Largest positive peak.

Represents major ventricular activation.

Most important point in ECG.

---

## S Wave

Negative deflection after R.

Final stage of ventricular activation.

---

## T Wave

Ventricular Repolarization.

Heart prepares for the next beat.

---

# 9. What are R-Peaks?

R-peaks are the tallest peaks in ECG.

```text
        R
       /\
      /  \
```

Each R-peak corresponds to one heartbeat.

Stored as:

```python
data['rpeaks']
```

Example:

```python
[120, 840, 1560, 2275]
```

These values are ECG sample indices.

---

# 10. RR Interval

RR Interval:

Time between consecutive R-peaks.

```text
      R1                 R2
      /\                 /\
-----/--\---------------/--\-----

      <--- RR Interval --->
```

Formula:

```text
RR = R2_time - R1_time
```

---

# 11. Heart Rate Calculation

Heart Rate is derived from RR intervals.

Formula:

```text
HR = 60 / RR
```

Where:

* HR = Heart Rate (BPM)
* RR = RR Interval (seconds)

Example:

```text
RR = 0.8 sec

HR = 60 / 0.8

HR = 75 BPM
```

---

# 12. Data Synchronization

The two devices have independent clocks.

Therefore synchronization is required.

Method used:

Double-tap gesture.

Subjects tapped the wrist device against their chest.

The resulting acceleration pattern appears in both devices.

This pattern is used for alignment.

Process:

```text
Empatica E4
      ↓
Double Tap
      ↓
RespiBAN
      ↓
Match Peaks
      ↓
Synchronize Signals
```

---

# 13. Sliding Window Label Generation

Heart-rate estimation is performed using fixed windows.

Window Length:

8 seconds

Window Shift:

2 seconds

Example:

```text
0 sec ---------- 8 sec

2 sec ---------- 10 sec

4 sec ---------- 12 sec
```

Each window receives one HR label.

---

# 14. What is data['label']?

This is the target variable.

Stored as:

```python
data['label']
```

Contains:

Heart Rate (BPM)

generated from ECG.

Example:

```python
[74.2, 75.1, 78.3, 80.4]
```

Each value corresponds to one 8-second window.

---

# 15. Structure of data['signal']

```python
data['signal']
```

Contains:

```python
{
    'chest',
    'wrist'
}
```

---

## Wrist Signals

```python
data['signal']['wrist']
```

Contains:

```python
{
    'ACC',
    'BVP',
    'EDA',
    'TEMP'
}
```

---

## Chest Signals

```python
data['signal']['chest']
```

Contains:

```python
{
    'ACC',
    'ECG',
    'RESP',
    'EDA',
    'EMG',
    'TEMP'
}
```

---

# 16. Activity Labels

Stored in:

```python
data['activity']
```

Activities:

| ID | Activity     |
| -- | ------------ |
| 0  | Transition   |
| 1  | Sitting      |
| 2  | Stairs       |
| 3  | Table Soccer |
| 4  | Cycling      |
| 5  | Driving      |
| 6  | Lunch Break  |
| 7  | Walking      |
| 8  | Working      |

---

# 17. Structure of SX.pkl

```python
data.keys()
```

returns:

```python
{
    'activity',
    'label',
    'questionnaire',
    'rpeaks',
    'signal',
    'subject'
}
```

---

# 18. Machine Learning View

Inputs:

```python
PPG = data['signal']['wrist']['BVP']
ACC = data['signal']['wrist']['ACC']
```

Target:

```python
HR = data['label']
```

Training Pipeline:

```text
PPG + ACC
      ↓
Windowing
      ↓
Feature Extraction
      ↓
Neural Network
      ↓
Predicted Heart Rate
      ↓
Loss Calculation
      ↓
Model Update
```

---

# 19. Why ACC is Important

Motion introduces noise into PPG.

Examples:

* Walking
* Running
* Cycling
* Climbing Stairs

Motion causes:

```text
Motion
   ↓
PPG Distortion
   ↓
Incorrect Heart Rate
```

ACC helps identify and remove motion artifacts.

---

# 20. Final Dataset Summary

The dataset provides:

* Wrist PPG signals
* Wrist accelerometer signals
* Chest ECG signals
* Ground-truth heart rate labels
* Subject metadata
* Activity labels

The primary supervised learning task is:

```text
Wrist PPG + Wrist ACC
          ↓
Estimate Heart Rate
          ↓
Compare with ECG Ground Truth
```

In practical terms:

```python
X = [BVP, ACC]
y = HeartRateLabel
```

This is one of the most widely used datasets for research in wearable heart-rate estimation and motion-artifact-robust physiological signal processing.
