# FLOWPAY
# Real-Time Bank Payment Infrastructure
# Product Requirements Document
# Version 1.0 | February 2026
**CONFIDENTIAL - Tap2 / CloudMind Inc.**

---

## Document Information

| Attribute | Value |
|-----------|-------|
| Document Title | FlowPay Product Requirements Document |
| Product Name | FlowPay |
| Product Category | Core Infrastructure |
| Status | Production |
| Owner | Tap2 Platform Team |
| Last Updated | February 2026 |

---

## Executive Summary

FlowPay is Tap2's bank-to-bank payment infrastructure enabling instant money movement via FedNow, RTP, and ACH rails. It provides **60% better margins** than card networks while enabling real-time settlement 24/7/365.

FlowPay serves as the bank payment engine within the CardQL unified platform, handling all non-card money movement including payouts, B2B payments, and instant transfers.

### Key Value Propositions

- **Instant settlement** via FedNow and RTP (under 10 seconds, 24/7/365)
- **60% margin improvement** over card network fees
- **Single API** for ACH, FedNow, RTP with intelligent rail selection
- **Account verification** via Plaid for reduced fraud and returns
- **Request-to-Pay** for invoice collection and bill presentment

---

## Problem Statement

### Current Market Pain Points

| Pain Point | Impact | FlowPay Solution |
|------------|--------|------------------|
| Slow ACH Settlement | 3-5 business days delays cash flow | FedNow/RTP instant settlement |
| High Card Fees | 2.9% + $0.30 erodes margins | 0.5% bank rail costs |
| Limited Hours | ACH only on business days | 24/7/365 instant payments |
| Return/NSF Risk | 8% ACH return rate industry average | Real-time balance verification |
| Complex Integration | Multiple providers for different rails | Single unified API |

### Target Users

- **B2B Platforms**: Invoicing, supply chain, vendor payments
- **Marketplaces**: Seller payouts, instant disbursements
- **Gig Economy**: Instant worker payouts
- **Insurance/Lending**: Claims disbursement, loan funding

---

## Product Architecture

### Payment Rails

| Rail | Speed | Cost | Limit | Hours | Use Case |
|------|-------|------|-------|-------|----------|
| **FedNow** | <10 sec | $0.01-0.05 | $500K | 24/7/365 | Urgent payments, gig payouts |
| **RTP** | <15 sec | $0.01-0.045 | $1M | 24/7/365 | Business payments, high-value |
| **Same-Day ACH** | Same day | $0.10-0.25 | $1M | Business days | Payroll, recurring |
| **Next-Day ACH** | 1-2 days | $0.05-0.15 | $25M | Business days | Batch, low-priority |

### System Components

| Component | Description | Technology |
|-----------|-------------|------------|
| **FlowPay Gateway** | API gateway for all bank payment requests | Cloudflare Workers |
| **Rail Router** | Intelligent selection of optimal payment rail | ML on Workers AI |
| **Account Vault** | Secure storage of bank account details | Encrypted Cloudflare KV |
| **Verification Engine** | Real-time account and balance verification | Plaid, MX integration |
| **Settlement Ledger** | Double-entry tracking of all transfers | Cloudflare D1 |
| **Webhook Dispatcher** | Real-time status notifications | Cloudflare Queues |

---

## Core Features

### 1. Instant Bank Transfers

FlowPay enables true instant money movement between any US bank accounts via FedNow and RTP networks.

- **FedNow Integration**: Direct connection to Federal Reserve instant payment system
- **RTP Network**: The Clearing House real-time payments for higher limits
- **Instant Confirmation**: Real-time webhooks for payment completion
- **24/7 Availability**: Payments process any time, including holidays

### 2. Smart Rail Selection

AI-powered routing automatically selects the optimal rail based on speed requirements, cost, and recipient bank capabilities.

| Factor | Weight | Logic |
|--------|--------|-------|
| **Urgency** | High | Instant requested → FedNow/RTP; Standard → ACH |
| **Amount** | Medium | >$500K → RTP; <$500K → FedNow preferred |
| **Bank Support** | High | Check recipient bank FedNow/RTP participation |
| **Cost Sensitivity** | Medium | Merchant cost preference influences rail choice |
| **Time of Day** | Low | After hours → Instant rails; Business hours → Any |

