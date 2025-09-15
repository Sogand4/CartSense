## Cart Abandonment Predictor

### 1) User & Decision

User: Marketing/retention teams across different e-commerce retailers (multi-tenant SaaS offering)

Decision: Trigger a retention strategy (send reminder email, pop-up discount, etc.).

### 2) Target & Horizon

Target: Will this cart be purchased (1) or abandoned (0)?

Horizon: Next 24 hours after last cart activity.

### 3) Features (No Leakage)

- Cart value (total $).
- Number of items.
- Time since last activity.
- Device type (mobile/desktop).
- Shipping speed

Site level features:

- If site requires an account in order to purchase products ([High inidcator of cart abandonment](https://baymard.com/lists/cart-abandonment-rate)) - Binary flag (1=yes, 0=no)
- Shipping speeds offered
- Payment options available

Exclusions: payment status (leaks future), any info after checkout event, data from other retailers (each client only sees its own predictions)

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

Baseline and logistic regression are applied per-retailer, but trained to generalize across multiple stores. Guardrails ensure no leakage between tenants.

Hypothesis: Logistic regression will outperform the baseline by combining multiple weak predictors (cart value, item count, inactivity, device type) rather than relying on a single rule.

### 5) Metrics, SLA, and Cost
Metric(s): AUC-PR/MAE/etc. State why they fit harms/benefits.  
SLA: p95 latency, max cost per 10k predictions.

Metric: AUC-PR ([abandonment is more frequent](https://www.statista.com/statistics/477804/online-shopping-cart-abandonment-rate-worldwide/), so class imbalance).

SLA: p95 latency < 100 ms; cost ≤ free tier under 100 requests/day.

- Logistic regression + precomputed features should keep the API relatively fast and meet this threshold

Cost Envelope:

- 100 req/day → within free tier (AWS Lambda + DynamoDB).
- 50k req/hour spike → autoscaling serverless compute + caching, expected <$1 per 10k predictions.

Multi-tenant note: Predictions are isolated per retailer; no retailer’s data is exposed to others.

### 6) API Sketch

**Endpoint:**  
`POST /predict_cart`

**Request (JSON):**
```json
{
  "retailer_id": "store_12345",
  "cart_value": 45.99,
  "item_count": 3,
  "inactivity_hours": 12,
  "device_type": "mobile",
  "shipping_speed": "standard",
}
```

**Response (JSON):**
```json
{
  "prediction": "abandoned",
  "probability": 0.78,
  "model_version": "v1.0",
  "retailer_id": "store_12345"
}
```

Notes: 
- prediction is binary: "abandoned" or "purchased".
- probability = model confidence (0.0–1.0).
- model_version enables monitoring and rollback.
- retailer_id ensures multi-tenant isolation (no cross-retailer leakage).

**Endpoint:**  
`POST /retailer`
```json
{
  "retailer_id": "store_12345",
  "requires_account": true,
  "payment_options": ["credit_card", "paypal"],
  "shipping_speeds": ["standard", "express"]
}
```

**Endpoint:**  
`GET /retailer/{id}`
```json
{
  "retailer_id": "store_12345",
  "requires_account": true,
  "payment_options": ["credit_card", "paypal"],
  "shipping_speed": "standard",
  "last_updated": "2025-09-14T10:45:00Z"
}
```

**Endpoint:**  
`PATCH /retailer/{id}`
```json
{
  "requires_account": false
}
```

Notes:

- Retailer configs are managed separately (efficient, scalable).  
- `/predict_cart` stays lightweight (cart-level only).
- Features such as `requires_account` are site-level and constant per retailer, cached for efficiency.

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

**Offline Model Evaluation**  

- Run logistic regression on historical cart data and compare AUC-PR vs baseline score. Expect logistic regression to outperform baseline by leveraging multiple features instead of one threshold.

**Latency SLA Test**  

- Simulate 1,000 `/predict_cart` requests with synthetic clients, record response times, then compute p95 latency.

**Cost SLA Test**  
- Estimate costs under different loads:  
  - **Normal load (100 req/day):** fits within AWS/GCP free tier quotas (Lambda + DynamoDB).  
  - **Viral spike (50k req/hour):** autoscaling serverless compute with caching expected to keep costs ≤ $1 per 10k predictions.  

**Multi-Tenant Isolation**  
- Confirm that predictions are isolated per `retailer_id`.  
- Run tests to ensure no data from one retailer can be accessed or inferred by another.

### 11) Evolution & Evidence
Link a git hash (or range/tag) that shows the design’s evolution (commits, README updates, diagrams).  
Insight memo link (3 insights), assumption audit, and Socratic log references.
