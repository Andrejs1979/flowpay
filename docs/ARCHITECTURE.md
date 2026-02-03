# FlowPay Architecture

> System design and technical specifications for FlowPay real-time payment infrastructure

---

## Overview

FlowPay is built on **Cloudflare Workers** for edge-native performance, leveraging Cloudflare's global network for low-latency payment processing worldwide.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              Cloudflare Edge                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌──────────────┐      ┌──────────────┐      ┌──────────────┐              │
│  │   Gateway    │──────│ Rail Router  │──────│  Executions  │              │
│  │   (Workers)  │      │   (Workers)  │      │  (Workers)   │              │
│  └──────────────┘      └──────────────┘      └──────────────┘              │
│         │                      │                      │                     │
│         ▼                      ▼                      ▼                     │
│  ┌──────────────┐      ┌──────────────┐      ┌──────────────┐              │
│  │ Auth / Valid │      │  ML Routing  │      │ Webhook Disp │              │
│  │   (Workers)  │      │ (Workers AI) │      │   (Queues)   │              │
│  └──────────────┘      └──────────────┘      └──────────────┘              │
│         │                                                                      │
│         ▼                                                                      │
│  ┌──────────────────────────────────────────────────────────────────────┐    │
│  │                         Data Layer                                    │    │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐   │    │
│  │  │    D1   │  │    KV   │  │  Queues │  │  R2     │  │ Secrets │   │    │
│  │  │ Ledger  │  │ Vault   │  │Events   │  │ Docs    │  │  Store  │   │    │
│  │  └─────────┘  └─────────┘  └─────────┘  └─────────┘  └─────────┘   │    │
│  └──────────────────────────────────────────────────────────────────────┘    │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
                            │                    │
                            ▼                    ▼
