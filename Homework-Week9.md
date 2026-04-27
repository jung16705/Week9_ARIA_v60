# Week 9 Homework: ARIA v6.0 — The Validated Auditor

**Course:** NTU Remote Sensing & Spatial Information Analysis (遙測與空間資訊之分析與應用)  
**Instructor:** Prof. Su Wen-Ray  
**Assignment:** Week 9 Homework  
**Due Date:** See NTUCool (typically 1 week after class)  
**Case Study:** Matai'an Barrier Lake, Taiwan

---

## Overview

This week you transition from exploratory change detection (Week 8) to **validated, quantitative assessment**. ARIA v6.0 (Automated Remote Imaging Auditor, v6.0) is your framework for building professional-grade disaster impact reports with full accuracy metrics and confidence bounds.

**Key Deliverable:** A Jupyter notebook + markdown report that combines:
- Multi-spectral change detection (ΔNDVI, ΔNDWI, ΔBSI)
- Threshold optimization by F1-score
- Confusion matrix & accuracy metrics
- Confidence zones for operational decision-making
- Comparison with Week 8's Eyewitness Impact Table

---

## Core Requirements (7 Tasks)

### Task 1: Quantitative Change Detection (20%)

**MANDATORY: Cloud Masking with SCL (Scene Classification Layer)**

Before any index computation, you **must** implement cloud masking to avoid "phantom water" artifacts:

1. **Implement `stream_scl()` function** to load and apply Scene Classification Layer masks:
   - **SCL_CLEAR_CLASSES** = [2, 4, 5, 6, 7, 11] (vegetation, bare soil, water, snow, shadows, dark areas)
   - Load SCL band from Sentinel-2 L2A product
   - Create per-scene binary masks: `valid_pre`, `valid_mid`, `valid_post` for individual visualizations
   - Create **intersection mask `valid`** = valid_pre ∩ valid_mid ∩ valid_post for difference map computation
   
2. **Motivation:** Without cloud masking, clouds and cloud shadows create false "water" signals in NDWI (the "phantom water" problem), leading to inflated inundation estimates. This comparison will be a deliverable (Task 4 update below).

3. **Apply masks to all indices:** After computing NDVI, NDWI, BSI, apply the intersection mask `valid` before downstream analysis.

---

Compute three normalized difference indices for **Pre→Mid** and **Pre→Post**:

1. **Normalized Difference Vegetation Index (NDVI)**
   $$\text{NDVI} = \frac{\text{NIR} - \text{Red}}{\text{NIR} + \text{Red}}$$
   - Pre, Mid, Post calculated separately
   - Store as `ndvi_pre`, `ndvi_mid`, `ndvi_post`

2. **Normalized Difference Water Index (NDWI)**
   $$\text{NDWI} = \frac{\text{Green} - \text{NIR}}{\text{Green} + \text{NIR}}$$
   - Detects inundation and water extent
   - Store as `ndwi_pre`, `ndwi_mid`, `ndwi_post`

3. **Bare Soil Index (BSI)**
   $$\text{BSI} = \frac{(\text{SWIR} + \text{Red}) - (\text{NIR} + \text{Blue})}{(\text{SWIR} + \text{Red}) + (\text{NIR} + \text{Blue})}$$
   - Detects exposed soil / debris
   - Store as `bsi_pre`, `bsi_mid`, `bsi_post`

**Compute differences:**
- $\Delta\text{NDVI} = \text{NDVI}_{\text{Mid or Post}} - \text{NDVI}_{\text{Pre}}$
- $\Delta\text{NDWI} = \text{NDWI}_{\text{Mid or Post}} - \text{NDWI}_{\text{Pre}}$
- $\Delta\text{BSI} = \text{BSI}_{\text{Mid or Post}} - \text{BSI}_{\text{Pre}}$

**Define dual bounding boxes for analysis:**
- **MATAIAN_BBOX** (regional): BBOX_WEST/SOUTH/EAST/NORTH for full study area context
- **LAKE_BBOX_LONLAT** (focused): [121.27, 23.68, 121.32, 23.72] for lake-specific accuracy assessment
  - Use MATAIAN_BBOX for visualization context
  - Use LAKE_BBOX_LONLAT when sampling validation points and computing local accuracy metrics

**Deliverable:**
- 2×2 subplot (ΔNDVI Pre→Mid, ΔNDVI Pre→Post, ΔNDWI, ΔBSI) with colorbars
- Table showing min/mean/max for each difference layer
- Both MATAIAN_BBOX and LAKE_BBOX_LONLAT defined in code and used appropriately

---

### Task 2: Threshold Optimization (20%)

Sweep at least **5 different thresholds** to find the optimal change detection boundary.

**Procedure:**

