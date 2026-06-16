These are the main artifacts that could arrise when a Human/Person is wearing a wrist band and it's PPG is recorded

## 1. Motion Artifacts (MA)

The biggest offender. Any wrist/hand movement displaces the sensor relative to the skin, changing the optical path. This includes hand gestures during conversation, typing on a keyboard, cooking, eating, brushing teeth — basically any activity involving the hands. The artifact can range from mild baseline shifts to complete signal destruction.

## 2. Pressure Artifacts

When the wristband gets pressed against something — leaning your wrist on a desk, resting your chin on your hand, gripping a steering wheel tightly — it changes how firmly the sensor contacts the skin. Too much pressure occludes blood flow, too little creates an air gap. Both corrupt the signal.

## 3. Loose Band / Coupling Artifacts

If the band is worn loosely, it bounces on the skin with every movement. This intermittent contact creates sudden spikes and dropouts. It's especially common when people wear the band over the wrist bone rather than a fleshy area.

## 4. Ambient Light Artifacts

PPG is an optical measurement. External light sources can leak in and contaminate the photodetector reading — walking outside in bright sunlight, flickering fluorescent lights indoors, or even the flash from a phone screen. This adds a noise floor or periodic interference.

## 5. Temperature-Related Artifacts

Cold environments cause vasoconstriction — blood vessels in the wrist shrink, reducing the pulsatile signal amplitude dramatically. The sensor essentially loses the signal. Conversely, extreme heat causes vasodilation which can saturate the sensor. Transitioning between indoor and outdoor environments creates slow drifts.

## 6. Sweat / Moisture Artifacts

During exercise or in humid conditions, sweat accumulates under the sensor. This changes the optical coupling properties of the skin-sensor interface, introduces scattering, and can cause the band to slide more easily — compounding motion artifacts.

## 7. Skin Tone / Pigmentation Variability

Not a transient artifact but a persistent bias. Darker skin absorbs more light, reducing the signal-to-noise ratio. Tattoos under the sensor can completely block the optical signal. This isn't something that "appears and disappears" but it affects baseline signal quality.

## 8. Venous Pulsation Artifacts

The PPG ideally captures arterial pulsation only. But veins also have minor pulsations, and when the wrist is positioned below the heart (arm hanging down), venous pooling and pulsation increase. This adds a secondary waveform that distorts the true cardiac signal.

## 9. Respiration Artifacts

Breathing causes slow, periodic modulation of blood pressure and stroke volume. This shows up as a low-frequency (~0.15–0.4 Hz) baseline wander in the PPG. During deep or heavy breathing (exercise, anxiety), this becomes significant and can interfere with peak detection.

## 10. Muscle Contraction / Tendon Movement Artifacts

Gripping, clenching, or flexing the wrist causes tendons and muscles under the sensor to shift, physically deforming the tissue and altering blood volume readings. Think about gripping handlebars on a bike or squeezing a stress ball.

## 11. Electrical / Electromagnetic Interference (EMI)

Nearby electronics, MRI machines, or even poorly shielded sensor circuits can inject high-frequency noise into the signal. Less common in daily life but relevant in clinical or industrial settings.

## 12. Posture / Hydrostatic Artifacts

Raising your arm above your head versus letting it hang by your side changes the hydrostatic pressure in your wrist. Blood drains from the wrist when the arm is raised, reducing PPG amplitude. Frequent arm position changes create slow amplitude modulations.

## Quick Summary

| # | Artifact Type | Frequency Range | Severity |
|---|---|---|---|
| 1 | Motion | Broadband | Very High |
| 2 | Pressure | Low freq | High |
| 3 | Loose band | Broadband/spikes | High |
| 4 | Ambient light | DC or periodic | Moderate |
| 5 | Temperature | Very low / drift | Moderate |
| 6 | Sweat/moisture | Slow drift | Moderate |
| 7 | Skin tone/tattoo | DC (persistent) | Variable |
| 8 | Venous pulsation | ~cardiac freq | Low-Moderate |
| 9 | Respiration | 0.15–0.4 Hz | Low-Moderate |
| 10 | Muscle/tendon | Broadband | Moderate |
| 11 | EMI | High freq | Low |
| 12 | Posture/hydrostatic | Very low | Low-Moderate |

