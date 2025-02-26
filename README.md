# Jupyter_Notebook_GPS_Spoofing_and_Jamming_Detection_in_Drone_Flight_Using_Machine_Learning-2

# UAV Security Project: GPS Jamming & Spoofing Detection

## 1. Introduction
### 1.1 Research Background
- **Drone Security Concern**  
  Modern UAVs rely heavily on GPS signals for navigation. Attackers can exploit this dependency through **GPS Jamming** or **GPS Spoofing**, causing critical system malfunction or hijacking.

- **Motivation**  
  Traditional approaches detect known attacks effectively but may fail when **unseen or novel attack patterns** appear.

- **Objective**  
  1. Build a **machine learning-based attack detection model** using **UAVAttack** (Benign, Jamming, Spoofing).
  2. Evaluate **generalization performance** on a **newly generated synthetic dataset** simulating novel attack scenarios.

### 1.2 Scope of the Project
- **Datasets**  
  - **UAVAttack**: Training & validation  
  - **synthetic_new_data_aligned.csv**: Unseen test set for generalization test
- **Attack Types**:  
  - **GPS Jamming** → Transmitting strong noise to disrupt GPS  
  - **GPS Spoofing** → Injecting fake GPS signals to mislead UAV coordinates  
  - **Benign Flight** → Normal UAV flight logs without malicious interference

---

## 2. Dataset & Preprocessing
### 2.1 UAVAttack Dataset
- **Features**:  
  `lat, lon, alt, eph, epv, jamming_indicator, noise_per_ms, satellites_used, ...`
- **Class Distribution**: Approximately 60% benign, 40% attacks
- **Issues**:  
  - **Class Imbalance** (fewer attack samples)  
  - **Missing Values** in critical features (e.g., noise_per_ms, satellites_used)

### 2.2 Synthetic Dataset
- **Reason for Creation**:  
  Real-world attacks may differ from training patterns. We need to see if the model can handle **unseen data**.
- **Feature Set**: Same schema as UAVAttack but with different distributions.
- **PCA Analysis**:  
  The synthetic dataset shows a distribution shift compared to UAVAttack, indicating potential **generalization challenges**.

### 2.3 Preprocessing Steps
1. **Missing Value Handling**  
   - Drop columns with >90% missing.  
   - Impute remaining NaN values with median (SimpleImputer).
2. **Feature Scaling**  
   - Applied StandardScaler to numeric features (`accelerometer`, `gyro`, etc.).
3. **Train/Test Split**  
   - UAVAttack → 80% training, 20% validation.  
   - Synthetic dataset → final unseen test set.

---

## 3. Model Architecture & Methods
### 3.1 Traditional ML Models
- **Random Forest**: Ensemble of decision trees; robust but can be skewed by imbalance.  
- **XGBoost**: Gradient boosting framework; high performance but prone to overfitting if not tuned.  
- **Logistic Regression**: Baseline linear model; interpretable but less capable for complex patterns.

### 3.2 Class Imbalance Handling: SMOTE
- **SMOTE** (Synthetic Minority Over-sampling Technique)  
  Oversamples minority classes (GPS Jamming/Spoofing), enhancing recall for underrepresented attacks.

### 3.3 Stacking Ensemble
- **Base Models**: Random Forest, XGBoost, Logistic Regression  
- **Meta Model**: Logistic Regression (class_weight='balanced')  
- **Rationale**:  
  Combines strengths of multiple classifiers to achieve **better generalization**.  
  Final meta-classifier inputs base predictions, outputting improved performance.

---

## 4. Experimental Results
### 4.1 UAVAttack (Original Dataset)
| Model                 | Accuracy | Recall |
|-----------------------|----------|--------|
| Random Forest         | 0.60     | 0.64   |
| XGBoost              | 0.58     | 0.62   |
| Logistic Regression   | 0.45     | 0.42   |
| **Stacking Ensemble** | **0.65** | **0.68** |

- **SMOTE** improved minority-class detection slightly (Recall +4~6%).  
- **Stacking Ensemble** best overall, yet some attacks remain undetected (False Negatives).

### 4.2 Synthetic Dataset (Unseen)
| Model                 | Accuracy | Recall |
|-----------------------|----------|--------|
| Random Forest         | 0.52     | 0.50   |
| XGBoost              | 0.50     | 0.48   |
| Logistic Regression   | 0.38     | 0.34   |
| **Stacking Ensemble** | **0.56** | **0.54** |

- **Performance drop** of 5~10% for all models.  
- **False Negatives** increase, especially for Jamming.  
- Stacking still leads but shows a **gap vs. new patterns**.

---

## 5. Analysis: GPS Jamming & Spoofing
### 5.1 Probability Distribution Observations
- **Class 0 (Benign)**: Many samples predicted at lower probabilities (0.0~0.3) → potential **False Positives**.
- **Class 1 (Jamming)**: Shift from (0.8~1.0) to (0.0~0.5) → possible **False Negatives**.  
- **Class 2 (Spoofing)**: Broad 0~1 spread, some improvement but uncertain outliers remain → partial success.

### 5.2 Underlying Causes
- **Distribution Shift**: Synthetic data differs from UAVAttack → model fails to generalize.  
- **Complex Attack Patterns**: Partial jamming or stealthy spoofing can appear normal.  
- **Overfitting**: Model specialized to UAVAttack logs, not new signals.

---

## 6. Conclusion & Future Work
### 6.1 Conclusions
- **New Dataset Issue**: All models degrade in performance when faced with unseen attack patterns.
- **SMOTE & Stacking**: Helpful for class imbalance and baseline performance, but cannot fully solve the distribution gap.
- **Remaining Challenge**: GPS Jamming & Spoofing can still result in **False Negative** or **False Positive** predictions.

### 6.2 Future Plans
- **Deep Learning Model Testing**  
  - **LSTM/GRU** for sequential sensor data.  
  - **CNN** for spatial/temporal feature extraction.  
- **Domain Adaptation**  
  - Align feature distributions (original vs. new) to reduce generalization gap.
- **Further Data Augmentation**  
  - Introduce varied noise or advanced attack patterns to emulate real-world complexity.

---

**With this README, we demonstrate how a machine learning approach can detect GPS Jamming and Spoofing, and also reveal the limitation when encountering novel or unseen data.**
