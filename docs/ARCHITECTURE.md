# IceAI Platform — System Architecture

## Overview

The IceAI Platform is a four-module microservices system designed to integrate with IceCap Group's existing Salesforce-based infrastructure. Each module is independently deployable and communicates via REST APIs.

## Core Design Principles

1. **No rip-and-replace** — integrates with existing Salesforce LOS
2. **Human-in-the-loop** — AI assists; humans decide on every loan
3. **Confidence-first** — every AI output includes a confidence score and source citation
4. **Audit trail** — all AI actions logged for compliance review

---

## Module Architecture

### DocAI Service

```
Input: Raw documents (PDF, JPEG, PNG)
       via Salesforce portal file upload

Processing:
  1. File ingestion → S3/GCS blob storage
  2. Document type classification (bank stmt, appraisal, contract...)
  3. OCR → GPT-4 Vision API
  4. Field extraction → Claude API (structured JSON)
  5. Validation → business rules engine
  6. Confidence scoring → flag low-confidence fields

Output: Structured JSON → Salesforce REST API
        → Auto-populates 20+ deal fields
        → Generates exception checklist for AE review

Latency target: < 5 minutes end-to-end
Accuracy target: > 99.5% on high-confidence fields
```

### UnderwriteAI Service

```
Input: DocAI-extracted deal data (Salesforce fields)
       + Property comps (external API)
       + Borrower history (internal DB)

Processing:
  1. Retrieve structured deal data from Salesforce
  2. Fetch property market context (Zillow/ATTOM API)
  3. Load borrower track record from deal history DB
  4. Assemble context → Claude API with underwriting prompt
  5. Generate memo: summary, risk flags, recommendation
  6. Format → PDF / Salesforce note

Output: First-draft underwriting memo
        → Underwriter review queue
        → Editable in Salesforce

Latency target: < 8 minutes
Quality gate: human review required before any decision
```

### BrokerAI Service

```
Input: Natural language query from broker via chat widget
       + Broker ID (for deal history context)

Processing:
  1. Query classification (eligibility / rate / status / guideline)
  2. RAG retrieval from:
     - Loan guidelines vector store (Pinecone)
     - Rate sheet embeddings
     - Deal history DB (broker-scoped)
  3. Claude API with retrieved context + query
  4. Response generation + source citation
  5. Escalation trigger: complex queries → AE notification

Output: Instant chat response
        → Escalation ticket if needed
        → AE notification for flagged queries

Vector store: Pinecone (guidelines) + pgvector (deal history)
Re-indexing: Automated on guideline/rate sheet updates
```

### PriceAI Service

```
Input: Deal parameters:
       - Property type, state/zip
       - LTV ratio
       - Loan amount & term
       - Borrower FICO + track record
       - Current market conditions (rate index)

Processing:
  1. Feature engineering from deal parameters
  2. Market condition lookup (Fed rate feed, SOFR)
  3. ML model inference (XGBoost + historical deal data)
  4. Competitor rate check (web scraping / manual input)
  5. Spread optimization → recommended rate

Output: Recommended rate + confidence band
        → AE rate sheet
        → Broker quote in BrokerAI

Model training: Monthly retraining on new deal outcomes
Feature store: Feast (offline + online features)
```

---

## Data Flow Diagram

```
Broker uploads docs
        |
        v
[DocAI: Vision AI + LLM]
        |
        v
Salesforce fields auto-populated
        |
        +----------> [BrokerAI: answers broker questions throughout]
        |
        v
[UnderwriteAI: generates memo]
        |
        v
Underwriter reviews memo (< 4 hours)
        |
        v
[PriceAI: computes real-time rate]
        |
        v
AE presents rate to broker
        |
        v
Clear-to-Close issued
```

---

## Infrastructure

### Cloud Architecture (AWS)

```
Route 53 (DNS)
    |
CloudFront (CDN)
    |
ALB (Application Load Balancer)
    |
    +-- ECS/Fargate Cluster
    |       |
    |       +-- docai-service (GPU-enabled)
    |       +-- underwrite-service
    |       +-- brokerai-service
    |       +-- priceai-service
    |
    +-- RDS PostgreSQL (primary data store)
    +-- ElastiCache Redis (session + caching)
    +-- S3 (document storage)
    +-- SQS (async job queue)
    +-- Pinecone (vector search)
```

### CI/CD Pipeline

```
GitHub Push
    |
GitHub Actions
    |
    +-- Lint + Unit Tests
    +-- Integration Tests (Salesforce sandbox)
    +-- Docker build + push to ECR
    +-- Deploy to staging
    +-- Smoke tests
    +-- Deploy to production (manual gate)
```

---

## Security & Compliance

| Area | Approach |
|------|----------|
| Data encryption | AES-256 at rest, TLS 1.3 in transit |
| PII handling | SOC 2 Type II compliant; no PII in model training |
| AI provider data | Zero-retention API calls (no training on IceCap data) |
| Audit logging | All AI actions logged with actor, timestamp, confidence |
| ECOA compliance | AI outputs reviewed for disparate impact quarterly |
| FCRA | AI used for processing only, not credit decisioning |
| Rollback | All Salesforce writes have 24-hour rollback capability |

---

## API Reference (Internal)

### DocAI API

```
POST /api/v1/doc/process
  Body: { deal_id, file_url, document_type }
  Returns: { fields: {...}, confidence: {...}, flags: [...] }

GET /api/v1/doc/status/{job_id}
  Returns: { status, progress, eta }
```

### BrokerAI API

```
POST /api/v1/broker/chat
  Body: { broker_id, message, session_id }
  Returns: { response, sources, escalate: bool }

GET /api/v1/broker/deals/{broker_id}
  Returns: { deals: [...], summary: {...} }
```

### PriceAI API

```
POST /api/v1/price/quote
  Body: { property_type, state, ltv, amount, term, fico, track_record }
  Returns: { recommended_rate, confidence_band, comp_rates }
```