In a real application, you're almost never dealing with just one : they stack on top of each other. Someone jogging outside on a cold day? That's motion + sweat + temperature + respiration + loose band all happening simultaneously.

---

# Artifact Removal — Methods

The following sections document every denoising method implemented in `artifactPredictions.ipynb`. The shared task across all methods is **signal denoising**: given a noisy wrist BVP window `X`, recover the clean BVP window `Y`. Heart-rate labels are kept in the dataset but not used for denoising.

---

## Dataset & Preprocessing Pipeline

### Source Data
- **Dataset:** PPG-Dalia, 15 subjects (S1–S15).
- **Signal used:** Wrist photoplethysmography (BVP) at **64 Hz**.
- **Activity signal:** 4 Hz (used to modulate motion-correlated artifact strength).
- **Heart-rate labels:** 0.5 Hz ground truth (2-second label rate → 2-second windows).

### Windowing
Each subject's signals are cut into non-overlapping **2-second windows**:
- BVP window: **128 samples** (64 Hz × 2 s)
- Activity window: **8 samples** (4 Hz × 2 s)
- One HR label per window

The minimum of (number of labels, BVP windows, activity windows) is used per subject, yielding **64,697 total samples** across all 15 subjects.

### Artifact Simulation
Twelve artifact types are synthetically added to each clean BVP window using parameterised noise generators. All parameters (burst locations, amplitudes, frequencies, signs) are **randomly sampled per window** using a deterministic seed derived from `subject_index × 1,000,000 + sample_index`, ensuring reproducibility while giving each window a unique noise realisation.

| # | Artifact | Generation method |
|---|---|---|
| 1 | Motion | Gaussian noise scaled by activity envelope |
| 2 | Pressure | Gaussian burst mask at random time centres |
| 3 | Loose band coupling | Narrow burst mask × random ±1 sign |
| 4 | Ambient light | Sinusoid (0.8–1.3 Hz) + DC offset |
| 5 | Temperature drift | Linear ramp (random sign, 15–35% of signal range) |
| 6 | Sweat/moisture | Low-freq sinusoid + smoothed Gaussian noise |
| 7 | Skin tone/pigmentation | DC shift + scaled inverse of clean signal |
| 8 | Venous pulsation | Sinusoid near cardiac frequency (0.8–1.3 Hz) |
| 9 | Respiration | Sinusoid (0.15–0.4 Hz) |
| 10 | Muscle/tendon movement | Laplace noise scaled by activity envelope |
| 11 | EMI | High-frequency sinusoid (15–24 Hz) |
| 12 | Posture/hydrostatic | Very low-frequency sinusoid (0.02–0.06 Hz) |

All 12 artifacts are summed and added to the clean signal:

```
X = Y + Σ artifact_k
```

### Dataset Arrays
```
X      (64697, 128)  — noisy BVP input
Y      (64697, 128)  — clean BVP target
LABEL  (64697,)      — heart-rate ground truth (not used in denoising)
```

### Train / Val / Test Split
Random shuffle with seed 123, then:
- **Train:** 70% → 45,287 samples
- **Val:** 15% → 9,705 samples
- **Test:** 15% → 9,705 samples

### Standardisation
All neural methods standardise inputs using **training-set statistics only** (mean and std of `X_train`), computed once and applied to val/test:
```
x_standardised = (x − x_mean) / x_std
```
Outputs are un-standardised before computing MSE/MAE.

### Evaluation Metrics
- **MSE** — mean squared error between denoised output and clean target
- **MAE** — mean absolute error between denoised output and clean target

---

## Method 1: EMD + LMS Adaptive Filtering

