# Insight Memo

---

## Project / Sprint
**Project:**  1
**Sprint window:**  Sept 11, 2025 - Sept 23rd, 2025
**Team:**  Sogand Haji-Salimi

---

### 1. Insight: Changing features to include the strongest predictors of abandonment
**Evidence:** Baymard Institute research shows that requiring account creation, shipping methods available, and payment options available are important reasons behind cart abandonment (source: https://baymard.com/lists/cart-abandonment-rate).  

**Why it matters:** These fFeatures are cheap to collect and highly predictive. This demonstrates how not all valuable signals for cart abandonment come from user-level data — some of the strongest predictors are structural site-level decisions.  

**Limits:** Assumes research findings generalize across all retailers. It doesn’t capture nuance (e.g., some sites might have free trial that make account creation temporariliy bypassed).  

**Next question:** How much do our predictions improve by adding site-level configurations as features versus more specific cart-level features vs both? What are the tradeoffs of each?

---

### 2. Insight: Avoiding session IDs prevents a major surveillance risk
**Evidence:** Design trade-off explored in PIA:  
- Option A: Store session/cart IDs: Good data for analysis across sessions; comes with privacy risk for user
- Option B: Hash with salt: Good data for analysis across sessions; less privacy risk for user, but still possible
- Option C: No session IDs (selected) → prevents cross-session tracking and the retail-level features alone could be enough to build a strong model 

**Why it matters:** This was a pivotal decision. It reduced invasiveness while still meeting the functional requirement of predicting cart abandonment at the request level.  

**Limits:** Loses the ability to compare across multiple sessions (eg. can't detect patterns like user X always abandons cart if shipping is > $20).

**Next question:** Do business stakeholders see value in tracking repeat abandoners that outweighs the privacy risk?

---

### 3. Insight: Interpretability beats accuracy for adoption
**Evidence:** Logistic regression was selected over more complex models (decision trees, random forests) because its coefficients directly map to feature weights. This makes it possible to explain to stakeholders why a given cart was predicted as abandoned. By contrast, tree ensembles showed marginally higher offline accuracy but produced explanations that were harder to communicate.  

**Why it matters:** Marketing and retention teams need trust in the predictions to act on them (e.g., deciding whether to send a discount code). If the system feels like a black box, adoption would suffer, even if accuracy is higher. Interpretability, therefore, was a decisive design driver.  

**Limits:** Logistic regression may miss nonlinear interactions between features, placing a ceiling on predictive performance. In some domains, complex models like random forests could outperform by a wide margin, making interpretability a costly tradeoff.  

**Next question:** At what performance gap (e.g., 5%, 10%, 20% accuracy improvement) would it be justified to trade interpretability for a more complex but less transparent model?  

---

## Attachments
- Evidence pack:  
  - Baymard abandonment reasons table (Insight 1).  
  ![Baymard abandonment reasons table image](/CartSense//docs/AbandonmentReasons.png) 

  - [Project Spec – Features](./ProjectSpec.md#3-features-no-leakage) → supports Insight 1.  
  - [PIA – Data Inventory](./pia.md#2-data-inventory) → supports Insight 2.  
  - [Telemetry Decision Matrix](./TelemetryDecisionMatrix.md) → supports Insight 2.  
  - [Project Spec – Model Plan](./ProjectSpec.md#4-baseline-to-model-plan) → supports Insight 3.