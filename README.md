# Time Series

This project is a study of time series analysis, wearable physiological signals, heart rate estimation, and PPG signal denoising. It starts with basic forecasting concepts, then moves into photoplethysmography applications, and finally builds a more detailed signal processing workflow for wrist based heart rate data.

## Introduction

The Introduction folder covers the main foundations needed for time series forecasting. It explains what time series data is, why trend, seasonality, cycles, variance, residuals, and stationarity matter, and how transformations such as differencing, logarithms, and Box Cox transforms help make data easier to model.

The notes then move through decomposition, autocorrelation, partial autocorrelation, forecasting metrics, baseline forecasting methods, exponential smoothing, Holt linear trend models, Holt Winter seasonal models, autoregression, moving average models, ARIMA, SARIMA, and harmonic regression. The folder also includes images for SARIMA and Fourier series based seasonality modeling.

![SARIMA model](Introduction/Sarima_Image.png)

![Fourier equation](Introduction/fourier-Eqn.png)

## Applications

The Applications folder focuses on photoplethysmography. It explains how PPG uses light absorption and reflection to estimate blood volume changes, heart rate, oxygen saturation, rhythm patterns, and possible blood pressure relationships. It also covers practical sensor concerns such as wavelength choice, wrist placement, skin tone, posture, pressure, signal quality, and quality metrics such as signal to noise ratio, perfusion index, and template matching.

![PPG applications](Applications/Images/ppgApplications.png)

![PPG signal data](Applications/Images/ppgData.png)

![PPG quality assessment](Applications/Images/qualityAssessment.png)

## Signal Processing

The SignalProcessing folder is the main implementation focused part of the project. It studies wearable PPG and ECG data from the PPG Field Study dataset, explains the dataset structure, prepares heart rate prediction windows, builds baseline and neural models, and documents several artifact removal methods for noisy wrist PPG signals.

The dataset notes explain how the PPG Field Study data is organized across 15 subjects. Each subject has synchronized wrist and chest recordings. The wrist device provides BVP, accelerometer, electrodermal activity, and temperature signals. The chest device provides ECG, respiration, accelerometer, and other physiological channels. ECG is treated as the ground truth source for heart rate labels, while wrist BVP and wrist accelerometer data are used as model inputs.

The data processing workflow explains the sampling rates and alignment. Wrist BVP is sampled at 64 Hz, wrist accelerometer at 32 Hz, activity labels at 4 Hz, ECG at 700 Hz, and heart rate labels at 0.5 Hz. The notes describe how each label corresponds to a window of signal data, how R peaks from ECG are used to derive heart rate, and why synchronization is needed between the chest sensor and the wrist device.

The heart rate modeling notebook predicts heart rate in beats per minute from wrist PPG and wrist accelerometer windows. ECG is not used as an input because it is the ground truth source. The prediction task uses 8 second windows with a 2 second shift. Each example contains 512 BVP samples and 256 accelerometer samples. For neural sequence models, BVP is downsampled to align with accelerometer data, producing inputs with 256 time steps and 4 channels consisting of BVP, ACC x, ACC y, and ACC z.

The modeling workflow begins with engineered features from BVP and accelerometer windows. BVP features include mean, standard deviation, minimum, maximum, peak to peak range, mean absolute value, RMS energy, and dominant frequency. Accelerometer features are computed for each axis and for acceleration magnitude. These features are used for Ridge Regression, Random Forest Regression, Gradient Boosting Regression, and a feature based MLP.

The project also tests raw sequence neural models for heart rate prediction. A 1D CNN learns local signal patterns from the aligned BVP and accelerometer channels. A bidirectional LSTM learns temporal dependencies across each 8 second window. A Transformer model uses self attention to focus on useful time regions inside the signal. Results are compared with MAE, RMSE, and R squared, with MAE in beats per minute treated as the main performance metric.

The artifact removal notes document the major noise sources that affect wrist PPG. These include motion artifacts, pressure artifacts, loose band coupling, ambient light interference, temperature drift, sweat and moisture effects, skin tone and tattoo related bias, venous pulsation, respiration effects, muscle and tendon movement, electromagnetic interference, and posture based hydrostatic changes. These artifacts can overlap in real use, which makes wrist PPG denoising a difficult signal processing problem.

![Noise examples](SignalProcessing/Images/Noise.png)

The artifact removal pipeline uses clean wrist BVP windows and adds synthetic artifacts to create noisy input signals. Each BVP window is 2 seconds long and contains 128 samples. Activity windows are used to modulate motion related artifact strength. The final denoising dataset contains noisy BVP input windows, clean BVP targets, and heart rate labels that are kept with the data but not used for the denoising objective. Neural methods use training set statistics for standardization and are evaluated using MSE and MAE against the clean target signal.

![Noise addition workflow](SignalProcessing/Images/NoiseAddition.png)

The first denoising method is a classical EMD and LMS adaptive filtering pipeline. Empirical Mode Decomposition separates a noisy signal into intrinsic mode functions. Noisy components are identified using zero crossing rate, and LMS adaptive filtering is then used to cancel estimated noise. The notes also include EEMD and CEEMDAN style variants using noise assisted decomposition.

The second denoising method is CycleGAN for unpaired signal translation. It uses one generator to map noisy signals to clean signals and another generator to map clean signals back to noisy signals. Cycle consistency, identity loss, and adversarial loss are used together so the model can learn signal domain translation even without strictly paired examples.

