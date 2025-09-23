## Cart Abandonment Predictor

TODO: glossary + connect links
- is it okay i changed to PIPA? does no where else in the specs mentio nthe other ones?
- not keeping sessionId. so idk how to tell if my predictions were right?
- PIA: pivacy covered from section 4 rubric?
- TA feedback

### 1) User & Decision

User: Marketing/retention teams across different e-commerce retailers (multi-tenant SaaS offering)

Decision: Trigger a retention strategy (send reminder email, offer discount, free shipping, etc.).

*Insight:* By designing this as a multi-tenant SaaS, each retailer benefits from a shared model while tenant isolation ensures no cross-retailer data leakage.

### 2) Target & Horizon

Target: Predict if this cart will be purchased (1) or abandoned (0)?

Horizon: Next 24 hours after last cart activity.

### 3) Features (No Leakage)

**Cart-level features:** 
- Cart value (total $).
- Number of items.
- Time since last activity.
- Device type (mobile/desktop).
- Shipping speed

**Site-level features (cached per retailer):**  
*Based on ([high inidcators of cart abandonment](https://baymard.com/lists/cart-abandonment-rate))*
- Whether site requires an account in order to purchase products
- Number of shipping options offered  
- Number of payment options offered 

**Exclusions:**  
- Payment status (leaks future)
- Any information after checkout event  
- Any cross-retailer data (so each client only sees its own predictions)

### 4) Baseline to Model Plan

Baseline: Predict “abandoned” for all carts below $20 value.
Model: Logistic regression

- Binary classification
- Each feature is assigned a weight -> simple and interpretable model to explain to stakeholders
- Low latency inference (matrix multiply)
- Easy to deploy
- Captures interaction effects (e.g., large cart *and* long inactivity may be more likely to be highly abandoned)

Other models considered:

- Decision trees: More flexible, but doesn't capture interactions as well, less interpretable, and may overfit on small data
- Random forests: Stronger, but heavier to train/serve and so is likely overkill for this project

Baseline and logistic regression are applied per-retailer, but trained to generalize across multiple stores. Guardrails ensure no leakage between tenants.

Hypothesis: Logistic regression will outperform the baseline by combining multiple weak predictors (cart value, item count, inactivity, device type) rather than relying on a single rule.

See [measurement plan](./ProjectSpec.md/#10-measurement-plan) for more information

### 5) Metrics, SLA, and Cost

**Metrics used:**  
- AUC-PR: Better for our case because there is class imbalance ([abandonment is more frequent](https://www.statista.com/statistics/477804/online-shopping-cart-abandonment-rate-worldwide/)).  
- Precision/recall at operating point: ensures quality of alerts to marketing teams.  

**SLA:**  
- Latency: p95 < 100 ms  
- Cost: ≤ free tier under 100 requests/day  
- See [measurement plan](./ProjectSpec.md/#10-measurement-plan) for more information  

**Cost Envelope (AWS, with Free Tier)**  

| Component        | Normal Load (100 req/day)                | Viral Spike (50k req/hour ≈ 36.5M/month)        | Notes |
|------------------|-------------------------------------------|--------------------------------------------------|-------|
| API Gateway      | ~73k req/month → **$0** (within 1M free)  | 36.5M req/month → **~$127**                     | Main cost driver at high traffic (billed per request after 1M) |
| Lambda (compute) | ~7k ms/month → **$0** (within 400k GB-s free) | 36.5M req at 100 ms → **~$7.10**                | Cold start latency possible |
| DynamoDB (no caching) | 25 RCUs/WCUs provisioned → **$0**       | Auto scales; **~$10–15/month** based on write/read pricing + unit counts | Free tier covers baseline; surge gives nontrivial DB cost without caching |
| DynamoDB (with config caching) | 25 RCUs/WCUs provisioned → **$0** | **≈ $0–1/month** (only for cache misses / config refreshes) | Very small number of reads if configs are cached in memory or DAX |
| Storage (logs)   | ≤25 GB free                              | ≤25 GB free                                      | TTL limits raw ≤14d, aggregates ≤90d; fits free tier unless verbose per-request logging |
| **Total**        | **$0 (fits free tier)**                  | **~$140–150/month at surge (with caching reduces DB portion significantly)** | Surge cost ≈ $1 per 10k predictions; API Gateway remains the dominating cost |

**Design decision:**  
While on-demand mode would better handle unpredictable traffic spikes, it is not covered by the DynamoDB free tier [source](https://aws.amazon.com/dynamodb/pricing/provisioned/) and costs scale per request. Therefore, we use provisioned capacity mode (25 RCUs/WCUs), which fits normal load within the free tier. With auto scaling enabled plus caching of static retailer configs, DynamoDB cost during spikes drops near zero, while API Gateway + Lambda remain primary cost drivers.


With caching of retailer configs, reads drop to near zero because retailer configs are fetched once and reused in memory, keeping surge costs <$1. For writes, we avoid logging every request by only storing **aggregated abandonment metrics per retailer** with 90-day retention. This reduces DynamoDB write volume from millions of rows to a few thousand per month, keeping costs negligible (<$1).

### 6) API Sketch

#### Endpoints Table

| Method | Path                | Purpose                                | Auth Required |
|--------|---------------------|----------------------------------------|---------------|
| POST   | `/v1/predict_cart`  | Return abandonment prediction for cart | Bearer token  |
| POST   | `/v1/retailer`      | Create retailer configuration          | Bearer token  |
| GET    | `/v1/retailer/{id}` | Retrieve retailer configuration        | Bearer token  |
| PATCH  | `/v1/retailer/{id}` | Update retailer configuration          | Bearer token  |

---

#### Request/Response Examples

**Predict Cart**
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
  "retailer_id": "store_12345"
}
```

**Retailer Config**

**Request:**  
`POST /retailer`
```json
{
  "retailer_id": "store_12345",
  "requires_account": true,
  "payment_options": 2,
  "shipping_speeds": 2
}
```

**Response:**  
`POST /retailer`
```json
{
  "retailer_id": "store_12345",
  "requires_account": true,
  "payment_options": 2,
  "shipping_speeds": 2,
  "last_updated": "2025-09-14T10:45:00Z"
}
```

**Get Retailer Config**

**Request**
`GET /retailer/{id}`
```json
GET /v1/retailer/store_12345
Authorization: Bearer <token>
```

**Response:**  
`GET /retailer/{id}`
```json
{
  "retailer_id": "store_12345",
  "requires_account": true,
  "payment_options": 2,
  "shipping_speeds": 2,
  "last_updated": "2025-09-14T10:45:00Z"
}
```

**Update Retailer Config**  
`PATCH /retailer/{id}`
```json
{
  "requires_account": false
}
```

**Response:**  
`PATCH /retailer/{id}`
```json
{
  "retailer_id": "store_12345",
  "requires_account": false,
  "payment_options": 2,
  "shipping_speeds": 2,
  "last_updated": "2025-09-15T09:30:00Z"
}
```

## Error Example 
**Response (JSON):**
```json
{
  "error": "Invalid request",
  "message": "Missing required field: retailer_id",
  "status": 400
}
```

Status Codes

- 200 OK – successful GET/POST/PATCH
- 201 Created – retailer config successfully created
- 400 Bad Request – invalid/missing input
- 401 Unauthorized – missing or invalid token
- 404 Not Found – retailer ID does not exist
- 500 Internal Server Error – unexpected failure

Notes

Versioning: All endpoints are prefixed with /v1/.
Rate limits: 60 requests/minute per retailer.
Idempotency: POST requests may include an Idempotency-Key header to ensure retries are safe.
Tenant isolation: Predictions and configs are scoped by retailer_id; no cross-retailer leakage.

### 7) Privacy, Ethics, Reciprocity (PIA excerpt)

**Data inventory & purpose limitation**  

We collect only 6 minimal cart-level fields (cart value, item count, inactivity hours, device type, shipping speed, retailer_id) plus 3 retailer-level config flags (requires_account, number of shipping/payment options).  
- **Exclusions:** No names, emails, IPs, product SKUs, or session IDs (unlinkable by design).  
- **Purpose limitation:** Data is used solely for cart abandonment prediction; no secondary use (e.g., advertising, resale).  
- **Retention:** Raw logs ≤14 days; aggregate abandonment metrics ≤90 days.  
- **Access:** Retailers see only their own predictions/configs; ops team sees only aggregated metrics.  
See [PIA](./PIA.md) for full details.

**Telemetry decision matrix**  

We keep only high-value, low-invasiveness signals:  
- Request counts, latency (p95), and error rate (for system health).  
- Aggregate abandonment rates ≤90d, and short-term prediction probability logs ≤14d (for calibration).  
Dropped: full cart contents, raw cart/session IDs, and system load metrics (either too invasive or low value).  
See [TelemetryDecisionMatrix.md](./TelemetryDecisionMatrix.md) for full table and rationale.


**Guardrails**  
- **k-anonymity:** All aggregates checked for k ≥ 10 per retailer.  
- **Noise/jitter:** Small random perturbations added to aggregate metrics to reduce inference risk.  
- **TTL enforcement:** Raw request logs expire in ≤14 days, aggregates in ≤90 days.  
- **Tenant isolation:** Strict schema ensures no cross-retailer leakage.  
- **Access controls:** Role-based; audit logs track all developer/admin access.  
- **Disclosure:** Retailers must state in their privacy policy that anonymous cart signals are used to improve checkout.

**Reciprocity**  
- **To retailers:** Actionable predictions to reduce lost sales, plus aggregate benchmarks of abandonment trends.  
- **To customers:** Timely nudges (e.g., discounts, reminders) that reduce friction and improve checkout experience.  
- **Balance:** Value returned mitigates perception of “surveillance”; aggressive use of reminders remains opt-out at retailer level.


### 8) Architecture Sketch (1 diagram)

![Architecture Diagram](/CartSense/docs/Architecture.png)

**Architecture Diagram Code:**  
```
flowchart LR
A[Client: Retailer Checkout] -->|/predict_cart| B[API Gateway / Ingress]
B --> C[Compute: Lambda Predictor]

C --> D[(On-the-fly Feature Computation: inactivity, cart value, etc.)]

C --> E[(DynamoDB: Retailer Config Store, provisioned mode + autoscaling)]
E -. cache .-> C

C --> F[(DynamoDB: Aggregated Metrics ≤90d)]
C --> G[(Observability: logs / metrics ≤14d raw)]

subgraph Guardrails
H["Auth: Bearer token (API Gateway)"]
I["Tenant Isolation: retailer_id scope"]
J["Retention: raw ≤14d; aggregates ≤90d"]
K["Privacy: no PII, k-anonymity ≥10"]
end

G -. review .-> Guardrails
F -. review .-> Guardrails
```

**Trade-offs & Alternatives**

- **Serverless (Lambda)**  
  - Good: Grows easily from a few requests to a big spike (cheap/free at small scale).  
  - Bad: Can be slow at the very first request if the system is “cold.”  
  - Chose this option because it balances cost, elasticity, and simplicity

- **Containers (ECS/Kubernetes)**  
  - Good: Always ready (no “cold start”), more control over how it runs.  
  - Bad: More work to manage, more cost when traffic is low.  

- **Hybrid**  
  - Mix of both: mostly serverless, but keep a few containers “warm” for speed during spikes.  
  - Cache retailer settings in memory for speed; go back to the database only if needed.  

- **Feature Store**  
  - Option 1: Calculate features (like inactivity time) right when the request comes in → simple, but adds a tiny bit of delay.  
  - Option 2: Pre-calculate and store them → faster responses, but more moving parts and harder to keep up-to-date.  
  - Chose option 1 for the simpler design and since logistic regress is quick anyway

- **Retailer Config**  
  - Good: Stored once per retailer, easy to look up, no need to send the same info (like `requires_account`) every time.  
  - Alternative: Put all retailer info in every `/predict_cart` call → simpler design, but wastes bandwidth and could get inconsistent.  
 


### 9) Risks & Mitigations

**1. Misfire risk (false positives/negatives)**  
- **Risk:** Model predicts “abandoned” when the cart would have been purchased (false positive), or misses true abandonments (false negative).  
- **Mitigation/Test:** Evaluate precision and recall offline.  
- **Acceptance test:** Precision ≥ 0.70 at chosen operating point.  

**2. Tail latency risk**  
- **Risk:** Most requests are fast, but a small % take too long (e.g., cold starts in serverless).  
- **Mitigation/Test:** Run a load test with 1,000 synthetic requests, measure p95 latency.  
- **Acceptance test:** p95 < 100 ms.  

**3. Cost drift under viral spikes**  
- **Risk:** Costs rise above budget when traffic jumps to 50k requests/hour.  
- **Mitigation/Test:** Estimate costs with serverless pricing calculators.  
- **Acceptance test:** <$1 per 10k predictions at surge load.  

**4. Privacy & ethics risk**  
- **Risk:** Retailers could misuse predictions (e.g., aggressive reminders) leading to perceived surveillance, even without PII.  
- **Mitigation/Test:** Guardrails in PIA (no PII, TTL ≤14d raw, k-anonymity for aggregates, transparency in policies).  
- **Acceptance test:** Schema scan shows no disallowed fields; all aggregates meet k ≥ 10.  

**5. Multi-tenant isolation risk**  
- **Risk:** One retailer accidentally accesses another retailer’s predictions or metrics.  
- **Mitigation/Test:** Tenant scoping by `retailer_id`, strict auth checks, isolation tests.  
- **Acceptance test:** Unit/integration tests verify no cross-tenant data returned; access logs confirm isolation.  

**6. Model drift risk**  
- **Risk:** Customer behavior changes over time, degrading prediction accuracy.  
- **Mitigation/Test:** Monitor aggregate abandonment rates vs. predicted probabilities; retrain periodically. 
- **Acceptance test:** Predicted probabilities remain aligned with observed abandonment rates within an acceptable tolerance.  

### 10) Measurement Plan

**Offline Model Evaluation**  
- Run logistic regression on historical cart data and compare AUC-PR vs baseline (always predict “abandoned” if cart < $20).  
- **Acceptance test:** Logistic regression achieves higher AUC-PR than baseline by combining multiple features.  

**Latency SLA Test**  
- Simulate 1,000 `/predict_cart` requests with synthetic clients, record response times, then compute p95 latency.  
- **Acceptance test:** p95 latency < 100 ms.  

**Cost SLA Test**  
- Estimate costs under different loads using AWS pricing calculator:  
  - **Normal load (100 req/day):** must fit within AWS free tier quotas (Lambda + DynamoDB).  
  - **Viral spike (50k req/hour):** autoscaling serverless compute with caching expected to keep costs ≤ $1 per 10k predictions.  
- **Acceptance test:** Costs remain within free tier at normal load; ≤ $1 per 10k predictions at surge load.  

**Multi-Tenant Isolation**  
- Confirm predictions are isolated per `retailer_id`.  
- Test: request predictions using another retailer’s token or ID.  
- **Acceptance test:** Cross-retailer access is blocked (403 Unauthorized).  

### 11) Evolution & Evidence
Link a git hash (or range/tag) that shows the design’s evolution (commits, README updates, diagrams). 

[Insight memo](./InsightMemo.md)  
[Assumption Audit](./AssumptionsAudit.md)   
[Socratic Log References](./AiSocraticLog.md) 