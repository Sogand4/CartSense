# Privacy Impact Assessment (PIA)

## 1. Overview

**Project name:** CartSense - A Cart Abandonment Predictor  
**Purpose:** Provide e-commerce retailers with a probability estimate that a customer’s cart will be abandoned, enabling timely interventions (e.g., reminder email, discount popup).  
**Scope:** Multi-tenant prediction API available to multiple retailers.  
**Data processing summary:**  
- **Collection:**  
  - Cart-level features: cart value, item count, inactivity hours, device type, shipping speed.  
  - Retailer-level configs: requires_account flag, shipping speeds offered.  
- **Processing:** Logistic regression computes abandonment probability.  
- **Sharing:** Prediction returned only to requesting retailer. No cross-tenant sharing.  
- **Retention:** Raw logs ≤14 days; aggregated abandonment metrics ≤90 days.

## 2. Data Inventory

**Identifiers (retailer_id, session/cart IDs):**
- **Option A:** Store raw user/session/cart IDs → enables precise linking of carts over time but creates surveillance risk.  
- **Option B:** Hash session/cart IDs with daily [salt](./Glossary.md) → allows session-level calibration while reducing linkability across days.  
- **Option C (Selected):** Do not store session/cart IDs at all → prevents any cross-session tracking; rely on retailer_id only for tenant isolation.  
**Rationale:** Chose Option C to eliminate re-identification risk entirely, sacrificing fine-grained session analysis.

**Raw fields (per prediction request):** 
- `retailer_id` → Tenant scoping; purpose: isolate predictions; retention ≤90d; access = system only.  
- `cart_value` → Purpose: model input; minimization: numeric only (in CAD), no product details; retention ≤14d raw.  
- `item_count` → Purpose: model input; retention ≤14d raw.  
- `inactivity_hours` → Purpose: model input; retention ≤14d raw.  
- `device_type` (mobile/desktop) → Purpose: model input; retention ≤14d raw.  
- `shipping_speed` (chosen option, e.g., paypal/credit card) → Purpose: model input; retention ≤14d raw. 
- **Alternatives considered:**  
  - Store full SKU/product lists → higher predictive power, but high risk (unique baskets could re-identify users).  
  - Truncate to numeric summaries only (selected) → preserves predictive signal while minimizing invasiveness.  
 
**Retailer-level config fields (stored in config store and not sent each time, updated explicitly by retailer):**
- `requires_account` (boolean) → higher abandonment likelihood if true. Retained until updated/deleted by retailer.  
- `num_shipping_options` (int >= 0) → used as a signal in predictions. Retained until updated/deleted by retailer.  
- `num_payment_options` (int >= 0) → used as a signal in predictions. Retained until updated/deleted by retailer.
- **Alternatives considered:**  
  - Store full lists of payment/shipping methods → leaks competitive intelligence and unnecessary detail.  
  - Store only counts/booleans (selected) → reduces detail but captures abandonment risk signal.  

**Derived / aggregate fields (for monitoring, not prediction):**
- Aggregate abandonment rate per retailer (≤90d).
- Prediction probability logs (for calibration) (≤14d).
- Request count, latency, error rate (system health telemetry) (≤90d).
- **Alternatives considered:**  
  - Keep raw per-request logs indefinitely (high invasiveness).  
  - Keep raw logs short-term + aggregate metrics long-term (selected).  
**Rationale:** Balances calibration/ops needs with privacy (short TTLs, aggregation).  

**Metadata automatically collected by infrastructure:**
- Request timestamp (API Gateway, DB logs).
- API key / authentication token (for rate limits).
- Response time (Lambda logs).
- Cloud provider access logs (ALB/CloudWatch).
→ Retention follows default cloud log TTL unless restricted (≤14d raw, ≤90d aggregated).
- **Alternatives considered:**  
  - Retain indefinitely in CloudWatch (default).  
  - Enforce TTL ≤14d raw, ≤90d aggregate (selected). 

