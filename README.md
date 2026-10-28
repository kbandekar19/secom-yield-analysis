# Semiconductor Yield Analysis

Statistical process control and root cause investigation of end-of-line failures in a
semiconductor manufacturing process.

**Data:** SECOM dataset (UCI Machine Learning Repository) — 1,567 production runs,
590 monitored process parameters, 89 days of production (19 Jul – 17 Oct 2008).

---

## Summary

Baseline first-pass yield was **93.36%** (104 failures in 1,567 runs). Rather than a single
persistent defect source, the data shows a **process that changed partway through**: the
failure rate dropped from 12.92% to 4.11% around week 35 and stayed down.

Analysis of individual parameters initially produced 22 sensors significantly associated with
failure. Phase-stratified testing showed these associations were **confounded with the process
change** rather than causal. The actionable findings are at the process level: an undocumented
improvement that is not under control, a dated single-week excursion, and 116 parameters that
have not moved in three months.

---

## Findings

### 1. Yield improved 3x partway through, and nobody documented why

| Period | Units | Failures | Failure rate |
|---|---|---|---|
| Weeks 29–34 | 449 | 58 | 12.92% |
| Weeks 35–42 | 1,118 | 46 | 4.11% |

Difference confirmed at p < 10⁻⁶ (chi-square). Within each period the process is stable against
its own control limits — the early period was not suffering excursions, it was **consistently
bad**, which is a common-cause problem.

212 of 446 analysed parameters shifted significantly at the same point. A change that broad
suggests something systemic rather than a single setpoint tweak. Weekly volume also rose from 13
to roughly 200 units over the same period, which is consistent with a **production ramp**.
Sensor 28 shows a gradual rather than stepped transition, supporting that reading.

Alternative explanations the data cannot rule out: a recipe or equipment change, or a change to
the end-of-line test itself. The last of these matters most — if the test changed, measured yield
improved while the product did not.

### 2. A week-40 excursion that the combined control chart hides

A p-chart built across the whole period puts the centre line at 6.64%, blending two different
processes. Against those limits, week 40 looks unremarkable.

Recomputing limits **within the stable period** (centre line 4.11%) puts week 40 at 8.88%
against a UCL of 8.70% — **above the upper control limit**, a genuine special-cause excursion.

Sensors 78 and 79 both shifted significantly during that week (p < 0.0001), giving the
investigation a dated event with named parameters.

### 3. The apparent failure drivers are confounded with the process change

| Sensor | FDR-adjusted p (all data) | Cohen's d | Within Phase 1 alone |
|---|---|---|---|
| 59 | 0.00006 | 0.63 | p = 0.012 |
| 103 | 0.00009 | 0.61 | p = 0.259 |
| 129 | 0.00003 | 0.41 | p = 0.889 |

All three differ enormously between phases regardless of pass/fail (p = 10⁻⁴³ to 10⁻⁶⁹). Because
failures were concentrated in the early period, parameters that simply changed over time appear
associated with failure. Within Phase 1 alone, sensors 103 and 129 show no relationship to
failure at all.

Inside the stable period, 17 parameters clear a Welch t-test after FDR correction, but **none
survive Mann-Whitney**. Inspection shows the t-test differences are driven by outliers: for
sensor 155, failed and passed units have near-identical medians (0.270 vs 0.280) while the pass
mean is inflated by values up to 12.61.

**Conclusion: no individual parameter is robustly associated with failure in this dataset.**

### 4. 116 parameters have not moved in 89 days

Of 590 parameters, 116 report an identical value across every run. Some are likely fixed
setpoints by design; others may be stuck or failed sensors. Either way, a parameter that cannot
vary cannot raise an alarm, and nothing flagged this in three months.

---

## Root cause analysis

### 5 Whys — week-40 excursion

1. Why did failures spike to 8.9%? → Process conditions changed; sensors 78 and 79 both shifted.
2. Why did those parameters shift? → Unknown — no equipment, maintenance or material records
   are linked to this data.
