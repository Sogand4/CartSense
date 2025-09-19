# Socratic Log (AI as Thinking Partner)

**Project:** Cart Abandonment Predictor  
**Sprint window:** Sept 11 – Sept 23, 2025  
**Author:** Sogand Haji-Salimi  

---

## Context
Exploring design trade-offs for a multi-tenant SaaS that predicts cart abandonment while meeting privacy, cost, and adoption constraints.

---

### Prompt A (design alternatives)
“Should we include session IDs, hash them, or drop them completely?”  
→ AI outlined 3 options: store IDs (profiling power, high risk), hash with salt (partial protection), or drop IDs entirely (no cross-session calibration).  

### Prompt B (red-team)
“Argue why dropping session IDs is a bad idea.”  
→ Risk: we lose calibration across repeat abandoners, and accuracy could drift without longitudinal linkage.  

**Inflection point**  
I realized structural site-level features (requires account, shipping/payment options) can carry strong predictive power even without user/session IDs. This reframed the design: unlinkability is not just ethical but also feasible.  

**Evidence**  
Baymard Institute stats on cart abandonment + my PIA trade-off table (Options A/B/C).  

**Outcome**  
Dropped session IDs entirely; updated the PIA and telemetry matrix to reflect unlinkability as a core guardrail.  

---

### Prompt A (design alternatives)
“What model should we choose for prediction: logistic regression, decision trees, or random forests?”  
→ AI highlighted trade-offs: LR = interpretable, lightweight; trees/ensembles = more accuracy, less transparency.  

### Prompt B (red-team)
“Convince me that logistic regression is not enough.”  
→ Risk: tree ensembles could significantly outperform and make LR look naïve.  

**Inflection point**  
I saw that interpretability directly impacts adoption by marketing teams. Accuracy alone is not the deciding factor; trust and explainability matter more.  

**Evidence**  
Caruana & Niculescu-Mizil (2006) survey + Kaggle reports showing LR is often competitive on tabular data.  

**Outcome**  
Chose logistic regression as baseline model. Logged assumption that if gap >10% AUC emerges in future testing, switch to more complex models.  

---

### Prompt A (design alternatives)
“How should we handle cost and scaling under normal vs viral traffic?”  
→ AI compared serverless vs containers vs hybrid, with trade-offs on cost, latency, and idle overhead.  

### Prompt B (red-team)
“Why not just stick with containers?”  
→ Containers give control and avoid cold starts, but at high idle cost under low traffic.  

**Inflection point**  
The realization was that viral surges (50k req/hour) are the actual bottleneck, not normal traffic (100 req/day). Cost at surge load (<$1 per 10k predictions) matters more than shaving a few ms latency.  

**Evidence**  
AWS Lambda calculator runs showing $7/month for 36M requests under surge scenario.  

**Outcome**  
Adopted serverless-first architecture with caching. Containers considered only as hybrid fallback.  

---

## Attributions
- **AI:** Generated trade-off options (session ID design, model selection, serverless vs container), suggested red-team perspectives, provided AWS calculator guidance, and surfaced literature sources.  
- **Me:** Chose final design directions, validated assumptions with public benchmarks (Baymard, Caruana et al.), drafted PIA/insight memo, and structured evidence.  
