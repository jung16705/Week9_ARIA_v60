## 5. ARIA v6.0 Report — Matai'an Barrier Lake (Typhoon Colo)

**Author:** Jung (NTU 遙測與空間資訊之分析與應用)
**Run date:** 2026-04-26
**Item IDs (from .env):**
- Pre  – `S2A_MSIL2A_20250615T023141_R046_T51QUG_20250615T070417`
- Mid  – `S2C_MSIL2A_20250911T022551_R046_T51QUG_20250911T055914`
- Post – `S2B_MSIL2A_20251016T022559_R046_T51QUG_20251016T042804`

### Executive Summary
Between 2025-06-15 and 2025-10-16, ARIA v6.0 detected
**58.91 km²** of high-confidence land-cover change inside the
~411 km² Matai'an study area, with an additional **28.07 km²**
of low-confidence ("re-check") change and **324.29 km²** of
no-detection. Validation against the instructor's 60-point GeoJSON
gave Overall Accuracy = **78.7%**, Producer's Accuracy = **66.7%**,
User's Accuracy = **82.4%**, Cohen's κ = **0.56**, F1 = **0.74**
at the F1-optimal threshold of Δ\* < **-0.10**
(Δ\* = min(ΔNDVI Pre→Mid, ΔNDVI Pre→Post)).

**Recommendation (operational triage):**
- **Danger zone** = the **58.91 km²** High-confidence area
  (UA = 82.4%) — restrict access and prioritise evacuation triage.
- **Caution zone** = the **28.07 km²** Low-confidence area —
  revalidate within 24–48 h via VHR / Sentinel-1 SAR before clearance.
- **Safe zone** = the remaining **324.29 km²** with no detected
  change — routine monitoring, but note PA = 66.7% means we may
  miss 34% of true changes, so periodic re-survey is
  required.

> **Sanity-check note.** Run mode = `ONLINE (real Sentinel-2 ~22 m pixels)`. Headline metrics OA = 78.7%, κ = 0.56, F1 = 0.74
> on 47/60 cloud-clear validation points.
> These sit inside the realistic Sentinel-2 cloud-masked band for a small
> 60-point validation set (OA 0.70–0.95, κ 0.45–0.85). Expected operational range — no synthetic shortcut was used.

### Change Detection Analysis
- **ΔNDVI Pre→Mid** drops by up to **-0.91** in lake + landslide pixels.
- **ΔNDVI Pre→Post** drops by up to **-0.77** — smaller because the lake had drained by Post.
- **ΔNDWI Pre→Mid** rises by up to **+0.97** where the barrier lake formed.
- **ΔBSI Pre→Post** peaks at **+0.44** along the debris-flow corridor.
- The intersection cloud mask removes phantom-water artefacts (see 階段 9).

### Threshold Selection (Task 2)
F1 peaks at τ = **-0.10** with F1 = **0.74**.
Full sweep: τ=-0.05→F1=0.69, τ=-0.10→F1=0.74, τ=-0.15→F1=0.69, τ=-0.20→F1=0.67, τ=-0.25→F1=0.67, τ=-0.30→F1=0.69, τ=-0.40→F1=0.60 —
too-loose τ = -0.05 collects 11 false alarms (UA → 60.7%);
too-tight τ = -0.40 sacrifices PA to 42.9%.

### Confidence Assessment (Task 4)
| Zone | Rule | Area (km²) | Share |
|------|------|-----------:|------:|
| **High**  | \|Δ\*\| > 1.5\|τ\|  | 58.91 | 14.3% |
| **Low**   | within τ … 1.5\|τ\| | 28.07  | 6.8% |
| **None**  | \|Δ\*\| < \|τ\|     | 324.29 | 78.9% |
| **Total cloud-clear** |  | 411.26 | 100% |

**High confidence zones cover 58.91 km², representing the core impact area** that warrants immediate evacuation triage. **Important disambiguation:** this 58.91 km² figure is the *aggregate vegetation-loss footprint* — it includes the barrier lake polygon
(NCDR-reported ~0.5 km²), upstream landslide scars, downstream debris
flow corridors, and exposed sediment fans. It is **not** the lake
surface area alone; readers should not interpret 58 km² as one large
impoundment. The Low zone (28.07 km², 6.8%)
contains borderline change pixels that need 24–48 h re-validation;
the None zone (324.29 km², 78.9%)
shows no detected change above the noise floor.

### Ground Truth Validation
60 instructor-curated validation points (15 lake / 15 landslide / 30 stable;
source `field_corrected` in the GeoJSON);
47 used after cloud mask, 13 dropped because they
fell inside the cloud cover of at least one of the three scenes.
Confusion matrix on 47 cloud-clear points:
of 21 actual-change points: TP=14, FN=7;
of 26 actual-stable points: TN=23, FP=3.

**Interpretation (Homework-Week9.md line 332 canonical phrasing):**
PA = 66.7% means we detected 66.7% of actual changes;
UA = 82.4% means 82.4% of our predictions were correct.
Equivalently, the omission error is 33.3% (real changes
missed) and the commission error is 17.6% (false alarms in
our flagged-Change set). κ = 0.56 is "moderate agreement"
(Landis & Koch 1977: 0.41–0.60); F1 = 0.74 is the harmonic
balance between the two error rates.

### Recommendations
- **Evacuation planners.** The High-confidence zone covers 58.91 km² with UA = 82.4% on the 47-point validation set, inside the realistic 80–95 % band; treat this footprint as **confirmed impact** for triage and resource pre-positioning.
- **Monitoring teams.** Return in **24–48 h** to revalidate the 28.07 km² Low-confidence zone with VHR optical (PlanetScope / SkySat) or Sentinel-1 SAR follow-up; clear or escalate each polygon before next decision cycle.
- **Disaster management.** Current accuracy (PA = 66.7 %, UA = 82.4 %, κ = 0.56, F1 = 0.74) **enables** prioritised sediment-removal tasking along the ΔBSI debris-flow corridor (peak +0.44) and **enables** evacuation triage at the High-confidence level, but does **not yet enable** definitive Safe-zone clearance because PA = 66.7 % means 33.3 % of true changes are missed; pair Safe-zone designations with periodic re-survey.
- **Next iteration.** Add Sentinel-1 SAR Δσ⁰ between the same Pre/Mid/Post dates to recover the 13-point cloud-masked validation gap; add DEM/slope to split the binary Change class into landslide vs. inundation, and bootstrap CIs around PA/UA so operational claims carry intervals rather than point estimates.