3. Why is that unknown? → Process data and production event records are not connected.
4. Why not? → No traceability requirement linking sensor data to maintenance, lots and changes.
5. **Root cause:** without linked records, every excursion is investigated from scratch and
   recurring causes cannot be recognised.

### 5 Whys — week-35 improvement

1. Why did yield improve 3x? → Operating conditions changed; 212 of 446 parameters shifted.
2. Why did they shift? → Undocumented; possibly a ramp, a recipe change, or a test change.
3. Why undocumented? → No change-control record ties process changes to yield outcomes.
4. Why does that matter? → The improvement cannot be deliberately maintained or repeated.
5. **Root cause:** the improved condition is accidental rather than specified, so nothing
   prevents it drifting back.

### Fishbone — week-40 excursion

| Category | Candidate causes | Status |
|---|---|---|
| Machine | Equipment drift or fault; sensors 78/79 shifted that week; maintenance event | Shift confirmed, cause not |
| Material | New lot or supplier change | Untestable — no lot data |
| Method | Recipe or setpoint change; altered rework procedure | Untestable — no change records |
| Man | Operator or crew change; training gap | **Ruled out** — no shift effect (p = 0.88) |
| Measurement | Test equipment drift or recalibration | Untestable — no gauge data |
| Environment | Temperature or humidity excursion | Untestable — not instrumented |

---

## Recommendations

**Containment (days)**

- p-chart monitoring on weekly yield with limits from the stable period, so an excursion like
  week 40 is flagged within a week rather than at quarterly review.
- Audit the 116 constant-value parameters with maintenance; restore or formally retire each.

**Corrective (weeks)**

- Identify and document what changed around week 35 and write it into the control plan.
- Investigate week 40, starting from sensors 78 and 79.
- I-MR charts on the highest-impact shifted parameters (28, 103, 79, 78, 59) to catch drift back
  toward the earlier operating point.

**Preventive (ongoing)**

- Change control: log any revision to the end-of-line test alongside yield data, so a measurement
  change cannot be mistaken for a process improvement.
- Re-run capability analysis once the process is confirmed stable and spec limits are available.

---

## Method

1. Define and baseline — yield, defect rate, data quality
2. Stratification — by week, shift and day of week, with chi-square tests
3. p-charts with variable control limits; phase split at the change point
4. Cleaning — dropped 116 constant and 28 >50%-missing parameters; median-imputed the remaining
   1.6%, retaining 446 parameters
5. Hypothesis testing — Welch t-test and Mann-Whitney across all parameters, Benjamini-Hochberg
   FDR correction, Cohen's d for effect size
6. Confounding analysis — phase-stratified retesting; parameter-vs-phase comparison
7. FMEA and corrective action planning

### Decision log

| Decision | Reason | Risk |
|---|---|---|
| Dropped 116 constant parameters | No variation, cannot explain pass/fail | May be stuck sensors; flagged for maintenance |
| Dropped 28 parameters >50% missing | Too sparse for reliable statistics | Could hide a signal; listed for review |
| Median-imputed remaining 1.6% | Robust to outliers; dropping rows would cost 1,106 of 1,567 runs | Assumes missing-at-random, unverified |
| Kept all runs, tagged by phase | The two periods behave differently and are analysed separately | — |

---

## Limitations

- Spec limits are not available in this dataset, so Cp and Cpk could not be calculated.
- The phase boundary at week 35 was derived from yield data, not from a documented event.
- No parameter showed a robust association with failure within the stable period; conclusions are
  limited to process-level behaviour and require confirmation from production records.
- 80 parameters reached raw p < 0.05 and 22 survived FDR correction — a reminder that testing 446
  hypotheses at α = 0.05 would produce roughly 22 false positives by chance alone.

---

## Running it

See `data/README.md` for the dataset, then run `notebooks/01_baseline.ipynb`.

Requires: pandas, numpy, scipy, matplotlib

## Source

McCann, M. & Johnston, A. (2008). *SECOM*. UCI Machine Learning Repository.
https://archive.ics.uci.edu/dataset/179/secom