1. Define thresholds to test (e.g., -0.10, -0.20, -0.30, -0.40, -0.50 for ΔNDVI)
2. For each threshold:
   - Classify pixels: Change if $\Delta\text{NDVI} < \text{threshold}$, else No Change
   - Use teacher's `validation_points.geojson` to score predictions
     *(Note: these 60 points were corrected by the instructor using Google Earth Pro VHR imagery, NCDR reports, and Sentinel-2 visual interpretation.)*
   - Calculate: **F1-score**, **PA (Producer's Accuracy)**, **UA (User's Accuracy)**
3. Plot threshold vs. metric (F1, PA, UA on same figure)
4. **Select best threshold** based on highest F1-score (or balanced PA/UA per requirements)

**Deliverable:**
- Line plot: threshold (x-axis) vs. F1 / PA / UA (y-axis, three lines)
- Table: threshold, TP, FP, TN, FN, F1, PA, UA for each sweep
- Statement: "Best threshold is X with F1 = Y because..."

---

### Task 3: Confusion Matrix & Accuracy Assessment (20%)

Using the **best threshold from Task 2** and teacher-provided `validation_points.geojson`:

1. **Build 2×2 confusion matrix:**
   ```
   Predicted      | Change | No Change
   ---+---+---+---+--------+----------
   Actual Change  |   TP   |    FN
   Actual No Chg  |   FP   |    TN
   ```

2. **Calculate metrics:**
   - **Overall Accuracy (OA):** $(TP + TN) / (TP + FP + TN + FN)$
   - **Producer's Accuracy (PA):** $TP / (TP + FN)$ — of real changes, how many did we catch?
   - **User's Accuracy (UA):** $TP / (TP + FP)$ — of our predictions, how many were right?
   - **Kappa Coefficient:** $\kappa = \frac{OA - p_e}{1 - p_e}$ where $p_e$ is expected agreement by chance
   - **F1-Score:** $2 \times \frac{\text{PA} \times \text{UA}}{\text{PA} + \text{UA}}$

3. **Bonus (+5%):** Collect your own 20+ ground truth points using Google Earth Pro time-series imagery of the Matai'an area (compare pre/post typhoon VHR images). Record each point as `lon, lat, ground_truth_class, source` where source = "Google Earth Pro" or "news report". Re-run the accuracy assessment with YOUR points and compare results to the teacher's GeoJSON. This is the correct approach in real remote sensing projects — building your own independent validation dataset.