### Overview
A classical signal-processing pipeline that requires no training data. It decomposes the noisy signal into intrinsic mode functions (IMFs), identifies the noisy ones by their zero-crossing rate, and uses LMS adaptive filtering to cancel the estimated noise.

### Step 1 — Empirical Mode Decomposition (EMD)
A custom EMD implementation decomposes the input into IMFs via iterative sifting:
1. Find all local maxima and minima using `scipy.signal.find_peaks`.
2. Fit cubic spline envelopes through the maxima (upper) and minima (lower).
3. Subtract the mean envelope from the signal: `h = h − 0.5*(upper + lower)`.
4. Repeat until the sifting change `‖hprev − h‖² / ‖hprev‖²` drops below `stop_threshold = 0.05`.
5. Subtract the resulting IMF from the residual and repeat for up to `max_imfs = 5` IMFs.

### Step 2 — Noise IMF identification
IMFs with **zero-crossing rate ≥ median ZCR** across all IMFs are classified as noisy. Their sum is used as the noise reference signal for the adaptive filter.

### Step 3 — LMS Adaptive Noise Cancellation
A normalised LMS filter of order 8 runs sample-by-sample:
```
estimated_noise[n] = w · x_ref[n:n+order]
denoised[n]        = primary[n] − estimated_noise[n]
w                 += (μ / (x_ref·x_ref + ε)) × denoised[n] × x_ref
```
`μ = 0.05`, `ε = 1e-8`.

### Variants
| Variant | Description |
|---|---|
| `"emd"` | Single EMD pass on the noisy signal |
| `"eemd"` | Ensemble of EMD runs with added white noise (8 members), averaged |
| `"ceemdan"` | Paired ±noise ensemble (16 members), averaged |

### Notes
- Computationally expensive (no GPU).
- In the benchmark example, the ground-truth noise `NOISE_test` is passed as the LMS reference — this is an upper-bound demonstration, not a real deployment scenario.
- In the blind setting (`reference_noise=None`), the EMD-estimated noise is used instead.

---

## Method 2: CycleGAN (Unpaired)

### Overview
An **unpaired** image-to-image translation approach adapted for 1D signals. Two generators and two discriminators learn to translate between the noisy and clean signal domains without requiring paired examples at training time.

### Architecture

**Generators** (`CycleGenerator1D`) — shared structure for both G_noisy→clean and G_clean→noisy:
```
Conv1d(1→64, k=7) → InstanceNorm → ReLU
→ 4 × ResBlock1D(64)
→ Conv1d(64→1, k=7)
```
Each `ResBlock1D` is two `Conv1d(64→64, k=7)` layers with `InstanceNorm` and a skip connection.

**Discriminators** (`CycleDiscriminator1D`) — PatchGAN-style:
```
Conv1d(1→64,  k=7, stride=2) → LeakyReLU(0.2)
Conv1d(64→128, k=7, stride=2) → InstanceNorm → LeakyReLU(0.2)
Conv1d(128→256,k=7, stride=2) → InstanceNorm → LeakyReLU(0.2)
Conv1d(256→1,  k=7)
```
Output is a patch-level score map (shape ~`(B, 1, 16)`), not a single scalar.

### Loss
LSGAN (MSE adversarial loss) with cycle-consistency and identity terms:
```
L_G = L_adv(G_N→C) + L_adv(G_C→N)
    + λ_cycle × [L1(G_C→N(G_N→C(x_noisy)), x_noisy)
               + L1(G_N→C(G_C→N(x_clean)), x_clean)]
    + λ_id    × [L1(G_N→C(x_clean), x_clean)
               + L1(G_C→N(x_noisy), x_noisy)]

λ_cycle = 10,  λ_id = 2
```

### Training
- **Dataset:** `UnpairedSignalDataset` — clean index randomised per `__getitem__` call
- **Optimizer:** Adam, lr=2e-4, betas=(0.5, 0.999)
- **Epochs:** 30, batch size 128
- **Inference:** `G_noisy_to_clean` is used at test time

---

## Method 3: FCGAN (Fully Convolutional Conditional GAN, Paired)

