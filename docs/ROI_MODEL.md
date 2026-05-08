# IceAI Platform — ROI Model & Business Case

## Assumptions (Based on Published IceCap Metrics)

| Assumption | Value | Source |
|-----------|-------|--------|
| Monthly deal volume | 75 deals/month | PrivateLenderLink profile |
| Total funded 2025 | $1.1 billion | IceCap website |
| Avg loan size | ~$1.23M ($1.1B / 900 deals) | Calculated |
| Average AE prep time / file | 25 minutes | Industry standard |
| Underwriter review time | 1–2 business days | IceCap FAQ |
| Broker routine query rate | ~40% of AE time | Estimate |
| Avg deal origination fee | ~1.5% of loan | Industry standard |

---

## Module-by-Module ROI

### DocAI

| Metric | Before | After | Delta |
|--------|--------|-------|-------|
| AE prep time / file | 25 min | 5 min | -20 min (-80%) |
| Data entry error rate | 3–5% | <0.5% | -90% |
| Monthly AE hours saved | 0 | 25 hrs (75×20min) | +25 AE-hrs/mo |
| FTE equivalent | 0 | ~2.5 FTE | Capacity gain |

**Annual value:** 300 AE-hours/year saved × $75/hr loaded cost = **$22,500/year** in direct labor.
Plus error-correction avoidance: 3% of 75 deals/month = 2.25 rerework cycles × $500/incident = **$1,350/month** = **$16,200/year**.

### UnderwriteAI

| Metric | Before | After | Delta |
|--------|--------|-------|-------|
| Underwriter review time | 1–2 days | <4 hours | -75% |
| Deals reviewable/week (1 UW) | 3–4 | 10–12 | +3x capacity |
| CTC speed | 24 hrs (heroic) | 24 hrs (routine) | Consistency |

**Strategic value:** Removes the underwriter bottleneck as deal volume scales to 100+ deals/month.

### BrokerAI

| Metric | Before | After | Delta |
|--------|--------|-------|-------|
| AE time on routine queries | 40% | 15% | -25% AE time |
| Broker wait time | Hours/business day | Instant | Near-zero |
| After-hours deal questions | Unanswered | Instant | New capability |

**Annual value:** 1 AE × 2,080 hrs/year × 25% time recovered × $75/hr = **$39,000/year** per AE freed.
**Broker retention:** Faster responses → stickier broker relationships → estimated 10–15% increase in repeat broker deal flow.

### PriceAI

| Metric | Before | After | Delta |
|--------|--------|-------|-------|
| Rate sheet type | Static monthly | Real-time dynamic | — |
| Spread improvement | Baseline | +15–25bps | +$1,845–$3,075/deal |
| Annual revenue impact | Baseline | +$1.65M–$2.77M | At $1.1B volume |

**Calculation:** $1.1B × 0.0015 (15bps) = **$1.65M/year** conservative; $1.1B × 0.0025 (25bps) = **$2.75M/year** optimistic.

---

## Combined ROI Summary

| Source | Annual Value | Type |
|--------|-------------|------|
| DocAI: labor savings | $22,500 | Direct cost |
| DocAI: error avoidance | $16,200 | Risk reduction |
| UnderwriteAI: capacity | $150,000+ | Scale enablement |
| BrokerAI: AE time freed | $39,000+ | Productivity |
| BrokerAI: broker retention | $300,000+ | Revenue (est.) |
| PriceAI: spread improvement | $1,650,000–$2,750,000 | Revenue |
| Capacity lift (75→100 deals) | $3,000,000–$5,000,000 | Revenue |
| **TOTAL (conservative)** | **~$5.2M/year** | |
| **TOTAL (optimistic)** | **~$8.3M/year** | |

---

## Implementation Cost Estimate

| Item | Monthly | Annual |
|------|---------|--------|
| AI Engineer (1 FTE) | $15,000–$20,000 | $180,000–$240,000 |
| Claude API | $3,000–$6,000 | $36,000–$72,000 |
| GPT-4 Vision API | $1,000–$3,000 | $12,000–$36,000 |
| Pinecone / pgvector | $500–$1,500 | $6,000–$18,000 |
| AWS infrastructure | $2,000–$5,000 | $24,000–$60,000 |
| **Total** | **~$22,000–$36,000** | **~$258,000–$426,000** |

---

## ROI Calculation

```
Conservative:
  Revenue + savings: $5,200,000/year
  Implementation cost: $426,000/year (high end)
  Net ROI: $4,774,000/year
  ROI multiple: 12.2x

Optimistic:
  Revenue + savings: $8,300,000/year
  Implementation cost: $258,000/year (low end)
  Net ROI: $8,042,000/year
  ROI multiple: 32.1x

Payback period: 45–90 days after full deployment
```

---

## Key Risk Factors

| Risk | Likelihood | Mitigation |
|------|-----------|-----------|
| AI extraction accuracy below target | Low | Human review gate; confidence scoring |
| Salesforce API rate limits | Medium | Request batching; caching layer |
| Broker resistance to chatbot | Low | AE escalation always available |
| ML model data requirements | Medium | Start with rule-based pricing; ML trains over time |
| Regulatory compliance issue | Low | Legal review before launch; AI for processing only |
