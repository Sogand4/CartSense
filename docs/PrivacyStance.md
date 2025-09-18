# Privacy Stance  

This file documents where we actively resisted adding surveillance capabilities to the system.  

**Where we pushed back:**  
- Refused to collect SKUs or session IDs, even though they would increase model accuracy.  
- Refused to store IPs/User Agents from logs, avoiding device/browser fingerprinting.  
- Reduced retailer configs (shipping/payment methods) to counts/booleans instead of full lists.  

**Alternative designs considered:**  
- Hashing session IDs with salt (kept some utility, but still allowed profiling).  
- Keeping full shipping/payment lists (more detail, but unnecessary for prediction).  
- Storing indefinite logs (easier debugging, but surveillance archive risk).  

**Compromises we made:**  
- Retained prediction probabilities for ≤14 days to validate model calibration.  
- Chose serverless logging defaults, but with TTL restrictions, to balance cost and privacy.  

**Who benefits from each decision:**  
- **Retailers:** Still get actionable predictions and aggregated metrics.  
- **Customers:** Protected from cross-session tracking or advertising repurposing.  
- **Developers:** Still retain minimal telemetry for debugging and model evaluation.  