### Overview
A **paired** conditional GAN (analogous to Pix2Pix) where the discriminator sees the `(noisy, clean_or_generated)` pair. The generator learns to predict and subtract the artifact noise residual.

### Generator (`FCGANGenerator1D`)
Flat stack of three `Conv1d` blocks followed by a residual subtraction:
```
Conv1d(1→64, k=9) → BN → ReLU
Conv1d(64→64, k=9) → BN → ReLU
Conv1d(64→64, k=9) → BN → ReLU
Conv1d(64→1,  k=9)   ← predicts residual noise
output = noisy_input − predicted_residual
```

### Discriminator (`FCGANDiscriminator1D`)
Takes the concatenated `(noisy, clean_or_generated)` pair as a 2-channel input:
```
Conv1d(2→64,   k=7, stride=2) → LeakyReLU(0.2)
Conv1d(64→128, k=7, stride=2) → BN → LeakyReLU(0.2)
Conv1d(128→256,k=7, stride=2) → BN → LeakyReLU(0.2)
Conv1d(256→1,  k=7)
```

### Loss
```
L_G = L_adv(D(noisy, G(noisy)), 1)  +  λ_L1 × L1(G(noisy), clean)
L_D = 0.5 × [L_adv(D(noisy, clean), 1) + L_adv(D(noisy, G(noisy).detach()), 0)]

λ_L1 = 100
```

### Training
- **Dataset:** `PairedSignalDataset` (matched noisy-clean pairs)
- **Optimizer:** Adam, lr=2e-4, betas=(0.5, 0.999)
- **Epochs:** 30, batch size 128

---

## Method 4: VQVAE (Vector-Quantised Variational Autoencoder)

### Overview
A **paired** encoder-decoder with a **discrete vector-quantised bottleneck**. The model learns a codebook of prototype latent vectors; during the forward pass, each encoded position is snapped to its nearest codebook entry via a straight-through estimator.

### Architecture (`VQVAEPPGDenoiser1D`)

**Encoder** (input `(B, 1, 128)` → `(B, embedding_dim, 32)`):
```
Conv1d(1→H,  k=7) → BN → ReLU
Conv1d(H→H,  k=4, stride=2) → BN → ReLU       # 128→64
Conv1d(H→H,  k=4, stride=2) → BN              # 64→32
2 × ResidualBlock1D(H)
ReLU
Conv1d(H→embedding_dim, k=1)
```
`H = hidden_channels = 32`

**VQ Layer** (`VectorQuantizeSignal`):
- Codebook: `nn.Embedding(codebook_dim=64, embedding_dim=32)`
- Each encoded position finds its nearest codebook vector by L2 distance.
- Straight-through gradient: `quantized = encoded + (nearest − encoded).detach()`
- Loss: `L_codebook + commitment_cost × L_commitment`, with `commitment_cost = 0.25`
- Tracks codebook perplexity to monitor codebook utilisation.

**Decoder** (`(B, embedding_dim, 32)` → `(B, 1, 128)`):
```
Conv1d(embedding_dim→H, k=3) → BN
2 × ResidualBlock1D(H)
ReLU
ConvTranspose1d(H→H, k=4, stride=2) → BN → ReLU   # 32→64
ConvTranspose1d(H→H, k=4, stride=2) → BN → ReLU   # 64→128
Conv1d(H→1, k=7)
```
An `F.interpolate` guard corrects any length mismatch after upsampling.

### Loss
```
L = L1(denoised, clean) + L_codebook + commitment_cost × L_commitment
```

### Training
- **Optimizer:** Adam, lr=2e-4
- **Gradient clipping:** max norm 1.0
- **Early stopping:** patience=6 epochs on val MSE, min_delta=1e-4
- **Checkpointing:** best model saved to `denoising_checkpoints/best_vqvae_ppg.pt`
- **Epochs:** up to 30, batch size 128

---

## Method 5: 1D U-Net

