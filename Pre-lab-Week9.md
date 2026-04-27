# Week 9 Pre-Lab: Change Detection & Validation — ARIA v6.0 Setup

**Course:** NTU Remote Sensing & Spatial Information Analysis (遙測與空間資訊之分析與應用)  
**Instructor:** Prof. Su Wen-Ray  
**Week:** 9 | **Theme:** Change Detection & Validation  
**Time Required:** ~15 minutes

---

## Objectives

By the end of this pre-lab, you will:
- Verify that your Week 8 environment and data are still accessible
- Confirm all required Python packages are installed
- Review confusion matrix fundamentals from memory
- Prepare item IDs for the Matai'an barrier lake case study

---

## Step 1: Verify Week 8 Environment

### 1a. Activate Your Virtual Environment

```bash
# Example for conda
conda activate remo_w8
# OR if using venv
source ~/remo_env/bin/activate
```

### 1b. Confirm Key Packages Are Available

Run the following in a Python interactive session or notebook:

```python
import pystac_client
import stackstac
import sklearn
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

print("✓ pystac_client:", pystac_client.__version__)
print("✓ stackstac version OK")
print("✓ sklearn:", sklearn.__version__)
print("✓ All core dependencies loaded successfully")
```

If any import fails, reinstall the missing package:

```bash
pip install pystac_client stackstac scikit-learn
```

---

## Step 2: Install scikit-learn (If Not Already Done)

scikit-learn is essential for confusion matrix computation and accuracy metrics.

```bash
pip install scikit-learn
```

**Verify installation:**

```python
from sklearn.metrics import confusion_matrix, precision_score, recall_score, f1_score
print("✓ scikit-learn metrics ready for Week 9")
```

---

## Step 3: Review Confusion Matrix Fundamentals

### Key Definitions (Commit to Memory)

| Term | Meaning |
|------|---------|
| **True Positive (TP)** | Model predicted "Change"; ground truth confirms "Change" |
| **False Positive (FP)** | Model predicted "Change"; but ground truth says "No Change" (False Alarm) |
| **True Negative (TN)** | Model predicted "No Change"; ground truth confirms "No Change" |
| **False Negative (FN)** | Model predicted "No Change"; but ground truth says "Change" (Missed) |

### Accuracy Metrics (Formulas to Know)

**Overall Accuracy (OA):** How many predictions were correct overall?  
$$OA = \frac{TP + TN}{TP + FP + TN + FN}$$

**Producer's Accuracy (PA):** Of the actual change pixels, how many did we detect? (Sensitivity, Recall)  
$$PA = \frac{TP}{TP + FN}$$

**User's Accuracy (UA):** Of the pixels we said changed, how many actually did? (Precision)  
$$UA = \frac{TP}{TP + FP}$$

**Kappa Coefficient:** Agreement beyond chance  
$$\kappa = \frac{OA - p_e}{1 - p_e}$$
where $p_e$ is the expected agreement by chance.

---

## Step 4: Self-Test — Hand Calculation

**Scenario:** You ran change detection on the Matai'an barrier lake. Your model vs. field validation gave:

```
Predicted      | Change | No Change
---+---+---+---+--------+----------
Actual Change  |   18   |    2
Actual No Chg  |    1   |   79
```

**Calculate (without a calculator, step by step):**

1. **TP =** _____  
   **FP =** _____  
   **TN =** _____  
   **FN =** _____

2. **Overall Accuracy (OA) =**  
   $(TP + TN) \div (TP + FP + TN + FN) = $ _____ $ \div $ _____ $ = $ _____%

3. **Producer's Accuracy (PA) =**  
   $TP \div (TP + FN) = $ _____ $ \div $ _____ $ = $ _____%  
   *(What fraction of actual changes did we correctly detect?)*

4. **User's Accuracy (UA) =**  
   $TP \div (TP + FP) = $ _____ $ \div $ _____ $ = $ _____%  
   *(What fraction of our predictions were correct?)*

**Answer Key** (check your work after completing):
- TP=18, FP=1, TN=79, FN=2
- OA = 97/100 = 97%
- PA = 18/20 = 90%
- UA = 18/19 ≈ 94.7%

**Interpretation:** The model is good at both detecting real changes (PA) and avoiding false alarms (UA). OA is high.

---

## Step 5: Prepare Your Week 8 Item IDs

You will need three Sentinel-2 item IDs from Week 8:

| Phase | Variable Name | Expected Value | Status |
|-------|---------------|----------------|--------|
| **Pre-event** | `PRE_ITEM_ID` | `S2A_MSIL2A_...` before barrier lake formed | ☐ Ready |
| **Mid-event** | `MID_ITEM_ID` | `S2A_MSIL2A_...` during early crisis | ☐ Ready |
| **Post-event** | `POST_ITEM_ID` | `S2A_MSIL2A_...` after remediation work | ☐ Ready |

**If you lost your IDs:** Retrieve them by re-running Week 8 Lab's search, or check your notebook outputs.

---

## Step 6: Prepare for Ground Truth Data

### What to Expect in Class

**Teacher will provide:** `validation_points.csv`
- 20+ manually verified field points
- Columns: `longitude`, `latitude`, `actual_change_binary` (0 or 1)
- Covers both changed and unchanged areas near the barrier lake

### Alternative (Optional Challenge)

If you want extra credit, you may collect your own 20+ ground truth points using:
- Google Earth Pro / ArcGIS base imagery
- Photos from public news sources (date-stamped)
- Local knowledge or field visit (if possible)

**Format your own CSV the same way as teacher's version** so your code is compatible.

---

## Step 7: Reflection Questions (Optional)

Answer these before class to deepen understanding:

1. **Why might PA ≠ UA?** When would you prefer high PA over high UA, and vice versa?
2. **Spectral mixing:** In a barrier lake disaster, what surfaces are "mixed" in a single Sentinel-2 pixel? Why is that a problem for change detection?
3. **Threshold trade-off:** If you lower your change detection threshold, what happens to TP, FP, PA, and UA?

---

## Checklist Before Class

- [ ] Verified pystac_client, stackstac, sklearn all work
- [ ] scikit-learn installed and confusion_matrix imported successfully
- [ ] Reviewed confusion matrix definitions (TP/FP/TN/FN)
- [ ] Completed self-test hand calculation
- [ ] Retrieved or recorded your three Week 8 item IDs (PRE, MID, POST)
- [ ] Understand that validation_points.csv will be provided in class
- [ ] Optional: reflected on PA vs. UA trade-off

**You're ready for Week 9!**

---

*Note: If you encounter any environment issues, post on NTUCool or email Prof. Su before class.*
