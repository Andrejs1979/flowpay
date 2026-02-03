# FlowPay Sprint Plan

> Sprint planning for FlowPay development
> Last updated: February 2026

---

## Sprint Overview

| Sprint | Duration | Focus | Status |
|--------|----------|-------|--------|
| **Sprint 0** | Week 1 | Project Setup & Foundation | 🔵 Active |
| **Sprint 1** | Weeks 2-3 | Account Verification | ⚪ Planned |
| **Sprint 2** | Weeks 4-5 | ACH Transfers | ⚪ Planned |
| **Sprint 3** | Weeks 6-7 | Instant Rails (FedNow) | ⚪ Planned |
| **Sprint 4** | Weeks 8-9 | Instant Rails (RTP) | ⚪ Planned |
| **Sprint 5** | Weeks 10-11 | Smart Rail Selection | ⚪ Planned |
| **Sprint 6** | Weeks 12-13 | Request-to-Pay | ⚪ Planned |
| **Sprint 7** | Weeks 14-15 | Batch Payouts | ⚪ Planned |

---

## Sprint 0: Project Setup & Foundation

**Dates:** Week 1
**Goal:** Establish development environment and infrastructure

### Tasks

#### Infrastructure Setup
- [x] Initialize repository with git
- [x] Create planning documents (PRD, EPICS, PROGRESS)
- [x] Create GitHub Issues for all epics
- [ ] Set up Cloudflare Workers project
- [ ] Configure wrangler.toml
- [ ] Create D1 database schema
- [ ] Set up KV namespaces
- [ ] Configure Cloudflare Queues

#### Development Environment
- [ ] TypeScript configuration
- [ ] Vitest testing setup
- [ ] ESLint & Prettier
- [ ] CI/CD pipeline
- [ ] Environment variable management

#### Marketing Website
- [ ] Create Astro marketing site
- [ ] Deploy to Cloudflare Pages

### Definition of Done
- [ ] All infrastructure deployed to development environment
- [ ] CI/CD pipeline passing
- [ ] Marketing site live at flowpay.pages.dev
- [ ] Team onboarding complete

---

## Sprint 1: Account Verification

