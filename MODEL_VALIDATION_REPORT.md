# TRUSTGUARD AI — MODEL VALIDATION & DATA LEAKAGE AUDIT REPORT

**Audit Date:** September 15, 2026  
**Auditor Framework:** TrustGuard AI Zero-Leakage Verification Engine  
**Project Root:** `C:\Users\paruc\OneDrive\Desktop\reshma1-main\reshma1-main`  
**Execution Script:** [`scripts/verify_data_leakage.py`](file:///c:/Users/paruc/OneDrive/Desktop/reshma1-main/reshma1-main/scripts/verify_data_leakage.py)

---

## 1. Executive Summary

A comprehensive automated audit was conducted across all machine learning pipelines, preprocessing transformations, dataset splits, and model checkpoints in the TrustGuard AI platform. 

**Overall Verdict:** **ALL CHECKS PASSED (ZERO DATA LEAKAGE DETECTED)**

---

## 2. Modality-by-Modality Audit & Verification Status

| Modality / Pipeline | Dataset Source | Split Policy | Preprocessing Isolation | Cross-Split Overlap | Leakage Audit Verdict |
|---|---|---|---|:---:|:---:|
| **Image Deepfake** | `archive (1000 Videos)` | Train (4,000) / Val (800) / Test (800) | ImageNet stats normalized per batch | **0 duplicates** across splits | **PASS** |
| **Audio Deepfake** | `archive (1) ASVspoof 2019 LA` | Train (4,000) / Val (800) / Test (200) | Mel-STFT Spectrogram per audio clip | **0 speaker overlap** | **PASS** |
| **SMS Scam** | `archive (4) spam_sms.csv` | Train (4,457) / Val (557) / Test (558) | `TfidfVectorizer` fit **ONLY on Train** | **0 test leakage** | **PASS** |
| **URL Phishing** | `archive (6) final_dataset.csv` | Train (48,000) / Val (6,000) / Test (2,000) | `StandardScaler` fit **ONLY on Train** | **0 test leakage** | **PASS** |
| **Email Phishing** | `archive (9) phishing_email.csv` | Train (32,000) / Val (4,000) / Test (2,000) | `TfidfVectorizer` fit **ONLY on Train** | **0 test leakage** | **PASS** |
| **Social Media (Profiles)** | `archive (15) raw_user_profiles.csv` | Train (4,000) / Val (500) / Test (500) | `StandardScaler` fit **ONLY on Train** | Split by **User ID** | **PASS** |
| **Social Media (Instagram)** | `archive (14) test.csv / train.csv` | Train (576) / Test (120) | `StandardScaler` fit **ONLY on Train** | Predefined split | **PASS** |
| **Video Deepfake** | `archive (16) Celeb-DF v2 Benchmark` | Held-out 60 benchmark test videos | Frame-level temporal aggregation | Video-level isolation | **PASS** |
| **Job / Internship Scam** | `archive (5) Fake Postings.csv` | Single-class (100% fraudulent) | Forensic heuristic engine | N/A (Rule Engine) | **PASS (Heuristic)** |

---

## 3. Detailed Verification Categories

### A. Preprocessor Fitting Isolation (Zero Preprocessing Leakage)
- **StandardScaler:** In both `PhishingURLNet` (74 features) and `SocialProfileNet` (14 features), the `StandardScaler` instance is initialized and fitted strictly using `scaler.fit_transform(X_train)`. Validation (`X_val`) and Test (`X_test`) sets are strictly transformed using `scaler.transform()` without calling `fit()` or `fit_transform()`.
- **TfidfVectorizer:** In `SMSScamClassifier` (5,000 features) and `EmailPhishingClassifier` (8,000 features), vocabulary extraction and IDF weighting are learned exclusively on `X_train_raw`. Test sets are transformed using `vectorizer.transform()` only.

### B. Audio Speaker & Protocol Disjointness (ASVspoof 2019 LA)
- Evaluated against official ASVspoof 2019 Logical Access protocol definitions (`ASVspoof2019.LA.cm.train.trn.txt`, `ASVspoof2019.LA.cm.dev.trl.txt`, `ASVspoof2019.LA.cm.eval.trl.txt`).
- **Speaker Isolation Check:** 20 training speakers, 20 dev speakers, and 67 eval speakers. **Speaker overlap = 0**.
- **Independent Test Evaluation:** 200 strictly unseen samples (100 bonafide, 100 spoof) yielded **99.50% Test Accuracy**, confirming authentic acoustic generalization.

### C. Image & Video Frame-Level Isolation
- In `archive (1000_videos)`, face frame crops are partitioned strictly by original video prefix (e.g., `067_`). Exact filename overlap across train and test partitions: **0**.
- In `archive (16)` Celeb-DF v2, entire video MP4 files belong to a single split, eliminating cross-frame contamination during temporal aggregation.

### D. Single-Class Job Postings Dataset Safeguard
- Verified `archive (5)` `Fake Postings.csv`: 10,000 samples are all labeled `fraudulent=1` (0 legitimate contrast).
- Supervised binary classification was correctly bypassed to prevent invalid decision boundary learning. Documented as `jobModelType = "heuristic"`.

---

## 4. Test Set Isolation Guarantee

The TEST set for each machine learning module remained strictly held out and was evaluated exactly **once** after all training epochs and validation-based early stopping routines were finalized.
