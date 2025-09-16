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
Policies (course policy, org policy, legal frameworks) and how you meet them.

## 10. Residual Risks & Trade-offs
Top residual risks, business/ethical rationale, and contingency measures.

## 11. Sign-off
Team members, date, version.  
Attach your telemetry decision matrix and any diagrams that clarify data flows.
