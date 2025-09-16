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

**Retailer-level config fields (stored in config store, not sent each time):**  
- `requires_account` (boolean) → higher abandonment likelihood if true. Retained until updated/deleted by retailer.  
- `shipping_speeds` (e.g., standard/express/overnight) → used as a signal in predictions. Retained until updated/deleted by retailer.  
- `payment_options` (e.g., paypal/credit card) → used as a signal in predictions. Retained until updated/deleted by retailer.

**Lawful basis:** Retailer collects data under their terms of service; API processes only aggregated/cart-level data.  
**Minimization:** No PII (no names, emails, addresses).         
**Access roles:** Retailer has access only to their predictions/configs.    

## 3. Linkability & Identifiability
Can events be linked across sessions/users? How? (e.g., referrer+time+token)  
Quasi-identifiers present (IP, UA, location) and re-identification risks.

## 4. Purpose Limitation & Secondary Use
Declared purposes. Process to approve new purposes.  
Protections against function creep.

## 5. Minimization & Retention
Aggregation/sampling strategies; bucketing/truncation; hashing/anonymization.  
Raw TTL (days); aggregate retention; secure deletion plan.

## 6. Access Control & Governance
Roles and least-privilege policies. Auditability of access.  
Third parties involved (if any) and contracts.

## 7. Transparency & Choice
“What we collect & why” communication plan.  
Opt-in/opt-out mechanisms where feasible.

## 8. Security
Threats (abuse, scraping, doxxing) and mitigations (WAF, rate limits, jitter).  
Secrets management and transport security.

## 9. Compliance & Policy Alignment
Policies (course policy, org policy, legal frameworks) and how you meet them.

## 10. Residual Risks & Trade-offs
Top residual risks, business/ethical rationale, and contingency measures.

## 11. Sign-off
Team members, date, version.  
Attach your telemetry decision matrix and any diagrams that clarify data flows.
