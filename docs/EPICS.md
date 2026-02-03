# FlowPay Epics & User Stories

> Product Requirements Document companion - Detailed user stories and acceptance criteria

---

## Epic 1: Account Verification

**Description**: Enable users to link and verify bank accounts securely with real-time confirmation.

**Priority**: P0 (Foundation - Q1 2026)

---

### Story 1.1: Link Bank Account

**As a** merchant
**I want to** link my customer's bank account
**So that** I can process payments without storing sensitive credentials

**Acceptance Criteria:**

- [ ] POST `/v1/accounts` creates a new account link
- [ ] Request includes `account_number`, `routing_number`, `account_holder_name`
- [ ] Response returns `account_id` and `status: "pending_verification"`
- [ ] Account details are encrypted in Cloudflare KV
- [ ] Routing number is validated against FedACH directory
- [ ] Returns 400 for invalid routing numbers
- [ ] Returns 400 for malformed account numbers

**API Contract:**

```typescript
// Request
POST /v1/accounts
{
  "account_number": "string",     // Will be encrypted
  "routing_number": "string",     // 9 digits
  "account_holder_name": "string",
  "account_type": "checking" | "savings",
  "metadata"?: object
}

// Response 201
{
  "id": "acct_1234567890",
  "object": "account",
  "status": "pending_verification",
  "bank_name": "Example Bank",
  "account_type": "checking",
  "last4": "6789",
  "created_at": "2026-02-03T10:00:00Z"
}
```

---

### Story 1.2: Instant Account Verification

**As a** merchant
**I want to** verify account ownership instantly via Plaid
**So that** I can process payments confidently and reduce returns

**Acceptance Criteria:**

- [ ] POST `/v1/accounts/:id/verify` initiates Plaid verification
- [ ] Supports Plaid Link token exchange flow
- [ ] Returns `status: "verified"` on successful match
- [ ] Stores Plaid `item_id` and `access_token` securely
- [ ] Returns 400 if account already verified
- [ ] Returns 404 for non-existent account
- [ ] Returns 422 if Plaid verification fails

**API Contract:**

```typescript
// Request
POST /v1/accounts/:id/verify
{
  "method": "plaid",
  "public_token": "string",
  "account_id": "string"
}

// Response 200
{
  "id": "acct_1234567890",
  "object": "account",
  "status": "verified",
  "verification_method": "plaid",
  "verification_at": "2026-02-03T10:05:00Z",
  "bank_name": "Example Bank",
  "last4": "6789"
}
```

---

### Story 1.3: Account Balance Check

**As a** merchant
**I want to** check account balance before debiting
**So that** I can avoid NSF returns and failed payments

**Acceptance Criteria:**

- [ ] GET `/v1/accounts/:id/balance` returns real-time balance via Plaid
- [ ] Returns `available_balance` and `current_balance`
- [ ] Returns 403 if account not verified
- [ ] Returns 404 for non-existent account
- [ ] Returns 503 if Plaid balance fetch fails
- [ ] Caches balance for 5 minutes to reduce API calls

**API Contract:**

```typescript
// Request
GET /v1/accounts/:id/balance

// Response 200
{
  "account_id": "acct_1234567890",
  "available_balance": {
    "amount": 500050,
    "currency": "USD"
  },
  "current_balance": {
    "amount": 500100,
    "currency": "USD"
  },
  "last_updated": "2026-02-03T10:10:00Z"
}
```

---

### Story 1.4: Micro-Deposit Verification (Fallback)

**As a** merchant
**I want to** verify accounts via micro-deposits when Plaid is unavailable
**So that** I can still serve customers at unsupported banks

**Acceptance Criteria:**

- [ ] POST `/v1/accounts/:id/verify` with `method: "micro_deposits"` initiates flow
- [ ] System sends two random deposits (< $1.00 each)
- [ ] Deposits arrive within 1-3 business days
- [ ] POST `/v1/accounts/:id/confirm` with amounts completes verification
- [ ] User has 3 attempts to enter correct amounts
- [ ] Account locks after 3 failed attempts
- [ ] Micro-deposits are automatically recalled after verification

**API Contract:**