### Overview
A symmetric **encoder-decoder with skip connections** that directly bridge each encoder level to its corresponding decoder level. Unlike FCGAN (no lateral connections) and VQVAE (discrete bottleneck breaks spatial correspondence), the U-Net passes fine-grained local structure around the bottleneck via channel concatenation — preserving PPG peak morphology that artifact noise corrupts.

### Architecture (`UNet1DPPGDenoiser`)

```
Input (B, 1, 128)
  │
  ▼ enc1: Conv1d(1→32, k=7) → BN → ReLU           (B, 32, 128)  ──────────────────┐ skip
  ▼ enc2: Conv1d(32→64, k=4, stride=2) → BN → ReLU (B, 64,  64)  ─────────────┐   │
  ▼ enc3: Conv1d(64→128, k=4, stride=2) → BN → ReLU(B,128,  32)  ─────────┐   │   │
  ▼ enc4: Conv1d(128→256, k=4, stride=2)→ BN → ReLU(B,256,  16)           │   │   │
  ▼ bottleneck: Conv1d(256→256, k=3) → BN → ReLU   (B,256,  16)           │   │   │
  ▼ up3: ConvTranspose1d(256→128, k=4, stride=2)     (B,128,  32)          │   │   │
  ▼ cat(enc3 skip) → Conv1d(256→128, k=3) → BN → ReLU ──────────────────── ┘   │   │
  ▼ up2: ConvTranspose1d(128→64, k=4, stride=2)      (B, 64,  64)              │   │
  ▼ cat(enc2 skip) → Conv1d(128→64,  k=3) → BN → ReLU ──────────────────────── ┘   │
  ▼ up1: ConvTranspose1d(64→32, k=4, stride=2)       (B, 32, 128)                  │
  ▼ cat(enc1 skip) → Conv1d(64→32,   k=3) → BN → ReLU ────────────────────────────  ┘
  ▼ output_conv: Conv1d(32→1, k=7)                   (B,  1, 128)
```

Skip connections are **concatenation** (doubles channels), followed by a fuse convolution to reduce back to the decoder width.

### Loss & Training
- **Loss:** L1 reconstruction
- **Optimizer:** Adam, lr=1e-3
- **Scheduler:** `ReduceLROnPlateau` — halves lr after 3 stagnant val-MSE epochs, floor at 1e-5
- **Gradient clipping:** max norm 1.0
- **Early stopping:** patience=6 on val MSE
- **Checkpointing:** `denoising_checkpoints/best_unet_ppg.pt`
- **Epochs:** up to 30, batch size 128

---

## Method 6: DPNet — Bidirectional Mamba (SSM)

### Overview
Implementation of **DPNet** from the paper (fig. 1). The core novelty is replacing convolutional or attention-based sequence modelling with a **selective State Space Model (Mamba)** run in both forward and backward directions. A learnable weighted residual blends the model output with the original noisy signal.

### Overall Architecture (`DPNet`)
```
Noisy PPG (B, 1, 128)
  │
  ▼ Conv(1→64, k=7) → BN → SiLU
  ▼ Conv(64→d_model, k=3) → BN → SiLU      ← input projection
  │
  ▼ Transpose → (B, L, d_model)
  │
  ▼ × n_blocks: BidirectionalMambaBlock
  │
  ▼ Transpose → (B, d_model, L)
  │
  ▼ Conv(d_model→64, k=3) → BN → SiLU
  ▼ Conv(64→1, k=7)                         ← output projection
  │
  ▼ output = sigmoid(α) × model_out + (1−sigmoid(α)) × noisy_input
             ↑ α is a learnable scalar parameter, initialised at 0.9
```
Default: `n_blocks=5, d_model=64, d_state=8, expand=2` (paper uses `d_model=128`).

### Mamba Layer (`MambaLayer`, fig. b)
Each Mamba layer is:
```
x → LayerNorm
  → Linear(d_model → 2×d_inner)  [in_proj, no bias]
  → split into (x_ssm, z)
x_ssm → SiLU → MambaSSMBlock → [SSM output]  ─┐
z     → SiLU                                   ├─ element-wise multiply
                                               ─┘
  → Linear(d_inner → d_model)  [out_proj]
  → + residual (x before LayerNorm)
```
`d_inner = d_model × expand = 64 × 2 = 128`