**Lawful basis:** Retailer collects data under their terms of service; API processes only aggregated/cart-level data.  
**Minimization:** No PII collected (no names, emails, addresses, IPs, or session IDs).  
**Access roles:** Retailers see only their predictions/configs; ops team sees aggregate metrics; no cross-retailer access.  
**Could be derived:** From repeated cart values + inactivity, one could infer browsing session length or total spend trends (guarded via TTL + aggregation).  


## 3. Linkability & Identifiability

- **Option A:** Link carts across sessions using stable IDs → enables longitudinal profiling (surveillance risk).  
- **Option B:** Hash IDs with salt → enables daily session analysis but still allows short-term profiling.  
- **Option C (Selected):** Collect no user/session IDs → each prediction request is unlinkable across sessions.  
**Rationale:** Eliminates linkage risk; limits analysis to cart-level only.

## 4. Purpose Limitation & Secondary Use

- Declared purpose: cart abandonment prediction.  
- No secondary use allowed (e.g., advertising, resale).  
- Any new purpose would require explicit re-approval and disclosure to retailers.  
- Protections: API schema limits fields; no PII accepted.

## 5. Minimization & Retention

- **Cart features:** 6 minimal fields only. SKUs, user IDs, IPs explicitly excluded.  
- **Retailer config:** Reduced to counts/booleans, not full option lists.  
- **Logs:**  
  - Raw logs (requests, probabilities) TTL ≤14d.  
  - Aggregates (abandonment rate, latency) TTL ≤90d.  
- **Alternatives considered:**  
  - Indefinite log retention (higher ops convenience, high risk).  
  - Bucketing/truncation (medium ops cost, reduces granularity).  
  - Short TTLs (selected) → balances utility and privacy.  

[Secure deletion](./Glossary.md):  
Logs are automatically replaced in rotation and database TTL policies handle expiration of each record

## 6. Access Control & Governance

- **Option A:** Developers have full DB access → risk of insider misuse.  
- **Option B:** Role-based access control; retailers see their own metrics only; ops see only aggregate metrics (selected).  
- **Rationale:** Preserves monitoring value while minimizing insider surveillance capability.  

- Roles:  
  - Retailer: access only to their predictions and aggregated metrics.  
  - Developers: least-privilege, limited to monitoring system health.  
- Audit logs track all admin access for extra security
- Hosted in AWS (Lambda for compute, DynamoDB/S3 for storage); no additional third-party processors


## 7. Transparency & Choice

- **Cart-level features collected:** cart value, item count, inactivity time, device type, shipping speed
  - *Why:* These are needed to estimate the likelihood a cart will be abandoned.  

- **Retailer-level configs collected:** whether an account is required, shipping speeds offered, payment types offered  
  - *Why:* These settings strongly affect abandonment behavior and are used as model inputs.

- Retailers disclose in their privacy policy: “We use anonymous cart signals (e.g., cart value, item count, device type) and store settings (e.g., checkout requirements, shipping options) to improve checkout experience.”  
- API provider publishes summary of what data is collected and why.  
- **Choice:** End users may decline reminders or marketing nudges at the retailer level (opt-out handled by retailer).  

## 8. Security & Threat Modeling  

**Threat scenarios, surveillance capabilities, and mitigations:**  

- **Linkage attacks:** Repeated cart values + inactivity + timestamps *could be linked across requests to profile a single user’s shopping behavior over time.*  
  *Mitigation:* No user/session IDs collected; raw logs TTL ≤14d; aggregation prevents cross-session profiling.  

- **Function creep:** Abandonment logs *could be reused to build advertising profiles or track customers beyond checkout.*  
  *Mitigation:* API schema enforces cart-level fields only; secondary uses explicitly disallowed; any new purpose requires re-approval.  

- **Third-party leakage:** Cloud logs (API Gateway, ALB, CloudWatch) *could expose request metadata to AWS operators or misconfigured [IAM](./Glossary.md) roles.*  
  *Mitigation:* TTL ≤14d for raw logs; strict IAM policies; no external processors beyond AWS.  

