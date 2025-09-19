# Assumption Audit (Checklist)

Purpose: make your assumptions explicit, test them, and record how results change.

## Project / Sprint
- Project:
- Sprint window:

---

## Assumptions

| Assumption | Why it might fail | Test you ran | Result | Impact on conclusions |
|------------|------------------|--------------|--------|------------------------|
| Example: Immigration proxy = permits | Under/over-counts; lagged behavior | Spec A vs A+lag; compare against alt proxy | Coeff ↓ 30% with lag | Weaken claim; note uncertainty |

---

## Design-Only Test Patterns
- Back-of-the-envelope: cost/latency/throughput math (public prices/limits, simple queuing).
- Synthetic probe plan: define how you would measure (e.g., 1 req/min for 10 min; record p95), even if you don’t run it now.
- Offline baseline check: tiny CSV or synthetic sample; compare majority class vs. rule baseline (describe method + metric).
- Schema inspection: assert “no PII/no leakage” fields in event schemas; list disallowed fields you checked for.
- Feature timeline table: show each feature’s availability at prediction time; mark and remove anything arriving after T0.
- Policy trace: map a clause (e.g., retention ≤ 14d) to a control (lifecycle + deletion job) and its acceptance test.
- Privacy guardrails: k-anonymity threshold for public aggregates; acceptance test = no groups with k < 10.

---

## Copy-Ready Examples
- **SLA assumption:** “p95 < 100 ms @ 100 RPS.”  
  – Test method: synthetic probe plan; or calculator estimate of tail latency/cost.  
  – Accept: p95 < 100 ms and cost ≤ $25 per 10k predictions.  

- **Baseline beats trivial:**  
  – Test method: on a 1k-row (or synthetic) sample, compare majority class vs. rule; compute precision/recall.  
  – Accept: rule baseline improves precision by ≥ 10% at similar recall.  

- **No leakage:**  
  – Test method: feature timeline table (T-15m, T-5m, T+...); remove features sourced after prediction.  
  – Accept: 0 features sourced after prediction time.  

- **Privacy guardrails:**  
  – Test method: event schema scan for disallowed fields (e.g., name, email, exact GPS); k ≥ 10 for any public aggregate.  
  – Accept: no disallowed fields; all aggregates meet k ≥ 10.  

- **Cost envelope:**  
  – Test method: AWS calculator for API Gateway + Lambda + egress at target RPS; add 30% buffer.  
  – Accept: ≤ $25 per 10k predictions at expected traffic.  

---

## Sensitivity / Spec Grid
Briefly summarize which specs you ran and how estimates moved. Link to plots/spec-curve.

---

## Actions Taken
Kept/changed/dropped assumptions and why.  
If you can’t test an assumption now, note it and propose a feasible future test.
