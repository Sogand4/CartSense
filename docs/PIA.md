# Privacy Impact Assessment (PIA)

## 1. Overview
Project name, purpose, and scope.  
Data processing summary (collection → processing → sharing → retention).

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
Fields collected (raw and derived).  
For each field: purpose, lawful basis (if applicable), minimization, retention, access roles.

**Fields collected:**  
- `retailer_id` → Tenant scoping; purpose: isolate predictions; retention ≤90d; access = system only.  
- `cart_value` → Purpose: model input; minimization: numeric only, no product details; retention ≤14d raw.  
- `item_count` → Purpose: model input; retention ≤14d raw.  
- `inactivity_hours` → Purpose: model input; retention ≤14d raw.  
- `device_type` (mobile/desktop) → Purpose: model input; retention ≤14d raw.  
- `shipping_speed` (e.g., paypal/credit card) → Purpose: model input; retention ≤14d raw.  

**Retailer-level config fields (stored in config store, not sent each time):**  
- `requires_account` (boolean) → higher abandonment likelihood if true. Retained until updated/deleted by retailer.  
- `shipping_speeds` (e.g., standard/express/overnight) → used as a signal in predictions. Retained until updated/deleted by retailer.  
- `payment_options` (e.g., paypal/credit card) → used as a signal in predictions. Retained until updated/deleted by retailer.

**Lawful basis:** Retailer collects data under their terms of service; API processes only aggregated/cart-level data.  
**Minimization:** No PII (no names, emails, addresses).         
**Access roles:** Retailer has access only to their predictions/configs.    

## 3. Linkability & Identifiability

- No direct user identifiers collected.  
- Events cannot be linked across sessions.
- Quasi-identifiers are not collected by API.  
- Re-identification risk considered low.

## 4. Purpose Limitation & Secondary Use

- Declared purpose: cart abandonment prediction.  
- No secondary use allowed (e.g., advertising, resale).  
- Any new purpose would require explicit re-approval and disclosure to retailers.  
- Protections: API schema limits fields; no PII accepted.

## 5. Minimization & Retention

- Only collecting bare minimum data needed: 5 cart-level fields collected.  
- No storage of sensitive raw data beyond TTL.  
- Bucketing/truncation is not applied because no sensitive identifiers are collected, and raw data is retained for ≤14 days only.
- **Raw logs:** ≤14 days.  
- **Aggregates (abandonment rates per retailer):** ≤90 days.  
- Secure deletion: logs are automatically replaced in rotation and database TTL policies handdle expiration of each record

## 6. Access Control & Governance

- Roles:  
  - Retailer: access only to their predictions and aggregated metrics.  
  - Developers: least-privilege, limited to monitoring system health.  
- Audit logs track admin access.  
- Hosted in AWS (Lambda for compute, DynamoDB/S3 for storage); no additional third-party processors

## 7. Transparency & Choice

- **Cart-level features collected:** cart value, item count, inactivity time, device type, shipping speed
  - *Why:* These are needed to estimate the likelihood a cart will be abandoned.  

- **Retailer-level configs collected:** whether an account is required, shipping speeds offered, payment types offered  
  - *Why:* These settings strongly affect abandonment behavior and are used as model inputs.

- Retailers disclose in their privacy policy: “We use anonymous cart signals (e.g., cart value, item count, device type) and store settings (e.g., checkout requirements, shipping options) to improve checkout experience.”  
- API provider publishes summary of what data is collected and why.  
- **Choice:** End users may decline reminders or marketing nudges at the retailer level (opt-out handled by retailer).  

## 8. Security
Threats (abuse, scraping, doxxing) and mitigations (WAF, rate limits, jitter).  
Secrets management and transport security.

- Threats: scraping of predictions, overuse (denial of service), misuse of retailer IDs.  
- Mitigations:  
  - Authentication (API keys).  
  - Tenant isolation enforced on every request.  
  - Transport security: HTTPS.  
  - Secrets stored in environment variables, not code.  
  - Small jitter added to aggregate metrics to prevent inference attacks.


## 9. Compliance & Policy Alignment

- **Course policy:** Design follows course guidelines — no personally identifiable information (PII) is collected; solution is scoped to free-tier cost assumptions under normal load.  
- **Organizational policy:** Retailers are responsible for obtaining customer consent for reminders/marketing nudges. API provider processes only anonymous cart-level features and retailer configs.  
- **Legal frameworks:** Aligned with GDPR/CPRA principles:  
  - **Data minimization:** Only 5 cart-level fields collected (cart value, item count, inactivity, device type, shipping_speeds_offered).  
  - **Purpose limitation:** Data is used solely for cart abandonment prediction, not advertising or profiling.  
  - **Retention limits:** Raw logs TTL ≤14 days, aggregated metrics ≤90 days, enforced by database TTL policies.  
- **Security compliance:** All transport over HTTPS; least-privilege access; secrets stored in environment variables.

## 10. Residual Risks & Trade-offs

- **False positives / customer annoyance:** Predictions may sometimes mark carts as abandoned when they would be purchased, leading to unnecessary reminders. *Trade-off:* Higher recall helps retailers recover more lost sales. *Mitigation:* Allow retailers to tune operating threshold.  
- **Cold-start latency in serverless compute:** A small percentage of requests may exceed latency target due to Lambda warm-up. *Trade-off:* Serverless is cheaper and simpler at baseline scale. *Mitigation:* Pre-warming or hybrid container fallback could reduce this.  
- **Perception of “surveillance”:** Even without PII, customers may feel tracked if reminders are too aggressive. *Trade-off:* Abandonment prediction adds retailer value. *Mitigation:* Reciprocity ensures value returned to customers (timely discounts, fewer irrelevant nudges); disclosure in retailer privacy policies.  

## 11. Sign-off
- **Team member:** Sogand Haji-Salimi  
- **Date:** 2025-09-14  
- **Version:** v1.0  

**Attachments:**  
TODO
- [Telemetry Decision Matrix](./TelemetryDecisionMatrix.md)
- Architecture diagram
- Project spec