```typescript
// Initiate
POST /v1/accounts/:id/verify
{
  "method": "micro_deposits"
}

// Response 202
{
  "id": "acct_1234567890",
  "status": "awaiting_micro_deposits",
  "estimated_arrival": "2026-02-06"
}

// Confirm
POST /v1/accounts/:id/confirm
{
  "amount1": 23,
  "amount2": 67
}

// Response 200
{
  "id": "acct_1234567890",
  "status": "verified",
  "verification_method": "micro_deposits"
}
```

---

## Epic 2: ACH Transfers

**Description**: Process standard ACH transfers for low-cost, batch-friendly payments.

**Priority**: P0 (Foundation - Q1 2026)

---

### Story 2.1: Create ACH Transfer

**As a** merchant
**I want to** initiate an ACH transfer
**So that** I can move money between bank accounts

**Acceptance Criteria:**

- [ ] POST `/v1/transfers` creates a new transfer
- [ ] Requires `source_account_id` (verified) and `destination_account`
- [ ] Supports `credit` (push) and `debit` (pull) directions
- [ ] Defaults to next-day ACH
- [ ] Returns 400 if source account not verified
- [ ] Returns 422 for insufficient balance (debit)
- [ ] Creates double-entry ledger record
- [ ] Emits `transfer.created` webhook

**API Contract:**

```typescript
// Request
POST /v1/transfers
{
  "source_account_id": "acct_1234567890",
  "destination": {
    "account_number": "string",
    "routing_number": "string",
    "account_holder_name": "string",
    "account_type": "checking" | "savings"
  },
  "amount": {
    "value": 10000,  // cents
    "currency": "USD"
  },
  "direction": "credit" | "debit",
  "speed": "next_day" | "same_day",
  "description"?: string,
  "metadata"?: object
}

// Response 201
{
  "id": "trf_abc123",
  "object": "transfer",
  "status": "pending",
  "amount": {
    "value": 10000,
    "currency": "USD"
  },
  "fee": {
    "value": 10,
    "currency": "USD"
  },
  "rail": "ach",
  "speed": "next_day",
  "estimated_arrival": "2026-02-05",
  "created_at": "2026-02-03T12:00:00Z"
}
```

---

### Story 2.2: Same-Day ACH

**As a** merchant
**I want to** expedite ACH transfers for same-day arrival
**So that** I can meet urgent payment deadlines

**Acceptance Criteria:**

- [ ] `speed: "same_day"` flag enables same-day ACH
- [ ] Cutoff time is 2:00 PM CT for same-day processing
- [ ] Requests after cutoff default to next-day
- [ ] Higher fee ($0.15-$0.25) applies
- [ ] Returns warning if same-day unavailable (weekends, holidays)
- [ ] Supports ACH format with `same_day` service class code

---

### Story 2.3: Transfer Status Tracking

**As a** merchant
**I want to** check transfer status
**So that** I can provide updates to my customers

**Acceptance Criteria:**

- [ ] GET `/v1/transfers/:id` returns current status
- [ ] Status transitions: `pending` → `processing` → `completed` | `failed`
- [ ] Includes `rail` and `trace_number` when available
- [ ] Returns 404 for non-existent transfer
- [ ] Includes `error` details for failed transfers

**Status Flow:**

| Status | Description |
|--------|-------------|
| `pending` | Created, awaiting submission |
| `processing` | Submitted to ACH network |
| `completed` | Funds delivered |
| `failed` | Rejected by network or bank |
| `returned` | ACH return (NSF, etc.) |

**API Contract:**

```typescript
// Response 200
{
  "id": "trf_abc123",
  "object": "transfer",
  "status": "completed",
  "amount": { "value": 10000, "currency": "USD" },
  "rail": "ach",
  "trace_number": "0313000123456789",
  "settled_at": "2026-02-04T08:30:00Z",
  "created_at": "2026-02-03T12:00:00Z"
}
```

---

### Story 2.4: Transfer Webhooks

**As a** merchant
**I want to** receive real-time status updates
**So that** I can react to payment events without polling

**Acceptance Criteria:**

- [ ] Webhook URL configurable via dashboard/API
- [ ] Emits `transfer.pending` on submission to rail
- [ ] Emits `transfer.completed` on successful settlement
- [ ] Emits `transfer.failed` on failures
- [ ] Emits `transfer.returned` on ACH returns
- [ ] Webhooks include retry logic (exponential backoff)
- [ ] Signature verification via HMAC header
- [ ] Test webhook endpoint for verification