┌─────────────────────────┐      ┌──────────────────────────────────────────┐
│   External Integrations │      │         Payment Rails                    │
├─────────────────────────┤      ├──────────────────────────────────────────┤
│ • Plaid (Verification)  │      │ • FedNow (Federal Reserve)              │
│ • MX (Backup Verification)│    │ • RTP (The Clearing House)              │
│ • FedNow API            │      │ • ACH (Originating Bank)                │
│ • RTP Network           │      │                                          │
│ • Banking Partners      │      │                                          │
└─────────────────────────┘      └──────────────────────────────────────────┘
```

---

## Technology Stack

### Core Platform

| Component | Technology | Purpose |
|-----------|------------|---------|
| **Runtime** | Cloudflare Workers | Edge compute, low latency |
| **Language** | TypeScript | Type safety, developer experience |
| **Framework** | Hono | Lightweight, edge-optimized web framework |
| **Deployment** | Wrangler | Cloudflare CLI tool |
| **CI/CD** | GitHub Actions | Testing, deployment automation |

### Data Storage

| Component | Technology | Purpose |
|-----------|------------|---------|
| **Ledger** | Cloudflare D1 | Transaction history, double-entry accounting |
| **Vault** | Cloudflare KV | Encrypted bank account storage |
| **Queues** | Cloudflare Queues | Webhook delivery, async processing |
| **Cache** | Cloudflare KV | Rail participation, routing decisions |
| **Documents** | Cloudflare R2 | Statements, reconciliation files |

### External Services

| Service | Purpose |
|---------|---------|
| **Plaid** | Account verification, balance checks |
| **MX** | Backup verification provider |
| **FedNow** | Instant payment rail |
| **RTP Network** | Instant payment rail |
| **Originating Bank** | ACH file transmission |
| **Sentry** | Error tracking, monitoring |
| **Datadog** | Metrics, observability |

---

## Project Structure

```
flowpay/
├── src/
│   ├── index.ts                 # Entry point, Hono app setup
│   ├── config/                  # Configuration management
│   │   ├── env.ts               # Environment variable schema
│   │   ├── constants.ts         # Constants, limits, fees
│   │   └── features.ts          # Feature flags
│   ├── middleware/              # Request middleware
│   │   ├── auth.ts              # API key authentication
│   │   ├── cors.ts              # CORS handling
│   │   ├── rate-limit.ts        # Rate limiting
│   │   ├── validation.ts        # Request validation
│   │   └── error-handler.ts     # Error handling
│   ├── routes/                  # API routes
│   │   ├── v1/
│   │   │   ├── accounts.ts      # Account endpoints
│   │   │   ├── transfers.ts     # Transfer endpoints
│   │   │   ├── requests.ts      # Request-to-Pay endpoints
│   │   │   ├── payouts.ts       # Batch payout endpoints
│   │   │   ├── rails.ts         # Rail participation endpoints
│   │   │   └── webhooks.ts      # Webhook management
│   │   └── index.ts             # Route aggregator
│   ├── services/                # Business logic
│   │   ├── accounts/
│   │   │   ├── service.ts       # Account service
│   │   │   ├── verification.ts  # Verification logic
│   │   │   └── vault.ts         # Encrypted storage
│   │   ├── transfers/
│   │   │   ├── service.ts       # Transfer service
│   │   │   ├── rail-router.ts   # Rail selection
│   │   │   ├── ach.ts           # ACH processing
│   │   │   ├── fednow.ts        # FedNow processing
│   │   │   └── rtp.ts           # RTP processing
│   │   ├── webhooks/
│   │   │   ├── dispatcher.ts    # Webhook delivery
│   │   │   └── signature.ts     # Signature verification
│   │   └── ledger/
│   │       ├── service.ts       # Double-entry ledger
│   │       └── reconciliation.ts# Reconciliation logic
│   ├── integrations/            # External integrations
│   │   ├── plaid/
│   │   │   ├── client.ts        # Plaid API client
│   │   │   └── types.ts         # Plaid types
│   │   ├── fednow/
│   │   │   ├── client.ts        # FedNow API client
│   │   │   ├── iso20022.ts      # ISO 20022 parsing
│   │   │   └── signing.ts       # JWT signing
│   │   ├── rtp/
│   │   │   ├── client.ts        # RTP API client
│   │   │   └── signing.ts       # Signing logic
│   │   └── ach/
│   │       ├── generator.ts     # ACH file generation
│   │       └── parser.ts        # ACH return parsing
│   ├── models/                  # Data models
│   │   ├── account.ts           # Account model
│   │   ├── transfer.ts          # Transfer model
│   │   ├── ledger.ts            # Ledger entry model
│   │   ├── webhook.ts           # Webhook model
│   │   └── event.ts             # Event model
│   ├── db/                      # Database
│   │   ├── schema.ts            # D1 schema
│   │   ├── migrations/          # Migration files
│   │   └── seed.ts              # Seed data
│   ├── types/                   # Shared types
│   │   ├── api.ts               # API request/response types
│   │   ├── rails.ts             # Rail types
│   │   └── events.ts            # Event types
│   ├── utils/                   # Utilities
│   │   ├── crypto.ts            # Encryption, hashing
│   │   ├── validation.ts        # Validation helpers
│   │   ├── formatting.ts        # Number/date formatting
│   │   └── errors.ts            # Error classes
│   └── workers/                 # Background workers
│       ├── ach-processor.ts     # ACH batch processing
│       ├── reconciliation.ts    # Daily reconciliation
│       └── webhook-retry.ts     # Webhook retry logic
├── tests/
│   ├── unit/                   # Unit tests
│   ├── integration/            # Integration tests
│   └── e2e/                    # End-to-end tests
├── scripts/
│   ├── deploy.ts               # Deployment script
│   ├── migrate.ts              # Database migration
│   └── seed.ts                 # Seed data
├── docs/
│   ├── PRD.md                  # Product requirements
│   ├── EPICS.md                # User stories
│   ├── PROGRESS.md             # Sprint progress
│   └── ARCHITECTURE.md         # This file
├── wrangler.toml               # Cloudflare config
├── package.json
├── tsconfig.json
└── README.md
```

---

## Core Services

### Gateway Service

The Gateway is the entry point for all API requests.

**Responsibilities:**
- Request validation and parsing
- API key authentication
- Rate limiting
- Request routing to appropriate services
- Response formatting

**Key Routes:**

```typescript
// API Versioning
app.route('/v1/*', v1Router)