![CycleGAN architecture](SignalProcessing/Images/CycleGan.png)

The third method is FCGAN, a paired fully convolutional conditional GAN. Its generator predicts the noise residual and subtracts it from the noisy signal. Its discriminator receives noisy and clean or generated signal pairs, making it closer to a Pix2Pix style denoising setup for one dimensional PPG windows.

![FCGAN architecture](SignalProcessing/Images/FCGan.png)

The fourth method is a Vector Quantised Variational Autoencoder. The VQVAE encoder compresses the noisy signal into a latent sequence, snaps each latent vector to a learned codebook entry, and decodes the quantised representation into a denoised signal. This method studies whether a discrete bottleneck can preserve clean PPG structure while suppressing artifact patterns.

![VQVAE model view 1](SignalProcessing/Images/VQVAE-1.png)

![VQVAE model view 2](SignalProcessing/Images/VQVAE-2.png)

The fifth method is a 1D U Net. It uses an encoder decoder structure with skip connections so that local waveform details can pass around the bottleneck. This is useful for PPG because preserving peak shape and timing is important when recovering clean pulse morphology from noisy windows.

The sixth method is DPNet with bidirectional Mamba style selective state space blocks. This architecture processes the signal forward and backward through sequence modeling layers and blends the model output with the original noisy input using a learnable weighted residual. It is designed to model temporal structure in the PPG window while still retaining useful information from the measured signal.

The artifact removal comparison places the classical DSP method, CycleGAN, FCGAN, VQVAE, 1D U Net, and DPNet under the same denoising objective. The neural methods share the same train, validation, and test split, use the same standardization strategy, and are evaluated on held out data using MSE and MAE. Together, this part of the project turns the repository from basic time series notes into an applied wearable signal processing study.

## CGM — PPG-Based Glucose Estimation

The `SignalProcessing/cgm.ipynb` notebook builds a continuous glucose monitoring pipeline that predicts blood glucose values directly from raw PPG signals using the public Mazandaran CGM dataset version 2.

The dataset contains 67 PPG recordings from 23 participants sampled at 2175 Hz. Each recording is paired with one blood glucose reading measured at the time of the PPG capture.

The preprocessing pipeline applies a fourth-order Butterworth band-pass filter from 0.5 Hz to 8 Hz to remove baseline drift and high-frequency noise. The filtered signal is then downsampled from 2175 Hz to 30 Hz using polyphase resampling, producing a 300-sample window per recording that corresponds to 10 seconds of PPG at 30 Hz.

The dataset is split by participant ID using GroupShuffleSplit so that no subject appears in more than one of training, validation, and test. This produces 34 training recordings across 13 subjects, 16 validation recordings across 5 subjects, and 17 test recordings across 5 subjects. The subject-wise split gives a stricter estimate of how the model generalizes to unseen individuals.

Training data is augmented by adding Gaussian noise at three standard deviation levels (0.005, 0.01, 0.02), expanding the 34 training recordings to 136 augmented examples. Validation and test signals are kept untouched. Global mean and standard deviation normalization is fitted on the augmented training set only and then applied to all three splits to avoid leakage.

Three deep learning models are trained and compared for glucose regression.

Model 1 is a basic 1D CNN with two convolutional blocks followed by global average pooling and a fully connected regression head. It acts as the baseline for the comparison.

Model 2 is a Hybrid CNN-GRU. It runs three parallel branches on the same normalized PPG window. Branch one uses large convolutional kernels (sizes 15 and 7) to capture slow PPG envelope features. Branch two uses small convolutional kernels (sizes 5 and 3) to capture local pulse morphology details. Branch three is a two-layer GRU that models the sequential dynamics of the waveform. Each branch projects to a 64-dimensional embedding. The three embeddings are concatenated and passed through a dropout layer and a two-stage regression head.

Model 3 is a CNN with Bidirectional GRU and Self-Attention. Three sequential convolutional blocks with kernel sizes 7, 5, and 3 extract features and reduce the sequence length from 300 to 37 time steps through MaxPool layers. A Dropout layer regularizes the CNN output before it is fed into a two-layer bidirectional GRU that processes temporal context in both forward and backward directions, producing 128-dimensional hidden states per time step. An additive self-attention layer computes a softmax-weighted sum across time steps to produce a single 128-dimensional context vector. A fully connected head with Dropout then maps this vector to the glucose prediction.

All three models are trained with the AdamW optimizer using L1Loss (MAE) as the training criterion and early stopping based on validation MAE with a patience of 25 epochs.

Evaluation uses four metrics on the held-out test set: MAE, RMSE, MAPE, and R squared. The Clarke Error Grid is also applied to each model to assess clinical safety by categorizing predictions into zones A through E based on how much a prediction error would affect a clinical treatment decision.

A 10-fold cross-validation study is run across all three models using GroupKFold to ensure subject-level separation in every fold. Each fold trains from scratch with per-fold augmentation and per-fold normalization fitted only on the fold training data, then evaluates on the held-out fold test subjects. The cross-validation runs for a fixed 60 epochs per fold per model and reports MAE, RMSE, MAPE, and R squared for every fold. Results are summarized as mean and standard deviation across folds and visualized as side-by-side boxplots for all four metrics across the three models.
