## Tools & Workflows (n8n)

This section defines the workflows available to AI agents.  
Each entry includes: endpoint, inputs, outputs, guardrails, and escalation rules.  
All workflows are implemented in n8n and exposed as webhook endpoints.

---

### finance:daily_digest
- **Endpoint**: `POST https://n8n.yourdomain.com/webhook/finance-daily-digest`
- **Purpose**: Collect Shopify sales, Meta Ads spend, and compute profit metrics.
- **Input JSON**:
  ```json
  {
    "date": "YYYY-MM-DD",
    "report_channels": ["slack://growth-daily"]
  }
  ```
- **Output JSON**:
  ```json
  {
    "spend": 450.00,
    "revenue": 975.00,
    "roas": 2.16,
    "breakeven_roas": 1.43,
    "profit": 300.00,
    "winners": [{"campaign":"A","roas":3.1,"profit":180}],
    "losers": [{"campaign":"C","roas":0.9,"loss":-120}]
  }
  ```
- **Guardrails**:
  - Read-only: no direct budget changes.
  - If `roas < breakeven_roas` for 3 consecutive days → escalate to CEO Agent.
  - If `loss > $500` in one day → notify CEO Agent immediately.

---

### ads:creative_health
- **Endpoint**: `POST https://n8n.yourdomain.com/webhook/ads-creative-health`
- **Purpose**: Monitor ad fatigue and performance drop-offs.
- **Input JSON**:
  ```json
  {
    "lookback_days": 7,
    "threshold_ctr_drop": 0.3
  }
  ```
- **Output JSON**:
  ```json
  {
    "creatives_checked": 12,
    "fatigued_creatives": [
      {"id":"AD123","ctr_drop":0.45,"recommendation":"replace"},
      {"id":"AD456","ctr_drop":0.32,"recommendation":"test variant"}
    ]
  }
  ```
- **Guardrails**:
  - Read-only: outputs recommendations only.
  - Requires human approval before replacing creatives.

---

### ads:push_variants
- **Endpoint**: `POST https://n8n.yourdomain.com/webhook/ads-push-variants`
- **Purpose**: Submit new ad copy/creative variants to Meta Ads.
- **Input JSON**:
  ```json
  {
    "product":"SwiftMop",
    "variants":[
      {"headline":"Shine Your Car in Minutes","primary_text":"LazyToolz SwiftMop cleans better & faster.","cta":"Shop Now"}
    ],
    "dry_run": true
  }
  ```
- **Output JSON**:
  ```json
  {
    "status":"QUEUED",
    "review_url":"https://business.facebook.com/adsmanager/preview/12345"
  }
  ```
- **Guardrails**:
  - Must run first with `"dry_run": true`.
  - Human approval required before `"dry_run": false` deployment.
  - Cannot exceed $X/day per new campaign without CEO approval.

---

### inventory:alerts
- **Endpoint**: `POST https://n8n.yourdomain.com/webhook/inventory-alerts`
- **Purpose**: Track stock levels and forecast out-of-stock risks.
- **Input JSON**:
  ```json
  {
    "threshold_days": 14
  }
  ```
- **Output JSON**:
  ```json
  {
    "low_stock_products": [
      {"sku":"SWIFTMOP-001","days_left":10,"recommendation":"Reorder 500 units"},
      {"sku":"COBLITE-007","days_left":7,"recommendation":"Reorder 300 units"}
    ]
  }
  ```
- **Guardrails**:
  - Notify COO Agent if inventory < 14 days.
  - Escalate if inventory < 7 days.

---

### customers:sentiment_scan
- **Endpoint**: `POST https://n8n.yourdomain.com/webhook/customers-sentiment-scan`
- **Purpose**: Scan reviews and support emails for customer sentiment.
- **Input JSON**:
  ```json
  {
    "lookback_days": 3
  }
  ```
- **Output JSON**:
  ```json
  {
    "reviews_scanned": 25,
    "negative_cases": [
      {"id":"REV-203","sentiment":"negative","text":"Mop broke after 2 uses","recommendation":"refund + apology email"}
    ],
    "positive_cases": [
      {"id":"REV-211","sentiment":"positive","text":"SwiftMop works great on my SUV"}
    ]
  }
  ```
- **Guardrails**:
  - Flag only; no auto-response without human approval.
  - Escalate to Customer Success Agent if >10% negative in last 3 days.

---

### email:cart_recovery
- **Endpoint**: `POST https://n8n.yourdomain.com/webhook/email-cart-recovery`
- **Purpose**: Trigger Klaviyo abandoned cart sequence with dynamic discount ladder.
- **Input JSON**:
  ```json
  {
    "customer_id":"CUST-8899",
    "sequence":"abandoned_cart"
  }
  ```
- **Output JSON**:
  ```json
  {
    "status":"SENT",
    "discount":"10%",
    "next_step":"if no purchase → send 12% in 24h"
  }
  ```
- **Guardrails**:
  - Discounts capped at 15%.
  - Cannot trigger sequence more than once per 30 days per customer.
