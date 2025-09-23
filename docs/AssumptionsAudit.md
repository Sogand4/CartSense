# Assumption Audit (Checklist)

Purpose: make your assumptions explicit, test them, and record how results change.

## Project / Sprint
- Project: Cart Abandonment Predictor
- Sprint window: Sept 11, 2025 - Sept 23rd, 2025

---

## Assumptions

| Assumption | Why it might fail | Test (design-only or future) | Result (so far) | Impact on conclusions |
|------------|------------------|------------------------------|-----------------|------------------------|
| Site-level features (requires account, shipping/payment options) are predictive of abandonment | Baymard study may not generalize to all retailers; some make account creation painless (e.g., OAuth) | **Design-only:** literature scan + plausibility check across 3 retailers’ checkout flows. **Future:** validate predictive weight in pilot data. | Confirmed variability (OAuth vs mandatory registration). No code run yet. | Still valuable predictors, but effect size may differ; must validate per retailer. |
| Dropping session IDs removes surveillance risk without killing model performance | Removing IDs means we can’t calibrate across repeat abandoners | **Design-only:** schema inspection + PIA trade-off table (Options A/B/C). **Future:** compare calibration with/without IDs on synthetic or pilot data. | Option C selected (no IDs). Prevents tracking. | Privacy preserved; calibration may weaken; rely on cart-level features until further evidence. |
| Logistic regression is sufficient for adoption (interpretability > small accuracy gain) | Random forest may significantly outperform | **Design-only:** literature review of AUC gaps on similar imbalanced datasets. **Future:** train random forest model on same features in prototype, compare to logistic regression. | Literature shows logistic regression is often competitive on tabular business data, especially when features are aggregated/engineered rather than raw. Also simpler to deploy and explain compared to ensembles. | Interpretability justifies choice for now; if gap >10% in future tests, revisit. |
| Free-tier provisioning (25 RCUs/WCUs) covers expected load | Viral spikes might exceed provisioned limits, causing throttling | **Back-of-envelope now:** 25 WCU ≈ 2.1M writes/day vs surge = 36.5M/month. **Future:** run AWS calculator + synthetic load probe. | Math shows fits free tier under assumptions; no load test yet. | Safe for 12-month trial; after expiry switch to on-demand or add auto-scaling. |
| SLA: p95 latency < 100 ms at 100 RPS | Cold starts or bursty load may push latency higher | **Future:** synthetic probe plan (1k requests, measure p95). **Design-only:** queuing math. | Not tested yet. | p95 < 100 ms at 100 RPS. | SLA may need container fallback if violated. |
| Privacy guardrails: no PII or unique aggregates | Rare features or schema creep could introduce leakage | **Design-only:** schema inspection. **Future:** automated event scan. | No disallowed fields in schema. | No disallowed fields; all aggregates k ≥ 10. | Keeps project compliant with PIPA/PIPEDA. |
| No feature leakage after prediction time (T0) | Developer might add “future” fields (e.g., payment status) | **Design-only:** feature timeline table. **Future:** regression test on feature schemas. | No leakage fields included. | 0 features sourced after T0. | Ensures validity of predictions. |

---

## Sensitivity / Spec Grid
- Site-level features: literature + flow check → range from strong to weak predictors. **Future:** validate weights in real data.  
- Session IDs: design-only PIA trade-off → only Option C meets privacy guardrails. **Future:** check calibration loss in pilot.  
- Model choice: literature suggests ≤5% gap. **Future:** confirm empirically with random forest.
- Cost: back-of-envelope math supports provisioned free tier. **Future:** rerun calculator with real traffic traces.  
- SLA latency: No probe yet; design target = p95 < 100 ms at 100 RPS. Future: load test with 1k synthetic requests.  
- Privacy guardrails: Schema inspection confirms no PII fields; future automated schema scan required.  
- No leakage assumption: Feature timeline table verified; no T+ fields (e.g., payment status) included. Future: regression test when schema evolves.  

---

## Actions Taken
- **Kept** site-level features, flagged for validation per retailer.  
- **Dropped** session IDs permanently, rationale logged in PIA.  
- **Kept** logistic regression as default; will revisit if accuracy gap >10% emerges.  
- **Chose** provisioned mode in free tier; plan to switch to on-demand post-trial.  
- **SLA latency:** Kept design target p95 < 100 ms at 100 RPS. No test yet; future action = synthetic load probe with 1k requests.  
- **Privacy guardrails:** Kept schema assumption (no PII); will enforce automated schema scan in future iterations.  
- **No leakage:** Kept timeline table assumption; confirmed no T+ features (e.g., payment status). Future regression tests required when schema evolves.  