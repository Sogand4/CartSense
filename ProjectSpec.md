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

If site requires an account in order to purchase products ([High inidcator of cart abandonment](https://baymard.com/lists/cart-abandonment-rate))

Referral source (direct / ad / search).

Exclusions: payment status (leaks future), any info after checkout event

### 4) Baseline → Model Plan
Baseline: Predict “abandoned” for all carts below $20 value.

Model: Logistic regression

- Binary classification
- Each feature is assigned a weight -> simple and interpretable model to explain to stakeholders
- Low latency inference (matrix multiply)
- Easy to deploy
- Captures interaction effects (e.g., large cart AND long inactivity may be more likely to be highly abandoned)

Other models considered:

- Decision trees: More flexible, but doesn't capture interactions as well, less interpretable, and may overfit on small data
- Random forests Stronger, but heavier to train/serve and so is likely overkill for this project

Hypothesis: Logistic regression will outperform the baseline by combining multiple weak predictors (cart value, item count, inactivity, device type) rather than relying on a single rule.
Offline test: Run logistic regression on historical cart data and compare AUC-PR vs baseline score. Expect logistic regression to outperform baseline by leveraging multiple features instead of one threshold.

### 5) Metrics, SLA, and Cost
Metric(s): AUC-PR/MAE/etc. State why they fit harms/benefits.  
SLA: p95 latency, max cost per 10k predictions.

Metric: AUC-PR ([abandonment is more frequent](https://www.statista.com/statistics/477804/online-shopping-cart-abandonment-rate-worldwide/), so class imbalance).

SLA: p95 latency < 100 ms; cost ≤ free tier under 100 requests/day.

- Logistic regression + precomputed features should keep the API relatively fast and meet this threshold
- Measurement plan: Simulate 1,000 `/predict_cart` requests with synthetic clients, record response times, then compute p95 latency.

Cost Envelope:

- 100 req/day → within free tier (AWS Lambda + DynamoDB).
- 50k req/hour spike → autoscaling serverless compute + caching, expected <$1 per 10k predictions.

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
