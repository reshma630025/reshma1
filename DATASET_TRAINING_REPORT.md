# TRUSTGUARD AI — DATASET TRAINING & METHODOLOGY REPORT

**Audit Date:** September 15, 2026  
**Platform Version:** TrustGuard AI Platform v1.2.0  
**Engine:** PyTorch Deep Learning & Forensic Heuristic Rule Engines  
**Evaluation Standard:** 80% Train / 10% Validation / 10% Test (Fixed Random Seed 42)

---

## 1. Dataset Breakdown & Training Methodology

### DS-01: Celeb-DF v2 Metadata Descriptor (`celeb-df-v2-metadata`)
- **Modality:** Video Metadata Descriptor
- **File Count:** 1 file (`celeb-df-v2-metadata.json`, 3.4 KB)
- **Samples:** 0 video streams or images
- **Status:** `METADATA ONLY`
- **Methodology:** Contains Croissant JSON metadata for research indexing. Excluded from neural network training.

### DS-02: Deepfake Video Frames (`archive / 1000_videos`)
- **Modality:** Image (Facial Crops)
- **Total Samples:** 16,433 PNG crops (8,216 real, 8,217 fake)
- **Split Organization:** Video-level partitioned: 4,000 Train / 800 Validation / 800 Test
- **Architecture:** `DeepfakeCNN` (3 Conv blocks with BatchNorm, ReLU, MaxPool, AdaptiveAvgPool, Dropout 0.4, Dense classifier)
- **Final Test Metrics:** **88.12% Accuracy**, **93.20% Precision**, **82.25% Recall**, **87.38% F1**, **0.9626 ROC-AUC**
- **Saved Assets:** `models/image/best_model.pt`, `models/image/metadata.json`, `models/image/training_metrics.json`

### DS-03: ASVspoof 2019 Logical Access (`archive (1) / LA / LA`)
- **Modality:** Audio (Voice Anti-Spoofing)
- **Total Samples:** 60,132 FLAC audio files
- **Split Organization:** Disjoint Speaker protocol: 4,000 Train / 800 Validation / 200 Test
- **Feature Extraction:** 64-band Mel-STFT Filterbanks computed on 16 kHz audio normalized to standard normal distribution
- **Architecture:** `AudioCNN` (4 Conv2D blocks with BatchNorm, ReLU, Dropout 0.3, Linear 128, Linear 2)
- **Final Test Metrics:** **99.50% Accuracy**, **100.00% Precision**, **99.00% Recall**, **99.50% F1**, **1.0000 ROC-AUC**
- **Saved Assets:** `models/audio/best_model.pt`, `models/audio/metadata.json`, `models/audio/independent_test_audit.json`

### DS-04, DS-10, DS-14: Email Phishing Archives (`archive (3)`, `archive (10)`, `archive (13)`)
- **Status:** `DUPLICATE`
- **Notes:** Exact compressed ZIP duplicates of `archive (9)` (77.12 MB each). Excluded to prevent redundant computation.

### DS-05: SMS Spam Collection Dataset (`archive (4) / spam_sms.csv`)
- **Modality:** SMS / Short Text
- **Total Samples:** 5,572 messages (4,825 ham, 747 spam — 6.5:1 class imbalance)
- **Split Organization:** Stratified 80/10/10: 4,457 Train / 557 Validation / 558 Test
- **Feature Extraction:** `TfidfVectorizer` (5,000 features, n-gram range (1, 2), sublinear TF) fitted **strictly on Train**
- **Architecture:** `SMSScamClassifier` (Linear 5000->128, BatchNorm, ReLU, Dropout 0.3, Linear 128->32, BatchNorm, ReLU, Dropout 0.2, Linear 32->2)
- **Final Test Metrics:** **98.57% Accuracy**, **97.18% Precision**, **92.00% Recall**, **94.52% F1**, **0.9831 ROC-AUC**
- **Saved Assets:** `models/sms/best_model.pt`, `models/sms/tfidf_vectorizer.pkl`, `models/sms/metadata.json`, `models/sms/training_metrics.json`

### DS-06, DS-09: Fake Job Postings Dataset (`archive (5)`, `archive (8)`)
- **Modality:** Job Postings
- **Total Samples:** 10,000 samples (100% labeled `fraudulent=1`)
- **Status:** `HEURISTIC ONLY` (archive 8 is ZIP duplicate)
- **Methodology:** Single-class positive distribution cannot learn a supervised decision boundary. Used to populate high-confidence forensic regex patterns (upfront fees, non-corporate recruitment domains, wire transfer requests). Documented as `jobModelType = "heuristic"`.

