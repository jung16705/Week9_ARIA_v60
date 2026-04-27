### LLM Response (Claude 4.7 Opus, 2026-04-26)

> **Confidence assessment.** OA = 78.7%, PA = 66.7%, UA = 82.4%,
> κ = 0.56, F1 = 0.74 on 47/60 cloud-clear GT points puts
> the detector in the *operational-with-caveats* band:
> 1. **Numbers are realistic.** OA 0.80–0.92 / κ 0.60–0.85 is exactly what
>    cloud-masked Sentinel-2 ΔNDVI change detection produces.
> 2. **Sample size matters.** With n = 47 (n_change = 21,
>    n_stable = 26) the 95% Wilson CI on PA is
>    [0.45, 0.83]; on UA it is [0.59, 0.94].
>    Quote those, not point estimates.
> 3. **13 points were dropped by the cloud mask.** Treat
>    those as *unknown-risk*, not zero-risk.
>
> **Operational guidance.**
> * High zone (58.9 km²) → confirmed-impact for evacuation triage.
> * Low zone  (28.1 km²) → re-validate ≤ 48 h via VHR / SAR.
> * Do NOT use this output to *exclude* areas — PA = 66.7% means
>   34% of true changes were missed.
>
> **Highest-ROI additions.** (1) Sentinel-1 SAR Δσ⁰ to close the
> 13-point cloud gap; (2) DEM/slope to split landslide vs.
> inundation; (3) bootstrap CIs; (4) larger validation set (≥200 pts).

### 反思 / Reflection

LLM 把點估計直接翻成 Wilson interval（PA: [0.45, 0.83],
UA: [0.59, 0.94]），這是我在 §13 應該主動做的。它把
"do not use to exclude areas" 寫得太絕對 — 操作上一定要劃出排除區
才能分配有限資源；正確的折衷是「排除 + 排程再驗」而不是永久 safe label。
最高 ROI 建議 (Sentinel-1 SAR fusion) 與 §H7 路線圖一致。
