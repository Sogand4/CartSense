## Cart Abandonment Predictor

### 1) User & Decision

User: E-commerce marketing/retention team.

Decision: Trigger a retention strategy (send reminder email, pop-up discount, etc.).

### 2) Target & Horizon

Target: Will this cart be purchased (1) or abandoned (0)?

Horizon: Next 24 hours after last cart activity.

### 3) Features (No Leakage)

Cart value (total $).

Number of items.

Time since last activity.

Device type (mobile/desktop).

Referral source (direct / ad / search).

Exclusions: payment status (leaks future), any info after checkout event.

### 4) Baseline → Model Plan
Baseline you can implement immediately (rule/heuristic).  
One simple model (e.g., logistic/tree/ETS) and why it’s better (hypothesis).

Baseline: Predict “abandoned” for all carts below $20 value.

Model: Logistic regression on features above. Hypothesis → captures interaction effects (e.g., large cart + long inactivity is highly abandoned).

### 5) Metrics, SLA, and Cost
Metric(s): AUC-PR/MAE/etc. State why they fit harms/benefits.  
SLA: p95 latency, max cost per 10k predictions.

Metric: AUC-PR (abandonment is more frequent, so class imbalance).

SLA: p95 latency < 100 ms; cost ≤ free tier under 100 requests/day.

### 6) API Sketch (if applicable)
POST /predict request/response schema. Include example payloads.

---

### 7) Privacy, Ethics, Reciprocity (PIA excerpt)
Data inventory, purpose limitation, retention, access (link your PIA).  
Telemetry decision matrix (value vs invasiveness vs effort).  
Guardrails: k-anonymity, jitter/aggregation, opt-ins, disclosure.  
Reciprocity: value returned and to whom.

---

### 8) Architecture Sketch (1 diagram)
Major components and data flow. Note trade-offs and alternatives.

---

### 9) Risks & Mitigations
Top 3 risks (technical/ethical) and how you will test or reduce them.

---

### 10) Measurement Plan
One minimal experiment to validate the baseline vs model (offline acceptable).  
How you will measure SLA (latency/cost).

---

### 11) Evolution & Evidence
Link a git hash (or range/tag) that shows the design’s evolution (commits, README updates, diagrams).  
Insight memo link (3 insights), assumption audit, and Socratic log references.