**Webhook Payload:**

```typescript
// Header: X-FlowPay-Signature: t=123,v1=abc123...
POST {
  "id": "evt_abc123",
  "object": "event",
  "type": "transfer.completed",
  "data": {
    "object": {
      "id": "trf_abc123",
      "object": "transfer",
      "status": "completed",
      // ... full transfer object
    }
  },
  "created_at": "2026-02-04T08:30:00Z"
}
```

---

## Epic 3: Instant Rails (FedNow/RTP)

**Description**: Enable real-time payment processing via FedNow and RTP networks.

**Priority**: P1 (Instant Rails - Q2 2026)

---

### Story 3.1: FedNow Integration

**As a** merchant
**I want to** send instant payments via FedNow
**So that** funds arrive in under 10 seconds

**Acceptance Criteria:**

- [ ] `rail: "fednow"` option in transfer creation
- [ ] Real-time credit push to recipient account
- [ ] Returns `status: "completed"` within 10 seconds
- [ ] Supports $500K per transfer limit
- [ ] Requires recipient bank FedNow participation check
- [ ] Falls back to ACH if FedNow unavailable
- [ ] 24/7/365 availability (no holidays)

**API Contract:**

```typescript
// Request
POST /v1/transfers
{
  "source_account_id": "acct_1234567890",
  "destination": { /* ... */ },
  "amount": { "value": 50000, "currency": "USD" },
  "rail": "fednow",  // Instant rail
  "speed": "instant"
}

// Response 201
{
  "id": "trf_fednow123",
  "status": "completed",
  "rail": "fednow",
  "settled_at": "2026-02-03T12:00:05Z",  // 5 seconds!
  "instant": true
}
```

---

### Story 3.2: RTP Network Integration

**As a** merchant
**I want to** send high-value instant payments via RTP
**So that** I can transfer up to $1M instantly

**Acceptance Criteria:**

- [ ] `rail: "rtp"` option in transfer creation
- [ ] Supports up to $1M per transfer
- [ ] Settlement within 15 seconds
- [ ] Requires RTP participant verification
- [ ] Includes sender/receiver identification
- [ ] Supports memo field (200 chars)

---

### Story 3.3: Bank Participation Check

**As a** merchant
**I want to** know if a recipient bank supports instant rails
**So that** I can set accurate expectations

**Acceptance Criteria:**

- [ ] GET `/v1/rails/participation?routing_number=:rn` returns capabilities
- [ ] Returns FedNow and RTP participation status
- [ ] Includes credit push support flag
- [ ] Includes request-for-pay support flag
- [ ] Caches results for 24 hours

**API Contract:**

```typescript
// Request
GET /v1/rails/participation?routing_number=021000021

// Response 200
{
  "routing_number": "021000021",
  "bank_name": "JPMORGAN CHASE BANK",
  "fednow": {
    "participating": true,
    "credit_push": true,
    "request_for_pay": true
  },
  "rtp": {
    "participating": true,
    "credit_push": true,
    "request_for_pay": false
  }
}
```

---

## Epic 4: Smart Rail Selection

**Description**: AI-powered routing selects optimal payment rail automatically.

**Priority**: P2 (Intelligence - Q3 2026)

---

### Story 4.1: Intelligent Rail Router

**As a** merchant
**I want to** let FlowPay choose the best rail
**So that** I don't need to manage routing logic

**Acceptance Criteria:**

- [ ] `rail: "auto"` option in transfer creation
- [ ] Considers urgency, amount, cost, and bank support
- [ ] Prefers instant rails when `urgency: "high"`
- [ ] Prefers RTP for amounts >$500K
- [ ] Falls back to ACH when instant unavailable
- [ ] Returns selected rail in response
- [ ] Logs routing decision for audit

**API Contract:**

```typescript
// Request
POST /v1/transfers
{
  "amount": { "value": 750000, "currency": "USD" },
  "rail": "auto",
  "urgency": "high",
  // ...
}

// Response
{
  "id": "trf_auto123",
  "rail": "rtp",  // Selected by router
  "rail_selection_reason": "high_value_and_urgency",
  "estimated_arrival": "2026-02-03T12:00:15Z"
}
```

---

### Story 4.2: Cost Optimization

