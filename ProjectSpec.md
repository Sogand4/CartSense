### 1) User & Decision

User: E-commerce operations/merchandising team

Decision: Flag products/orders with high return risk → improve sizing guides, adjust descriptions, or prepare logistics

### 2) Target & Horizon

Target: P(return within 30 days) (binary: return / no return)

Horizon: 30 days post-purchase

### 3) Features (No Leakage)

Product features: category (clothing, electronics, etc.), brand, size variant, price point

Order context: first-time buyer flag, bulk order indicator, applied discount code

Historical aggregates: return rate for this product in past 90 dayss, return rate for category overall

Time features: season (holiday vs off-season), day of week of purchase

Exclude (no leakage): actual return request status, customer identifiers (PII), shipping info updates after purchase

### 4) Baseline → Model Plan
Baseline you can implement immediately (rule/heuristic).  
One simple model (e.g., logistic/tree/ETS) and why it’s better (hypothesis).

---

### 5) Metrics, SLA, and Cost
Metric(s): AUC-PR/MAE/etc. State why they fit harms/benefits.  
SLA: p95 latency, max cost per 10k predictions.

---

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