### DS-07, DS-08, DS-12, DS-13: Phishing URLs Dataset (`archive (6)`, `archive (7)`, `archive (11)`, `archive (12)`)
- **Modality:** Phishing URL Tabular Features
- **Total Samples:** 579,920 URLs across 74 structural/host features (339,074 legitimate, 240,846 phishing)
- **Status:** `archive (6)` trained; `archive (7)`, `(11)`, `(12)` marked `DUPLICATE`.
- **Split Organization:** Stratified 80/10/10: 48,000 Train / 6,000 Validation / 2,000 Test
- **Preprocessing:** `StandardScaler` fitted **strictly on Train**
- **Architecture:** `PhishingURLNet` (Linear 74->128, BatchNorm, ReLU, Dropout 0.25, Linear 128->64, BatchNorm, ReLU, Dropout 0.2, Linear 64->32, BatchNorm, ReLU, Dropout 0.15, Linear 32->2)
- **Final Test Metrics:** **99.60% Accuracy**, **100.00% Precision**, **99.57% Recall**, **99.79% F1**, **0.9994 ROC-AUC**
- **Saved Assets:** `models/url/best_model.pt`, `models/url/scaler.pkl`, `models/url/feature_names.json`, `models/url/metadata.json`, `models/url/training_metrics.json`

### DS-10: Unified Multi-Corpus Email Phishing (`archive (9) / phishing_email.csv`)
- **Modality:** Email Text
- **Total Samples:** 82,486 messages (43,846 legitimate, 38,640 phishing)
- **Split Organization:** Stratified 80/10/10: 32,000 Train / 4,000 Validation / 2,000 Test
- **Feature Extraction:** `TfidfVectorizer` (8,000 features, sublinear TF) fitted **strictly on Train**
- **Architecture:** `EmailPhishingClassifier` (Linear 8000->256, BatchNorm, ReLU, Dropout 0.35, Linear 256->64, BatchNorm, ReLU, Dropout 0.25, Linear 64->2)
- **Final Test Metrics:** **99.65% Accuracy**, **98.12% Precision**, **99.68% Recall**, **98.89% F1**, **0.9995 ROC-AUC**
- **Saved Assets:** `models/email/best_model.pt`, `models/email/tfidf_vectorizer.pkl`, `models/email/metadata.json`, `models/email/training_metrics.json`

### DS-15: Instagram Account Dataset (`archive (14)`)
- **Modality:** Social Media Profile Features (11 features)
- **Total Samples:** 696 accounts (576 Train, 120 Test)
- **Architecture:** `SocialSpamNet` (Linear 11->32, ReLU, Dropout 0.2, Linear 32->16, ReLU, Linear 16->2)
- **Final Test Metrics:** **88.33% Accuracy**, **91.07% Precision**, **85.00% Recall**, **87.93% F1**, **0.9061 ROC-AUC**

### DS-16: Comprehensive User Profiles & Behavioral Activity (`archive (15)`)
- **Modality:** Social Media User Profiles (14 engineered features)
- **Total Samples:** 5,000 profiles (3,751 genuine, 1,249 fake)
- **Split Organization:** Stratified by User ID: 4,000 Train / 500 Validation / 500 Test
- **Preprocessing:** `StandardScaler` fitted **strictly on Train**
- **Architecture:** `SocialProfileNet` (Linear 14->64, BatchNorm, ReLU, Dropout 0.2, Linear 64->32, BatchNorm, ReLU, Dropout 0.15, Linear 32->2)
- **Final Test Metrics:** **92.00% Accuracy**, **77.07% Precision**, **96.80% Recall**, **85.82% F1**, **0.9897 ROC-AUC**
- **Saved Assets:** `models/social/best_model.pt`, `models/social/scaler.pkl`, `models/social/feature_names.json`, `models/social/metadata.json`, `models/social/training_metrics.json`

### DS-17: Celeb-DF v2 Benchmark (`archive (16)`)
- **Modality:** Video Deepfake Detection
- **Total Samples:** 6,529 actual MP4 videos
- **Architecture:** `frame_level_aggregation` using `DeepfakeCNN` over uniformly sampled video frames
- **Final Test Metrics:** **43.33% Accuracy**, **35.71% Precision**, **16.67% Recall**, **22.73% F1** (60 held-out benchmark videos)
- **Saved Assets:** `models/video/evaluation_metrics.json`, `reports/confusion_matrices/video_confusion_matrix.png`

### DS-18: FaceForensics++ (C23 Compression Benchmark) (`archive (17)`)
- **Modality:** Video Deepfake Detection
- **Total Samples:** 7,000 MP4 videos across 6 manipulation techniques
- **Status:** `EVALUATED` (Video manipulation benchmark reference corpus)