**As a** merchant
**I want to** minimize payment costs
**So that** I can improve my margins

**Acceptance Criteria:**

- [ ] `urgency: "standard"` prefers lowest-cost rail
- [ ] Uses next-day ACH when time permits
- [ ] Respects merchant's `cost_preference` setting
- [ ] Shows cost comparison in dashboard
- [ ] Alerts when instant selected due to deadline

---

## Epic 5: Request-to-Pay

**Description**: Send payment requests that customers approve via their banking app.

**Priority**: P3 (Advanced - Q4 2026)

---

### Story 5.1: Create Payment Request

**As a** merchant
**I want to** send a request-for-pay
**So that** my customer can approve payment from their bank app

**Acceptance Criteria:**

- [ ] POST `/v1/requests` creates a request-to-pay
- [ ] Includes invoice details (amount, due date, description)
- [ ] Generates approval link for customer
- [ ] Sends notification (email/SMS) if requested
- [ ] Supports QR code for in-person requests
- [ ] Expires after 30 days

**API Contract:**

```typescript
// Request
POST /v1/requests
{
  "amount": { "value": 50000, "currency": "USD" },
  "recipient": {
    "email": "customer@example.com",
    "phone": "+1xxx"
  },
  "due_date": "2026-02-17",
  "description": "Invoice #1234",
  "invoice_id": "inv_1234",
  "metadata"?: object
}

// Response 201
{
  "id": "req_rtp123",
  "object": "request_to_pay",
  "status": "pending",
  "amount": { "value": 50000, "currency": "USD" },
  "approval_url": "https://pay.flowpay.io/req/abc123",
  "qr_code": "data:image/svg+xml;base64,...",
  "expires_at": "2026-03-05T12:00:00Z"
}
```

---

### Story 5.2: Request Status & Webhooks

**As a** merchant
**I want to** know when a request is approved
**So that** I can update my records

**Acceptance Criteria:**

- [ ] GET `/v1/requests/:id` returns current status
- [ ] Status: `pending` → `approved` | `declined` | `expired`
- [ ] Emits `request.approved` webhook on approval
- [ ] Emits `request.declined` webhook on rejection
- [ ] Includes transfer_id when payment completes
- [ ] Supports canceling pending requests

---

## Epic 6: Batch Payouts

**Description**: Process multiple transfers in a single API call.

**Priority**: P3 (Advanced - Q4 2026)

---

### Story 6.1: Batch Payout Creation

**As a** marketplace operator
**I want to** pay multiple sellers at once
**So that** I can efficiently process payouts

**Acceptance Criteria:**

- [ ] POST `/v1/payouts` creates a batch payout
- [ ] Supports up to 10,000 items per batch
- [ ] Each item can have different amount/destination
- [ ] Partial processing supported (failures don't block all)
- [ ] Returns batch_id and individual item statuses
- [ ] Webhook emitted when batch completes

**API Contract:**

```typescript
// Request
POST /v1/payouts
{
  "items": [
    {
      "id": "payout_1",
      "destination": { /* account */ },
      "amount": { "value": 10000, "currency": "USD" }
    },
    {
      "id": "payout_2",
      "destination": { /* account */ },
      "amount": { "value": 25000, "currency": "USD" }
    }
  ],
  "rail": "auto"
}

// Response 201
{
  "id": "batch_abc123",
  "object": "payout_batch",
  "status": "processing",
  "total_items": 2,
  "total_amount": { "value": 35000, "currency": "USD" },
  "items": [
    { "id": "payout_1", "status": "processing", "transfer_id": "trf_xxx" },
    { "id": "payout_2", "status": "processing", "transfer_id": "trf_yyy" }
  ]
}
```

---

## Definition of Ready

A story is ready to be worked on when:

- [ ] Acceptance criteria are clearly defined
- [ ] API contract is specified
- [ ] Dependencies are identified and resolved
- [ ] Edge cases are considered
- [ ] Security implications are reviewed

---

## Definition of Done

A story is complete when:

- [ ] All acceptance criteria pass
- [ ] Unit tests written and passing (>80% coverage)
- [ ] Integration tests pass
- [ ] API documentation updated
- [ ] Code reviewed and approved
- [ ] Deployed to staging
- [ ] Security review completed (if applicable)

---

*End of Document*