// Health check
app.get('/health', healthCheck)

// Webhook signature verification (before route handler)
app.use('/v1/webhooks/*', verifyWebhookSignature)
```

---

### Rail Router Service

Intelligently selects the optimal payment rail using ML and rule-based logic.

**Routing Algorithm:**

```
┌─────────────────────────────────────────────────────────────────┐
│                    Rail Router Decision Engine                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. Parse Input                                                 │
│     ├─ urgency (instant/standard)                               │
│     ├─ amount                                                   │
│     ├─ recipient routing number                                 │
│     └─ merchant preferences                                     │
│                                                                 │
│  2. Check Bank Participation                                    │
│     ├─ Query cache for rail support                            │
│     ├─ Cache miss? → Query FedNow/RTP directories              │
│     └─ Return supported rails                                   │
│                                                                 │
│  3. Apply Business Rules                                        │
│     ├─ IF urgency = instant AND fednow_supported → FedNow       │
│     ├─ IF urgency = instant AND rtp_supported → RTP            │
│     ├─ IF amount > $500K → RTP (higher limit)                   │
│     ├─ IF cost_sensitive → ACH (lowest cost)                    │
│     └─ ELSE → Next-day ACH                                      │
│                                                                 │
│  4. Validate Constraints                                        │
│     ├─ Amount within rail limits                                │
│     ├─ Rail available (hours, holidays)                         │
│     └─ Merchant enabled for rail                                │
│                                                                 │
│  5. Return Decision                                             │
│     └─ Selected rail + trace_id for audit                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

### Account Verification Service

Handles account linking and verification through multiple providers.

**Verification Flow:**