- **Insider threats:** Developers with broad DB access *could extract retailer-specific customer behavior and share it internally or externally.*  
  *Mitigation:* Role-based access control; retailers see only their own metrics; ops team sees only aggregates; audit logs capture all admin access.  

- **Retention risks:** “For debugging” logs kept indefinitely *would create a de facto surveillance archive of shopping behavior.*  
  *Mitigation:* Automated log rotation + TTL policies; aggregates only retained ≤90d.  

- **Abuse / scraping / denial of service:** Attackers *could query the API repeatedly to reverse-engineer model outputs or overwhelm resources.*  
  *Mitigation:* Authentication (API keys), WAF + rate limiting, tenant isolation per request.  

- **Inference attacks on aggregates:** Rare cart patterns *could let a retailer infer competitors’ customer behaviors.*  
  *Mitigation:* Add small jitter/noise to aggregate metrics to reduce inference risk.  

**Secrets & transport security:**  
- API keys stored in environment variables, not code.  
- All transport via HTTPS.  


## 9. Compliance & Policy Alignment

- **Course policy:** Design follows course guidelines — no personally identifiable information (PII) is collected; solution is scoped to free-tier cost assumptions under normal load.  
- **Organizational policy:** Retailers are responsible for obtaining customer consent for reminders/marketing nudges. API provider processes only anonymous cart-level features and retailer configs.  
- **Legal frameworks:** Aligned with [PIPA](./Glossary.md) (BC) & [PIPEDA](./Glossary.md) (Canada):
  - **Data minimization:** Only 5 cart-level fields collected (cart value, item count, inactivity, device type, shipping_speeds_offered).  
  - **Purpose limitation:** Data is used solely for cart abandonment prediction, not advertising or profiling.  
  - **Retention limits:** Raw logs TTL ≤14 days, aggregated metrics ≤90 days, enforced by database TTL policies.  
- **Security compliance:** All transport over HTTPS; least-privilege access; secrets stored in environment variables.

## 10. Residual Risks & Trade-offs

- **False positives / customer annoyance:** Predictions may sometimes mark carts as abandoned when they would be purchased, leading to unnecessary reminders. *Trade-off:* Higher recall helps retailers recover more lost sales. *Mitigation:* Allow retailers to tune operating threshold.  
- **Cold-start latency in serverless compute:** A small percentage of requests may exceed latency target due to Lambda warm-up. *Trade-off:* Serverless is cheaper and simpler at baseline scale. *Mitigation:* Pre-warming or hybrid container fallback could reduce this.  
- **Perception of “surveillance”:** Even without PII, customers may feel tracked if reminders are too aggressive. *Trade-off:* Abandonment prediction adds retailer value. *Mitigation:* Reciprocity ensures value returned to customers (timely discounts, fewer irrelevant nudges); disclosure in retailer privacy policies.  

## 11. Data We Chose NOT to Collect  

To meet functional requirements while minimizing surveillance risk, we explicitly rejected the following fields:  

- **Full SKU / product lists:** Higher predictive power, but unique baskets could re-identify individual users.  
- **Session/cart IDs:** Would enable longitudinal tracking of users; dropped to ensure unlinkability across sessions.  
- **User IP / User Agent:** Available in infrastructure logs, but not used or stored; prevents device/browser fingerprinting.  
- **Full lists of payment/shipping methods:** Reduced to counts/booleans only; avoids leaking competitive intelligence while still capturing abandonment risk signals.  

## 12. Sign-off
- **Team member:** Sogand Haji-Salimi  
- **Date:** 2025-09-14  
- **Version:** v1.0  

**Attachments:**  

- [Telemetry Decision Matrix](./TelemetryDecisionMatrix.md)
- [PrivacyStance](./PrivacyStance.md)
- [Architecture diagram](./ProjectSpec.md/#8-architecture-sketch-1-diagram)
- [Project spec](./ProjectSpec.md)
