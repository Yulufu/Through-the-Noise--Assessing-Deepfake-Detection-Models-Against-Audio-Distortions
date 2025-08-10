# Through the Noise: Assessing Deepfake Detection Models Against Audio Distortions

![Status](https://img.shields.io/badge/status-work%20in%20progress-yellow)

> **Note:** This repository is actively being cleaned and refactored for public release. The code is being migrated from a private academic research environment to a fully documented and reproducible format.

## Abstract
Synthetic speech technology has advanced rapidly in recent years, making audio deepfakes increasingly convincing and accessible to create. This raises red flags for security and threatens the integrity of our digital communications. Though many studies have been done on improving detection models in clean audio environments, how detection models perform under real-world challenges is still uncertain. To address this gap, this thesis structured a series of audio augmentation to simulate common distortions in real-life such as additive noise, codec compression, time-stretching, pitch-shifting, and reverberation. To enhance the efficient testing process, a mini version of the ASVspoof 2019 dataset was generated. Then, three detection models were chosen as representatives for experiments: Wav2Vec2 (learning directly from raw audio), LCNN (learning from LFCC), and SafeEar (learning from privacy-preserving acoustic tokenization). The findings demonstrate that although Wav2Vec2 is the strongest under noise and compression, it does not work well under reverberation and extreme temporal changes. LCNN is the most stable among pitch and tempo changes but it is much more sensitive to noise. SafeEar deals with the extreme noise better than other models; however, it degenerates with downward pitch shifts and reverberation. In addition, an extended test was conducted to see if specific gaps in the acoustic frequency response might explain why all models’ performance declined under reverb. The research presents a detailed comparison of model robustness under realistic conditions and offers insights on designing more reliable and adaptable audio deepfake detection systems.

## Access to the Paper
You can find the pdf of the paper in this repo, named Fu_Yulu_Thesis_2025.pdf.

## Key Results

This thesis provides a detailed comparison of the robustness of three deepfake detection models—Wav2Vec2, LCNN, and SafeEar—against a suite of real-world audio distortions. The study reveals that no single model architecture is universally robust; instead, each exhibits unique strengths and vulnerabilities tied to its design.
The overall performance across all tested conditions is summarized below:

![alt text](figures/overall_roc_det_combined.png)

A detailed breakdown of each model's performance is as follows:

### 1. Wav2Vec2 (End-to-End, Raw Audio-Based)
Wav2Vec2 demonstrated the best overall performance (AUC ≈ 0.92) but was a "glass cannon," excelling in some areas while being surprisingly fragile in others.

- Strengths:
    - **Noise Robustness**: Exceptionally strong against all noise types, especially brown noise, where its performance remained nearly perfect even at 0 dB SNR. It consistently outperformed other models in low-to-moderate noise conditions (down to 15 dB SNR).
    - **Codec Compression**: Almost completely unaffected by any of the six tested audio codecs, maintaining near-perfect accuracy.
- Weaknesses:
    - **Temporal Distortions**: Highly sensitive to changes in speed and pitch. Performance dropped sharply when audio was slowed down below 0.5x speed or pitch was shifted by more than ±6 semitones.
    - **Reverberation**: The most vulnerable model to reverberation. Its accuracy collapsed in environments with long reverb times (e.g., large rooms), approaching random chance.

### 2. LCNN (Feature-Based, LFCC)
LCNN proved to be the specialist in handling spectral and temporal modifications, showcasing the power of its feature-based approach for these specific challenges.
- Strengths:
    - **Temporal Robustness**: By far the most robust model against time-stretching and pitch-shifting. It maintained excellent performance across all tested speeds (from 0.1x to 2.0x) and pitch shifts, where other models failed.
    - **Reverberation**: Showed better resilience to reverberation than Wav2Vec2, particularly in challenging large-room environments.
- Weaknesses:
    - **Noise Sensitivity**: The most sensitive model to additive noise. Its performance began to degrade significantly even at high SNRs (as early as 30 dB) for white and pink noise, which mask critical speech-frequency information in its LFCC features.
### 3. SafeEar (Privacy-Preserving, Acoustic Tokens)
SafeEar, the privacy-preserving model, offered a balanced but generally less accurate performance, highlighting a unique trade-off. It showed remarkable resilience in edge cases where others failed completely.
- Strengths:
    - **Extreme Noise Resilience**: Outperformed both Wav2Vec2 and LCNN in extremely noisy conditions (≤5 dB SNR), suggesting its token-based approach can still function when raw waveform and spectral features are heavily corrupted.
- Weaknesses:
    - **Downward Pitch Shifts**: Uniquely vulnerable to downward pitch shifts, where its performance degraded much more rapidly than with upward shifts.
    - **Reverberation**: Struggled with reverberation, showing significant performance drops in both small and large rooms.
    - **Brown Noise**: Its reliance on low-frequency prosodic cues made it the least effective model under brown noise, which masks those specific features.

### Summary Table of Model Robustness

| Distortion Type                  | Wav2Vec2  | LCNN      | SafeEar                        |
|:---------------------------------|:---------:|:---------:|:-------------------------------|
| **Overall Performance**          | Excellent | Good      | Fair                           |
| **Noise (White & Pink)**         | Excellent | Poor      | Fair (Good in extreme noise)   |
| **Noise (Brown)**                | Excellent | Good      | Poor                           |
| **Codec Compression**            | Excellent | Excellent | Good                           |
| **Time-Stretching**              | Poor      | Excellent | Good                           |
| **Pitch-Shifting**               | Fair      | Excellent | Fair (Poor on downward shifts) |
| **Reverberation**                | Poor      | Good      | Fair                           |


## Key Interpretations and Insights

Beyond the raw performance numbers, this research offers critical insights into why different models perform the way they do. The results highlight a fundamental trade-off in deepfake detection: the choice between processing raw audio waveforms versus pre-extracted acoustic features dictates a model's robustness profile.

### 1. The Raw vs. Feature-Based Divide: A Tale of Two Robustness Profiles
The opposing trends between LCNN and Wav2Vec2 clearly illustrate this trade-off:
- **Feature-Based Models (LCNN)**: LCNN's reliance on Linear Frequency Cepstral Coefficients (LFCCs) makes it exceptionally robust to temporal and pitch manipulations. Because LFCCs capture the general spectral shape of audio over short frames while being largely invariant to timing, the model can effectively "ignore" distortions like time-stretching and pitch-shifting. However, this same reliance is its greatest weakness: broadband noise (white and pink) directly corrupts the spectral information that LFCCs depend on, causing a rapid decline in performance even at high Signal-to-Noise Ratios (SNRs).
- **Raw Audio-Based Models (Wav2Vec2 & SafeEar)**: In contrast, models that learn from the raw waveform, like Wav2Vec2, develop a deep understanding of the precise temporal and phase structure of audio. This allows them to build highly noise-resistant representations, as they can learn to distinguish speech from unstructured noise. However, this makes them extremely sensitive to distortions that disrupt this delicate temporal structure. This explains their near-total failure under heavy reverberation and extreme time-stretching, which fundamentally smear these fine-grained details beyond what the model can generalize.

### 2. Reverberation: The Universal and Critical Challenge
A major takeaway from this research is that reverberation is a critical, and largely unsolved, challenge for all three types of models. While other distortions affected models differently, reverberation consistently caused significant performance degradation across the board, especially in environments with long decay times (e.g., large rooms).

The performance drop is likely due to two primary effects:
1. **Temporal Smearing**: Overlapping sound reflections mask crucial time-domain features like transients and precise phonetic timing, which is particularly damaging for models like Wav2Vec2 that rely on waveform structure.
2. **Spectral Distortion**: Reflections introduce comb filtering (sharp dips, or "nulls," in the frequency response) and spectral coloration, which corrupts the features used by models like LCNN and SafeEar.

The follow-up experiment on smoothing the Impulse Response (IR) spectra confirmed that mitigating these spectral nulls significantly improved LCNN's performance, reinforcing that feature-based models are highly susceptible to this type of distortion.

This insight strongly suggests that for deepfake detection systems to be reliable in the real world, future research must prioritize the development of reverberation-aware training techniques and model architectures.
## Key Figures
### 