### Selective SSM Core (`MambaSSMBlock`)
The SSM computes a linear recurrence with **input-dependent** parameters (selectivity):

**1. Local mixing:**  depthwise `Conv1d(d_inner, k=3)` + SiLU — mixes neighbouring timesteps before the SSM.

**2. Input projections:**  a single `Linear(d_inner → 2×d_state + 1)` produces `(log_Δ, B, C)` jointly.

**3. Delta (discretisation step):**
```
Δ = softplus(dt_proj(log_Δ)).clamp(dt_min=0.001, dt_max=0.1)
```
`dt_proj` is initialised with bias = `inv_softplus(0.01)` so Δ starts at 0.01. The hard clamp prevents `dA = exp(Δ·A)` from underflowing to 0, which would cause numerical instability.

**4. Diagonal A:**  `A = −exp(A_log)` — always negative, shape `(d_inner, d_state)`, learnable log-magnitude.

**5. Discretise (ZOH):**
```
dA[t]  = exp(Δ[t] ⊗ A)               ∈ (0, 1]      shape: (B, L, d_inner, d_state)
dBu[t] = Δ[t] · B[t] · x[t]          (outer product on d dims)
```

**6. Sequential scan** (numerically stable for L=128):
```python
h = zeros(B, d_inner, d_state)
for t in range(L):
    h      = dA[:,t] * h + dBu[:,t]
    y[:,t] = (h * C[:,t].unsqueeze(1)).sum(-1)
```

**7. Output:**  `y + D × u` (skip connection, D is a learnable per-channel scalar).

> **Why sequential scan, not parallel?** The log-space parallel scan (`exp(cumsum(log(dA)))`) overflows when Δ is large at initialisation. For L=128, the sequential loop is fast enough and numerically stable.

### Bidirectional Mamba Block (`BidirectionalMambaBlock`, fig. a)
```
x ─────────────────────────── MambaLayer (forward)  →  fwd_out
x → flip(dim=L) → MambaLayer (backward) → flip back →  bwd_out
                                                             │
         cat([fwd_out, bwd_out], dim=-1)   shape: (B, L, 2×d_model)
                │
         Linear(2×d_model → d_model)
                │
         LayerNorm
```

### Loss & Training
- **Loss:** L1 reconstruction
- **Optimizer:** Adam, lr=5e-4
- **Scheduler:** `ReduceLROnPlateau`, patience=3, factor=0.5, floor 1e-5
- **Gradient clipping:** max norm 1.0
- **Early stopping:** patience=6 on val MSE
- **Checkpointing:** `denoising_checkpoints/best_dpnet_ppg.pt`
- **Epochs:** up to 30, **batch size 64** (smaller than other methods due to the `(B, L, d_inner, d_state)` intermediate tensors)
- **Logged extras:** `alpha` value per epoch (shows how much the model trusts its own output vs. the noisy residual)

---

## Method Comparison Summary

| # | Method | Type | Paired? | Key mechanism |
|---|---|---|---|---|
| 1 | EMD + LMS | Classical DSP | No | Decompose → identify noisy IMFs → adaptive cancel |
| 2 | CycleGAN | GAN | No (unpaired) | Two-domain cycle-consistent translation |
| 3 | FCGAN | GAN | Yes | Conditional pix2pix — predict & subtract noise residual |
| 4 | VQVAE | VAE + VQ | Yes | Discrete codebook bottleneck, ConvTranspose decoder |
| 5 | 1D U-Net | CNN | Yes | Encoder-decoder with lateral skip connections |
| 6 | DPNet | SSM (Mamba) | Yes | Bidirectional selective state space model + weighted residual |

All neural methods (2–6) use the same standardisation, the same 70/15/15 split, and are evaluated with MSE and MAE on the held-out test set.