**Deliverable:**
- Confusion matrix display (sklearn's `confusion_matrix` + heatmap visualization)
- Metrics table: OA, PA, UA, Kappa, F1
- Interpretation: "PA=X% means we detected X% of actual changes. UA=Y% means Y% of our predictions were correct."
- *(If doing Bonus)* Your `my_validation_points.geojson` + comparison table showing metrics from teacher data vs. your own

---

### Task 4: Confidence Map & Three-Zone Classification (15%) + "Phantom Water" Error Case

Create a **confidence-based zoning** for operational decision-making:

**REQUIRED: "Phantom Water" Comparison Deliverable**

Before proceeding to confidence zones, create a side-by-side comparison showing the impact of cloud masking:
1. **Subplot A:** ΔNDWI computed **without** cloud mask (raw differences)
2. **Subplot B:** ΔNDWI computed **with** cloud mask applied (using SCL intersection mask from Task 1)
3. **Subplot C:** Difference visualization highlighting false positives (artifacts created by unmasked clouds)
4. **Caption:** "Without cloud masking, clouds and shadows create artificial 'water' signals (phantom water). Applying SCL masks removes these artifacts and produces accurate inundation mapping."

This deliverable demonstrates understanding of data quality issues and why validation is essential.

---

1. **Define confidence regions based on detection strength:**
   - **Zone 1 (High Confidence):** $|\Delta\text{NDVI}| > 1.5 \times \text{threshold}$ — strong change signal
   - **Zone 2 (Low Confidence):** Between threshold and Zone 1 boundary — borderline change
   - **Zone 3 (No Detection):** $|\Delta\text{NDVI}| \leq \text{threshold}$ — no significant change

2. **Compute area statistics:**
   - High confidence zone: ___ km²
   - Low confidence zone: ___ km²
   - No detection zone: ___ km²
   - Total study area: ___ km²

3. **Map visualization:**
   - Three-color map (e.g., red=high, yellow=low, blue=none) overlaid on a base map
   - Include legend, north arrow, scale

**Deliverable:**
- Color-coded map showing three zones
- Table with area (km²) for each zone
- Statement: "High confidence zones cover X km², representing the core impact area."

---

### Task 5: Validated Disaster Report (15%)

Write a professional assessment combining Tasks 1–4:

**Report structure (Markdown, ~500–800 words):**

1. **Executive Summary** (100 words)
   - Event: Matai'an barrier lake formation
   - Key finding: Area of change, confidence level
   - Recommendation: Safe / Caution / Danger zones

2. **Change Detection Analysis**
   - ΔNDVI / ΔNDWI / ΔBSI results from Task 1
   - Best threshold (Task 2) and justification
   - Confusion matrix & accuracy (Task 3)

3. **Confidence Assessment**
   - Three zones and areas (Task 4)
   - Which zones warrant immediate action?

4. **Ground Truth Validation**
   - Data source: teacher's CSV or your own collection
   - Sample points and locations
   - Any discrepancies with spectral data?

5. **Recommendations**
   - For evacuation planners: "High confidence zones have XX% certainty of impact"
   - For monitoring teams: "Return in [time] to revalidate low-confidence zones"
   - For disaster management: "Current accuracy enables [operational decision]"

**Deliverable:** Markdown section in your notebook titled "## 5. ARIA v6.0 Report"

---

### Task 6: AI Advisor — LLM Assessment (10%)

Feed your metrics to an LLM and request an operational assessment:

**Procedure:**

1. Prepare a summary of key metrics:
   ```
   - OA: X%
   - PA: Y%
   - UA: Z%
   - Kappa: K
   - High confidence area: A km²
   - Threshold: T
   ```

2. Prompt an LLM (ChatGPT, Claude, Gemini, etc.) with:
   > "Given these accuracy metrics from remote sensing validation of a barrier lake disaster, 
   > what confidence level would you assign to operational decisions? 
   > What additional data would improve confidence?"

3. **Document the exchange:**
   - Copy the exact prompt and LLM response into your notebook
   - Add your own interpretation: "The LLM highlights... which aligns/conflicts with..."

**Deliverable:** Markdown cell titled "## 6. AI Advisor Input" with prompt, response, and your reflection

---

### Task 7: Cross-Reference with Week 8's Eyewitness Impact Table (10%)

Retrieve your Week 8 Eyewitness Impact Table and compare:

**Comparison table:**

| Layer | W8 Finding | W9 Validated Finding | Agreement | Notes |
|-------|-----------|----------------------|-----------|-------|
| Vegetation Impact | "Damaged" / "Intact" / "?" | High/Low/None conf. change | Y/N | e.g., "W9 confirms W8's damage extent" |
| Water Inundation | … | … | Y/N | … |
| Debris Field | … | … | Y/N | … |

**Questions to answer:**
1. Did validation confirm or refute Week 8's visual interpretation?
2. Where did accuracy metrics show uncertainty? Did W8 overstate/understate those areas?
3. How did validation change your confidence in disaster extent?

**Deliverable:** Markdown section "## 7. Week 8 vs. Week 9 Comparison" with table and 3–4 sentences of analysis

---

## Professional Standards

### 1. Environment Reproducibility

**`.env` file (do NOT commit to GitHub):**
```
PRE_ITEM_ID=S2A_MSIL2A_...
MID_ITEM_ID=S2A_MSIL2A_...
POST_ITEM_ID=S2A_MSIL2A_...
BBOX_WEST=121.28
BBOX_SOUTH=23.56
BBOX_EAST=121.52
BBOX_NORTH=23.76
THRESHOLD_BEST=-0.15
```

**In your notebook:**
```python
import os
from dotenv import load_dotenv

load_dotenv()
PRE_ITEM_ID = os.getenv('PRE_ITEM_ID')
MID_ITEM_ID = os.getenv('MID_ITEM_ID')
POST_ITEM_ID = os.getenv('POST_ITEM_ID')
```

### 2. Captain's Log (Markdown Cells)

Between each major code section, insert a markdown cell describing:
- What you're doing and why
- Expected output
- Any insights or surprises

Example:
```markdown
## Captain's Log: Threshold Optimization

We're sweeping thresholds from -0.05 to -0.25 (ΔNDVI) 
to find the point where PA and UA balance best. 
F1-score is our primary metric. I expect F1 to peak 
around -0.15 based on Week 8's data spread.
```

### 3. Code Documentation

- Each function has a docstring with 1-line summary, parameters, returns
- Comments explain *why*, not just *what*
- Variable names are self-documenting (not `x`, `df1`, `result`)

---

## Grading Rubric (100%)

| Task | Component | Points | Criteria |
|------|-----------|--------|----------|
| **1. Change Detection** | Spectral indices (NDVI, NDWI, BSI) | 12% | All three computed correctly; clear difference maps |
| | Visualization | 8% | 2×2 or 3×2 subplots; labeled colorbars; statistics table |
| **2. Threshold Optimization** | Sweep design | 10% | ≥5 thresholds; covers plausible range; F1/PA/UA computed |
| | Plot & table | 10% | Clear threshold-vs-metric plot; tabulated results |
| **3. Confusion Matrix** | 2×2 matrix | 8% | TP/FP/TN/FN correct; formatted clearly |
| | Accuracy metrics | 7% | OA, PA, UA, Kappa, F1 all calculated & correct |
| | Interpretation | 5% | Explains what each metric means in context |
| **4. Confidence Map** | Zone classification | 8% | Three zones defined; areas computed |
| | Visualization | 7% | Color map; legend; professional appearance |
| **5. Disaster Report** | Content & structure | 9% | Executive summary, analysis, recommendations clear |
| | Writing quality | 6% | Concise, professional, no major grammar/spelling errors |
| **6. AI Advisor** | Prompt & response | 5% | Thoughtful prompt; documented response; your reflection |
| **7. W8 vs. W9 Comparison** | Accuracy | 4% | Table correctly compares findings |
| | Analysis | 6% | Explains agreement/disagreement; cites evidence |
| **Professional Standards** | .env reproducibility | 3% | Item IDs in .env; code reads from environment |
| | Captain's Log | 2% | ≥3 markdown cells explain reasoning |
| | Code quality | 2% | Well-commented; functions documented; clear variable names |

**Total: 100%**

---

## Bilingual Resources

### English Terms
- Change Detection = 變異檢測
- Producer's Accuracy = 製圖者精度 (or 測繪精度)
- User's Accuracy = 使用者精度
- Overall Accuracy = 總體精度
- Confidence Map = 信心度圖
- Threshold = 門值 / 閾值
- Ground Truth = 地面實況 / 地理事實

### Chinese Concepts
- **漏報率** (Omission Error) = 1 − PA (實際有變化但未檢出)
- **虛警率** (Commission Error) = 1 − UA (預測有變化但實際無)
- **Kappa 係數** = 考慮隨機一致性的精度
- **信度區** = 高、低、無檢測三個區域

---

## Important: Academic Responsibility & Output Verification

> **⚠️ 警告：你必須對作業內容負責。**

In previous weeks, some students submitted raw outputs without verifying whether results were correct or even reasonable. This is **dangerous** — in remote sensing, an unverified map can lead to wrong evacuation decisions, wasted resources, or missed hazards.

**Before you submit, you MUST:**

1. **Sanity-check every number.** If your OA = 99.9% or Kappa = 0.00, something is wrong. Ask yourself: "Does this make physical sense?"
2. **Inspect every figure.** Do your difference maps show change where you expect it (the lake area)? Or is the signal scattered randomly (likely a bug or unmasked clouds)?
3. **Understand what you wrote.** If someone asked you "Why did you choose this threshold?", could you answer? If not, you haven't finished the task.
4. **Don't copy-paste blindly.** Whether from the demo notebook, a classmate, or an AI tool — if you don't understand the code, you cannot verify the output.

**不懂可以提問（NTUCool、office hours、課堂上都可以），但不要敷衍交差。**  
**You are always welcome to ask questions, but do not submit work you haven't verified.**

Submitting unverified outputs will result in point deductions, even if the code runs without errors. A notebook that runs but produces wrong results is worse than one that crashes — because it looks correct and could mislead decision-makers.

---

## Submission Checklist

- [ ] **All outputs verified**: Every metric, figure, and table checked for reasonableness
- [ ] Jupyter notebook named `Week9_ARIA_v60_[Your_Name].ipynb`
- [ ] `.env` file (with dummy or real item IDs, NOT shared publicly)
- [ ] All 7 tasks completed with deliverables
- [ ] Professional presentation: clear markdown, figures, tables
- [ ] Captain's Log cells throughout notebook
- [ ] References to Week 8 comparison in final section
- [ ] Code is reproducible (runs without errors for TAs)
- [ ] Uploaded to NTUCool by due date (before 23:59)

---

## Additional Resources

- **scikit-learn metrics:** https://scikit-learn.org/stable/modules/model_evaluation.html
- **NDVI interpretation:** https://www.usgs.gov/faqs/what-ndvi-normalized-difference-vegetation-index
- **Confusion matrix tutorial:** https://en.wikipedia.org/wiki/Confusion_matrix
- **Prof. Su's Week 8 notebook:** Available on NTUCool

---

## Contact & Support

- **Questions?** Post on NTUCool Discussion or attend office hours
- **Data issues?** Check that your S2 items are from 2019–2020 (Matai'an event dates)
- **Environment errors?** Verify conda/pip packages in Week 9 Pre-Lab

**Good luck, and remember: validation is the bridge between cool maps and trusted science.**

---

*End of Homework Assignment — Week 9*