```
┌─────────────────────────────────────────────────────────────────┐
│                    Account Verification Flow                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. Link Account                                                │
│     └─ Store encrypted account details in KV                    │
│                                                                 │
│  2. Choose Verification Method                                  │
│     ├─ Plaid (preferred) ────────┐                             │
│     ├─ MX (backup)               ├─> Instant Verification       │
│     └─ Micro-deposits (fallback) │                             │
│                                │                                │
│                                ▼                                │
│                       3. Verify Ownership                       │
│                          ├─ Plaid Link Token                    │
│                          ├─ Exchange for Access Token           │
│                          ├─ Fetch Account Info                  │
│                          └─ Compare Account Numbers             │
│                                │                                │
│                                ▼                                │
│                       4. Optional Balance Check                 │
│                          └─ Fetch via Plaid Balance API         │
│                                │                                │
│                                ▼                                │
│                       5. Update Status                          │
│                          └─ Mark account as "verified"          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

### Settlement Ledger Service

Double-entry accounting system tracking all money movement.

**Schema:**

```sql
-- Accounts Table
CREATE TABLE accounts (
  id TEXT PRIMARY KEY,
  object TEXT DEFAULT 'account',
  status TEXT NOT NULL, -- pending_verification, verified, disabled
  bank_name TEXT,
  account_type TEXT, -- checking, savings
  last4 TEXT,
  verification_method TEXT,
  plaid_item_id TEXT,
  metadata TEXT, -- JSON
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Transfers Table
CREATE TABLE transfers (
  id TEXT PRIMARY KEY,
  object TEXT DEFAULT 'transfer',
  status TEXT NOT NULL, -- pending, processing, completed, failed, returned
  source_account_id TEXT,
  destination_account_id TEXT,
  amount INTEGER NOT NULL, -- cents
  currency TEXT DEFAULT 'USD',
  fee INTEGER DEFAULT 0, -- cents
  rail TEXT NOT NULL, -- ach, fednow, rtp
  speed TEXT NOT NULL, -- instant, same_day, next_day
  direction TEXT NOT NULL, -- credit, debit
  trace_number TEXT,
  error_code TEXT,
  error_message TEXT,
  metadata TEXT, -- JSON
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  settled_at DATETIME,
  FOREIGN KEY (source_account_id) REFERENCES accounts(id)
);

-- Ledger Entries (Double-Entry)
CREATE TABLE ledger_entries (
  id TEXT PRIMARY KEY,
  transfer_id TEXT NOT NULL,
  account_id TEXT NOT NULL,
  entry_type TEXT NOT NULL, -- debit, credit
  amount INTEGER NOT NULL, -- cents
  currency TEXT DEFAULT 'USD',
  balance_after INTEGER NOT NULL, -- Running balance
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (transfer_id) REFERENCES transfers(id),
  FOREIGN KEY (account_id) REFERENCES accounts(id)
);

-- Webhooks Table
CREATE TABLE webhooks (
  id TEXT PRIMARY KEY,
  url TEXT NOT NULL,
  secret TEXT NOT NULL, -- HMAC secret
  events TEXT NOT NULL, -- JSON array of event types
  active INTEGER DEFAULT 1,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Events Table (Audit Log)
CREATE TABLE events (
  id TEXT PRIMARY KEY,
  type TEXT NOT NULL,
  data TEXT NOT NULL, -- JSON
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Batches Table
CREATE TABLE payout_batches (
  id TEXT PRIMARY KEY,
  object TEXT DEFAULT 'payout_batch',
  status TEXT NOT NULL,
  total_items INTEGER NOT NULL,
  total_amount INTEGER NOT NULL,
  rail TEXT NOT NULL,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  completed_at DATETIME
);

-- Batch Items Table
CREATE TABLE payout_items (
  id TEXT PRIMARY KEY,
  batch_id TEXT NOT NULL,
  transfer_id TEXT,
  status TEXT NOT NULL,
  amount INTEGER NOT NULL,
  destination_account_id TEXT,
  error_message TEXT,
  FOREIGN KEY (batch_id) REFERENCES payout_batches(id),
  FOREIGN KEY (transfer_id) REFERENCES transfers(id)
);
```

---

### Webhook Dispatcher Service

Delivers real-time event notifications to merchants.

**Delivery Flow:**

```
┌─────────────────────────────────────────────────────────────────┐
│                     Webhook Delivery Flow                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. Event Triggered                                             │
│     └─ Transfer status changes, etc.                            │
│                                                                 │
│  2. Create Event Record                                        │
│     └─ Store in D1 events table                                │
│                                                                 │
│  3. Find Active Webhooks                                        │
│     └─ Query for matching event types                           │
│                                                                 │
│  4. Enqueue Delivery Jobs                                      │
│     └─ One job per webhook URL                                 │
│                                                                 │
│  5. Worker Processes Queue                                     │
│     ├─ Sign payload with HMAC                                   │
│     ├─ Send POST request                                       │
│     ├─ Check response (200-204 accepted)                       │
│     └─ On failure: retry with exponential backoff              │
│                                                                 │
│  6. Retry Logic                                                │
│     ├─ Attempt 1: immediate                                    │
│     ├─ Attempt 2: 1 minute delay                               │
│     ├─ Attempt 3: 5 minute delay                               │
│     ├─ Attempt 4: 30 minute delay                              │
│     └─ After 4 attempts: mark failed                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**Signature Format:**

```
X-FlowPay-Timestamp: 1707014400
X-FlowPay-Signature: t=1707014400,v1=abc123...

signature = HMAC-SHA256(secret, timestamp + '.' + payload)
```

---

## External Integrations

### Plaid Integration

**Purpose:** Account verification and balance checks

**Flow:**
1. Create Link Token: Server-side, client_id + secret
2. Exchange Token: Client-side, public_token for access_token
3. Fetch Accounts: Get account numbers via Auth
4. Verify Match: Compare account numbers
5. Balance Check: Optional real-time balance

**Configuration:**
```typescript
const plaidConfig = {
  environment: 'development' | 'sandbox' | 'production',
  clientId: env.PLAID_CLIENT_ID,
  secret: env.PLAID_SECRET,
  webhookKey: env.PLAID_WEBHOOK_KEY
}
```

---

### FedNow Integration

**Purpose:** Instant credit push via Federal Reserve network

**Protocol:** ISO 20022 message format over REST API

**Authentication:** JWT signed with mutual TLS certificate

**Key Endpoints:**
- `POST /pacs.008` - Credit Transfer message
- `GET /pacs.002` - Payment status query

**Message Format (ISO 20022):**
```xml
<Document xmlns="urn:iso:std:iso:20022:tech:xsd:pacs.008.001.08">
  <FIToFICstmrCdtTrf>
    <GrpHdr>
      <MsgId>MSG-20260203-001</MsgId>
      <CreDtTm>2026-02-03T12:00:00Z</CreDtTm>
      <NbOfTxs>1</NbOfTxs>
    </GrpHdr>
    <CdtTrfTxInf>
      <PmtId>
        <InstrId>TRF-FEDNOW-001</InstrId>
        <EndToEndId>trf_abc123</EndToEndId>
      </PmtId>
      <Amt>
        <InstdAmt Ccy="USD">100.00</InstdAmt>
      </Amt>
      <CdtrAgt>
        <FinInstnId>
          <ClrSysMmbId>
            <ClrSysId>USFN</ClrSysId>
            <MmbId>021000021</MmbId>
          </ClrSysMmbId>
        </FinInstnId>
      </CdtrAgt>
      <Cdtr>
        <Nm>Account Holder</Nm>
        <Acct>
          <Id>
            <Othr>
              <Id>123456789</Id>
            </Othr>
          </Id>
        </Acct>
      </Cdtr>
    </CdtTrfTxInf>
  </FIToFICstmrCdtTrf>
</Document>
```

---

### RTP Integration

**Purpose:** Instant credit push via The Clearing House network

**Protocol:** JSON over REST API

**Authentication:** API key + signed JWT

**Key Endpoints:**
- `POST /credit/payments` - Initiate credit payment
- `GET /credit/payments/{paymentId}` - Query payment status

---

### ACH Integration

**Purpose:** Same-day and next-day ACH transfers

**Protocol:** NACHA file format (flat file)

**Transmission:**
- Via originating banking partner SFTP
- File generation: ACH IAT/CTX format
- Return file processing: Daily SFTP polling

---

## Security Architecture

### Encryption

| Data Type | Storage | Encryption |
|-----------|---------|------------|
| Bank Account Numbers | Cloudflare KV | AES-256-GCM |
| Routing Numbers | D1 (plaintext) | - |
| Plaid Access Tokens | Cloudflare KV | AES-256-GCM |
| API Keys | D1 (hashed) | bcrypt |
| Webhook Secrets | D1 (hashed) | bcrypt |

### Key Management

- **Encryption Keys**: Cloudflare Workers Secrets (env vars)
- **Key Rotation**: Monthly rotation schedule
- **Key Escrow**: HSM-backed escrow for recovery

### Authentication

**API Keys:**
- Format: `fpk_live_...` or `fpk_test_...`
- Storage: Hashed with bcrypt in D1
- Scope: Read/Write/Admin permissions
- Rate Limits: Per-key limits

**Webhook Verification:**
- HMAC-SHA256 signature
- Timestamp replay protection (5 min window)
- Per-webhook secret

---

## Reliability & Resilience

### High Availability

| Component | Availability Strategy |
|-----------|----------------------|
| API Gateway | Cloudflare Workers (auto-scales, 99.99% SLA) |
| D1 Database | Multi-region replicas |
| KV Storage | Global replication |
| Queues | At-least-once delivery |

### Failure Handling

**Retries:**
- External API calls: 3 retries with exponential backoff
- Webhook delivery: 4 retries with increasing delays
- Idempotency keys for safe retries

**Circuit Breakers:**
- Plaid API: Open after 10 consecutive failures
- FedNow API: Open after 5 consecutive failures
- RTP API: Open after 5 consecutive failures

**Fallbacks:**
- Plaid → MX for account verification
- FedNow/RTP → ACH for instant payment fallback
- Micro-deposits for unsupported banks

---

## Monitoring & Observability

### Metrics (Datadog)

| Metric | Type | Description |
|--------|------|-------------|
| `api.requests.total` | Counter | Total API requests |
| `api.requests.errors` | Counter | Failed requests |
| `api.latency` | Histogram | Request latency |
| `transfers.created` | Counter | Transfers created |
| `transfers.completed` | Counter | Successful transfers |
| `transfers.failed` | Counter | Failed transfers |
| `webhooks.delivered` | Counter | Webhooks delivered |
| `webhooks.failed` | Counter | Webhook failures |
| `rail.selection.{rail}` | Counter | Rail selection counts |

### Logging

**Log Levels:**
- ERROR: Failed transfers, API errors
- WARN: Retry attempts, fallbacks
- INFO: Transfer state changes, webhook delivery
- DEBUG: Detailed request/response

**Log Format (JSON):**
```json
{
  "timestamp": "2026-02-03T12:00:00Z",
  "level": "INFO",
  "request_id": "req_abc123",
  "event": "transfer.completed",
  "transfer_id": "trf_xyz789",
  "rail": "fednow",
  "amount": 10000,
  "duration_ms": 5234
}
```

### Tracing

- Distributed tracing with Datadog APM
- Trace context: `x-datadog-trace-id` header
- Spans: API Gateway → Service → External API

---

## Deployment Architecture

### Environments

| Environment | Purpose | URL |
|-------------|---------|-----|
| `development` | Local development | `http://localhost:8787` |
| `staging` | Pre-production testing | `https://staging.api.flowpay.io` |
| `production` | Live traffic | `https://api.flowpay.io` |

### Deployment Pipeline

```
┌─────────────────────────────────────────────────────────────────┐
│                     CI/CD Pipeline                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. Push to Feature Branch                                      │
│     └─ Trigger: GitHub Actions                                  │
│                                                                 │
│  2. Run Tests                                                  │
│     ├─ Unit tests (Vitest)                                     │
│     ├─ Integration tests                                       │
│     └─ Type check (tsc)                                        │
│                                                                 │
│  3. Security Scan                                              │
│     └─ CodeQL, npm audit                                       │
│                                                                 │
│  4. Build                                                      │
│     └─ wrangler publish --env staging                          │
│                                                                 │
│  5. Deploy to Staging                                         │
│     └─ wrangler deploy --env staging                           │
│                                                                 │
│  6. Run Smoke Tests                                           │
│     └─ Health check, sample transfer                           │
│                                                                 │
│  7. Create PR → Review → Merge                                 │
│                                                                 │
│  8. Deploy to Production                                      │
│     └─ wrangler deploy --env production                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Performance Targets

| Metric | Target | Measurement |
|--------|--------|-------------|
| API Latency (p50) | < 100ms | Gateway response time |
| API Latency (p99) | < 500ms | Gateway response time |
| FedNow Settlement | < 10s | Transfer created to completed |
| RTP Settlement | < 15s | Transfer created to completed |
| ACH Settlement | 1-2 days | Transfer created to completed |
| Webhook Delivery | < 5s | Event to webhook sent |
| API Uptime | 99.99% | Monthly availability |

---

## Compliance

### Regulatory

- **FinCEN**: Money Services Business (MSB) registration
- **NACHA**: ACH operator rules compliance
- **FedNow**: Participation agreement
- **RTP**: Network rules compliance

### Data Privacy

- **GLBA**: Safeguards rule for financial data
- **Data Retention**: 7 years for transaction records
- **Right to Deletion**: Per customer request (where allowed)

### Audit

- Annual SOC 2 Type II examination
- Quarterly penetration testing
- Annual regulatory compliance review

---

*End of Document*