**Dates:** Weeks 2-3
**Goal:** Enable secure bank account linking and verification
**Epic:** Epic 1: Account Verification (#1)

### Stories

#### Story 1.1: Link Bank Account
- [ ] POST /v1/accounts endpoint
- [ ] Account number encryption in KV
- [ ] Routing number validation (FedACH)
- [ ] Bank name lookup
- [ ] Unit tests (>80% coverage)
- [ ] Integration tests
- [ ] API documentation

#### Story 1.2: Instant Account Verification (Plaid)
- [ ] Plaid Link integration
- [ ] POST /v1/accounts/:id/verify
- [ ] Token exchange flow
- [ ] Secure credential storage
- [ ] Error handling for Plaid failures
- [ ] Tests with Plaid sandbox

#### Story 1.3: Account Balance Check
- [ ] GET /v1/accounts/:id/balance
- [ ] Plaid Balance integration
- [ ] 5-minute response caching
- [ ] Permission checks (verified only)
- [ ] Error handling

#### Story 1.4: Micro-Deposit Verification (Fallback)
- [ ] POST /v1/accounts/:id/verify (micro-deposits)
- [ ] Random deposit generation (<$1.00)
- [ ] POST /v1/accounts/:id/confirm
- [ ] 3-attempt limit with account lock
- [ ] Automated recall after verification

### Definition of Done
- [ ] All acceptance criteria pass
- [ ] Unit tests >80% coverage
- [ ] Integration tests pass
- [ ] API documentation updated
- [ ] Security review complete
- [ ] Deployed to staging

---

## Sprint 2: ACH Transfers

**Dates:** Weeks 4-5
**Goal:** Process standard ACH bank-to-bank transfers
**Epic:** Epic 2: ACH Transfers (#2)

### Stories

#### Story 2.1: Create ACH Transfer
- [ ] POST /v1/transfers endpoint
- [ ] Double-entry ledger in D1
- [ ] Credit/debit direction support
- [ ] Next-day ACH default
- [ ] Fee calculation
- [ ] transfer.created webhook

#### Story 2.2: Same-Day ACH
- [ ] Same-day service class code
- [ ] Cutoff time enforcement (2PM CT)
- [ ] Weekend/holiday detection
- [ ] Higher fee logic
- [ ] Fallback to next-day

#### Story 2.3: Transfer Status Tracking
- [ ] GET /v1/transfers/:id
- [ ] Status state machine
- [ ] Trace number tracking
- [ ] Error detail returns

#### Story 2.4: Transfer Webhooks
- [ ] Webhook configuration (dashboard/API)
- [ ] All webhook events (pending, completed, failed, returned)
- [ ] HMAC signature verification
- [ ] Exponential backoff retry
- [ ] Test webhook endpoint

### Definition of Done
- [ ] All acceptance criteria pass
- [ ] End-to-end ACH flow working
- [ ] Webhook delivery verified
- [ ] Tests >80% coverage
- [ ] Deployed to staging

---

## Sprint 3: Instant Rails (FedNow)

**Dates:** Weeks 6-7
**Goal:** Enable FedNow instant payments
**Epic:** Epic 3: Instant Rails (#3)

### Stories

#### Story 3.1: FedNow Integration
- [ ] FedNow API connection
- [ ] rail: "fednow" transfer option
- [ ] <10 second settlement
- [ ] $500K limit enforcement
- [ ] Real-time completion status

#### Story 3.3: Bank Participation Check
- [ ] GET /v1/rails/participation
- [ ] FedNow participation directory
- [ ] 24-hour caching
- [ ] Credit push verification

### Definition of Done
- [ ] FedNow transfers working end-to-end
- [ ] <10 second p99 settlement time
- [ ] 24/7 availability verified
- [ ] Deployed to staging

---

## Sprint 4: Instant Rails (RTP)

**Dates:** Weeks 8-9
**Goal:** Enable RTP high-value instant payments
**Epic:** Epic 3: Instant Rails (#3)

### Stories

#### Story 3.2: RTP Network Integration
- [ ] RTP API connection
- [ ] rail: "rtp" transfer option
- [ ] <15 second settlement
- [ ] $1M limit enforcement
- [ ] Sender/receiver identification
- [ ] 200-char memo field support

#### Story 3.3: Bank Participation Check (RTP)
- [ ] RTP participation directory
- [ ] Request-for-pay support detection

### Definition of Done
- [ ] RTP transfers working end-to-end
- [ ] <15 second p99 settlement time
- [ ] Deployed to staging

---

## Sprint 5: Smart Rail Selection

**Dates:** Weeks 10-11
**Goal:** AI-powered automatic rail routing
**Epic:** Epic 4: Smart Rail Selection (#4)

### Stories

#### Story 4.1: Intelligent Rail Router
- [ ] rail: "auto" transfer option
- [ ] ML model for routing decisions
- [ ] Factors: urgency, amount, cost, bank support
- [ ] Routing audit logging
- [ ] Fallback logic

#### Story 4.2: Cost Optimization
- [ ] Cost preference settings
- [ ] Standard urgency → lowest cost
- [ ] Cost comparison dashboard
- [ ] Instant selection alerts

### Definition of Done
- [ ] Auto-routing production-ready
- [ ] ML model accuracy >90%
- [ ] Deployed to staging

---

## Sprint 6: Request-to-Pay

**Dates:** Weeks 12-13
**Goal:** Enable invoice-based payment requests
**Epic:** Epic 5: Request-to-Pay (#5)

### Stories

#### Story 5.1: Create Payment Request
- [ ] POST /v1/requests
- [ ] Invoice details storage
- [ ] Approval URL generation
- [ ] QR code generation
- [ ] 30-day expiration
- [ ] Email/SMS notifications

#### Story 5.2: Request Status & Webhooks
- [ ] GET /v1/requests/:id
- [ ] Status state machine
- [ ] request.approved webhook
- [ ] request.declined webhook
- [ ] Request cancellation

### Definition of Done
- [ ] Request-to-Pay flow working
- [ ] Deployed to staging

---

## Sprint 7: Batch Payouts

**Dates:** Weeks 14-15
**Goal:** Enable bulk transfer processing
**Epic:** Epic 6: Batch Payouts (#6)

### Stories

#### Story 6.1: Batch Payout Creation
- [ ] POST /v1/payouts
- [ ] Up to 10,000 items per batch
- [ ] Partial processing support
- [ ] Individual item status tracking
- [ ] Batch completion webhook

### Definition of Done
- [ ] Batch payouts working
- [ ] Performance tests pass (10K items)
- [ ] Deployed to staging

---

## Release Planning

| Milestone | Target Sprint | Deliverables |
|-----------|---------------|--------------|
| **Foundation MVP** | Sprint 2 | Account verification + ACH transfers |
| **Instant Payments** | Sprint 4 | FedNow + RTP support |
| **Smart Routing** | Sprint 5 | AI-powered rail selection |
| **Full Platform** | Sprint 7 | Request-to-Pay + Batch payouts |

---

## Sprint Ceremonies

### Sprint Planning
- **When:** Monday morning of each sprint
- **Duration:** 1 hour
- **Attendees:** Entire team
- **Goal:** Plan sprint backlog and capacity

### Daily Standup
- **When:** Daily at 10 AM
- **Duration:** 15 minutes
- **Format:** Yesterday, Today, Blockers

### Sprint Review
- **When:** Last day of sprint
- **Duration:** 1 hour
- **Goal:** Demo completed work to stakeholders

### Sprint Retrospective
- **When:** Last day of sprint
- **Duration:** 30 minutes
- **Goal:** Identify improvements

---

## Velocity Tracking

| Sprint | Planned | Completed | Velocity |
|--------|---------|-----------|----------|
| Sprint 0 | TBD | TBD | TBD |
| Sprint 1 | TBD | TBD | TBD |

---

*End of Document*