### 3. Account Verification

Real-time verification of bank accounts reduces fraud and ACH returns.

- **Instant Account Verification**: Plaid-powered real-time ownership confirmation
- **Balance Check**: Optional real-time balance verification before debit
- **Micro-Deposit Fallback**: Traditional verification when instant unavailable
- **Account Health Score**: Risk scoring based on account history

### 4. Request-to-Pay

Send payment requests that customers can approve and pay instantly from their bank.

- **Invoice Presentment**: Rich invoice details in payment request
- **One-Click Approval**: Customer approves via bank app notification
- **Instant Settlement**: Payment completes in seconds upon approval
- **Guaranteed Funds**: No risk of returns or NSF once approved

---

## API Specification

### Core Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/v1/transfers` | POST | Create a bank-to-bank transfer |
| `/v1/transfers/:id` | GET | Retrieve transfer status and details |
| `/v1/accounts` | POST | Link and verify a bank account |
| `/v1/accounts/:id/verify` | POST | Verify account ownership |
| `/v1/accounts/:id/balance` | GET | Check real-time account balance |
| `/v1/requests` | POST | Create a Request-to-Pay |
| `/v1/payouts` | POST | Batch payout to multiple recipients |

### Webhook Events

| Event | Trigger |
|-------|---------|
| `transfer.created` | Transfer initiated, pending settlement |
| `transfer.pending` | Transfer submitted to rail network |
| `transfer.completed` | Funds successfully delivered |
| `transfer.failed` | Transfer rejected or failed |
| `transfer.returned` | ACH transfer returned by receiving bank |
| `request.approved` | Customer approved Request-to-Pay |
| `request.declined` | Customer declined or ignored request |

---

## Pricing Model

| Transfer Type | Rate | Notes |
|---------------|------|-------|
| **FedNow Instant** | 0.5% + $0.25 | Capped at $5 per transfer |
| **RTP Instant** | 0.5% + $0.25 | Capped at $10 per transfer |
| **Same-Day ACH** | 0.3% + $0.15 | Capped at $3 per transfer |
| **Next-Day ACH** | 0.1% + $0.10 | Capped at $1 per transfer |
| **Account Verification** | $0.50 | Per Plaid verification |
| **Request-to-Pay** | $0.25 | Per request sent |

### Volume Discounts

| Monthly Volume | Discount |
|----------------|----------|
| $0 - $100K | Standard pricing |
| $100K - $1M | 10% discount |
| $1M - $10M | 25% discount |
| $10M+ | Custom enterprise pricing |

---

## Competitive Analysis

| Feature | FlowPay | Stripe ACH | Plaid Transfer | Dwolla |
|---------|---------|------------|----------------|--------|
| FedNow Support | Yes | No | No | Yes |
| RTP Support | Yes | No | No | Yes |
| Instant Verification | Yes | Via Plaid | Native | Via Plaid |
| Balance Check | Yes | No | Yes | No |
| Request-to-Pay | Yes | No | No | No |
| Smart Rail Selection | AI-powered | Manual | Manual | Manual |
| 24/7 Instant | Yes | No | No | Yes |

---

## Implementation Roadmap

| Phase | Timeline | Deliverables |
|-------|----------|--------------|
| **Foundation** | Q1 2026 | ACH integration, account verification, basic transfers |
| **Instant Rails** | Q2 2026 | FedNow and RTP integration, 24/7 availability |
| **Intelligence** | Q3 2026 | Smart routing, balance verification, risk scoring |
| **Advanced** | Q4 2026 | Request-to-Pay, batch payouts, international expansion |

---

## Success Metrics

| Metric | Target |
|--------|--------|
| Transfer Volume | $500M/year |
| Settlement Speed (Instant) | <10 seconds p99 |
| ACH Return Rate | <1% |
| Bank Coverage (FedNow) | >80% of US banks |
| API Uptime | 99.99% |

---

*End of Document*
