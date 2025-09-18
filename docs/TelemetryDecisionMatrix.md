## Telemetry Decision Matrix


| Telemetry Item                                                   | Value   | Invasiveness | Effort | Decision        |
|------------------------------------------------------------------|---------|--------------|--------|-----------------|
| Request count per retailer (number of predictions)               | High¹   | Low          | Low    | Keep            |
| p95 latency                                                      | High    | Low          | Low    | Keep            |
| Error rate (failed predictions due to bad input, timeouts, etc.) | High²   | Low          | Low    | Keep            |
| Full cart contents (SKUs)                                        | Medium  | High         | Medium | Drop            |
| Aggregate abandonment rate                                       | High    | Low³         | Low    | Keep (≤90d)     |
| Prediction probability logs (for calibration checks)             | Medium⁴ | Medium⁵      | Low    | Keep (≤14d raw) |
| Raw cart/session IDs                                             | Low     | High         | Low    | Drop            |
| System load metrics (CPU/memory)                                 | Low     | Medium       | Medium | Drop            |

<sup>1</sup> Helps detect viral spikes and manage scaling costs.  
<sup>2</sup> Shows system reliability.   
<sup>3</sup> Data is aggregated and anonymized across many carts, no single user can be traced.     
<sup>4</sup> Logged briefly to verify if predictions matched actual outcomes, support calibration, and allow retailers to adjust their own thresholds (e.g., trigger reminder if p ≥ 0.7).  
<sup>5</sup> Rare cart patterns + retailer_id could reveal user behavior.  