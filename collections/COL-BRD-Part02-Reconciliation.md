# PART 02 -- RECONCILIATION BOUNDED CONTEXT (BC-01)

---

## 7. RECONCILIATION CONTEXT

### 7.1 Context Overview

| Attribute | Details |
|---|---|
| Context Name | Reconciliation Context |
| Context ID | BC-01 |
| Domain Classification | Supporting Domain |
| Owner Team | Finance Operations / Loan Operations |
| Upstream Dependencies | External CBS/LMS (via ACL), Payment Gateway, NPCI (NACH/UPI), Banks (NEFT/RTGS) |
| Downstream Consumers | Delinquency Context (BC-02), Compliance & Audit Context (BC-10), Analytics Context (BC-11) |
| Primary Objective | Accurately match incoming payment transactions from all payment rails to the correct EMI installment within SLA, minimizing suspense and manual intervention |

### 7.2 Ubiquitous Language -- Reconciliation-Specific Terms

| Term | Definition |
|---|---|
| **Payment Rail** | A payment channel/network through which funds are received. Includes NACH, UPI, NEFT/RTGS, cheque, cash, BBPS, payment gateway, wallet, QR code. |
| **Matching Rule** | A configurable rule that defines criteria for matching an incoming transaction to an EMI. Rules include exact match, fuzzy match, reference-based match. |
| **Match Confidence** | A score (0-100%) indicating the system's confidence in a proposed match. Configurable threshold determines auto-match vs manual review. |
| **Suspense Entry** | An unmatched transaction parked in a suspense account pending resolution. |
| **Suspense Aging** | The number of days a transaction has remained unmatched in suspense. |
| **Auto-Match** | System automatically matches a transaction to an EMI without human intervention, based on matching rules and confidence threshold. |
| **Manual Match** | A transaction that requires human review to match to an EMI, typically because auto-match confidence is below threshold. |
| **Bounce/Return** | Failed auto-debit attempt returned by the borrower's bank with a return reason code. |
| **Return Code** | Standardized reason code for NACH/ECS bounce (e.g., 01=Insufficient Funds, 02=Account Closed, 03=No Such Account, 68=Account Frozen). |
| **Re-presentation** | Re-submitting a bounced NACH mandate for collection after a configurable waiting period. |
| **Break** | A mismatch between two reconciliation sources (e.g., bank statement vs LMS collections). |
| **Clearing** | The process of resolving a suspense entry by matching it to the correct loan account. |
| **UTR (Unique Transaction Reference)** | A unique identifier assigned to NEFT/RTGS transactions by the originating bank. |
| **Batch File** | A file containing multiple transactions processed together (e.g., NACH response file, BBPS settlement file). |

### 7.3 Aggregates

#### 7.3.1 Aggregate: ReconciliationBatch

| Aspect | Details |
|---|---|
| Aggregate Root | `ReconciliationBatch` |
| Identity | `ReconciliationBatchId` (UUID) |
| Description | Represents a batch of incoming transactions from a single payment rail for a specific date/session. |

**Entities:**
- `ReconciliationBatch` — Root. Contains batch metadata (source rail, file reference, date, total count, total amount, status).
- `IncomingTransaction` — Each transaction within the batch (transaction ID, amount, payer reference, UTR/UMRN, timestamp, raw data).

**Value Objects:**
- `PaymentRail` — Enum: NACH, UPI_AUTOPAY, UPI_COLLECT, NEFT, RTGS, IMPS, CHEQUE, CASH, BBPS, PAYMENT_GATEWAY, WALLET, QR_CODE
- `Money` — (amount, currency)
- `BatchStatus` — Enum: RECEIVED, VALIDATING, MATCHING, MATCHED, PARTIALLY_MATCHED, COMPLETED, FAILED
- `TransactionReference` — (referenceType, referenceValue) e.g., (UTR, "UTIB12345678")

**Invariants:**
- Sum of individual transaction amounts must equal batch total amount
- A batch cannot transition to COMPLETED until all transactions are either matched or explicitly moved to suspense
- No duplicate transactions within a batch (based on transaction ID + rail + date)

#### 7.3.2 Aggregate: MatchedPayment

| Aspect | Details |
|---|---|
| Aggregate Root | `MatchedPayment` |
| Identity | `MatchedPaymentId` (UUID) |
| Description | Represents a confirmed match between an incoming transaction and an EMI installment. |

**Entities:**
- `MatchedPayment` — Root. Links incoming transaction to loan account + EMI.

**Value Objects:**
- `MatchResult` — (matchType [AUTO/MANUAL], confidenceScore, matchedBy [rule ID or user ID], matchTimestamp)
- `EMIReference` — (loanAccountId, emiNumber, emiDueDate, emiAmount)
- `AllocationBreakup` — (principalPortion, interestPortion, penalInterestPortion, feesPortion, excessAmount)
- `MatchStatus` — Enum: PROPOSED, CONFIRMED, REVERSED

**Invariants:**
- A single incoming transaction can be matched to at most one EMI (unless it is a bulk/combined payment covering multiple EMIs)
- Once CONFIRMED, a match can only be reversed via an explicit reversal event with audit trail
- Allocation breakup must sum to the transaction amount

#### 7.3.3 Aggregate: SuspenseEntry

| Aspect | Details |
|---|---|
| Aggregate Root | `SuspenseEntry` |
| Identity | `SuspenseEntryId` (UUID) |
| Description | An unmatched transaction parked in suspense pending resolution. |

**Entities:**
- `SuspenseEntry` — Root. Contains unmatched transaction details, aging, resolution attempts.
- `ResolutionAttempt` — Records each attempt to resolve (manual match, return to payer, write-off, force-match).

**Value Objects:**
- `SuspenseStatus` — Enum: OPEN, UNDER_REVIEW, RESOLVED, RETURNED_TO_PAYER, WRITTEN_OFF
- `SuspenseAging` — (entryDate, currentAgeDays, agingBucket [0-7, 8-15, 16-30, 31-60, 60+])
- `SuspenseReason` — Enum: AMOUNT_MISMATCH, NO_REFERENCE, INVALID_ACCOUNT, DUPLICATE_SUSPECTED, PARTIAL_PAYMENT, EXCESS_PAYMENT, UNKNOWN

**Invariants:**
- Suspense entries older than 30 days must be escalated to a supervisor
- Suspense entries older than 90 days must be reported to management
- GL suspense account balance must equal sum of all OPEN suspense entries

#### 7.3.4 Aggregate: BounceRecord

| Aspect | Details |
|---|---|
| Aggregate Root | `BounceRecord` |
| Identity | `BounceRecordId` (UUID) |
| Description | Represents a failed auto-debit (NACH/ECS/UPI AutoPay) returned by the destination bank. |

**Entities:**
- `BounceRecord` — Root. Contains mandate reference, bounce details, return code, re-presentation eligibility.

**Value Objects:**
- `ReturnCode` — (code, description, category [TECHNICAL/FINANCIAL/ADMINISTRATIVE])
- `BounceCategory` — Enum: INSUFFICIENT_FUNDS, ACCOUNT_CLOSED, MANDATE_CANCELLED, TECHNICAL_ERROR, FROZEN_ACCOUNT, OTHER
- `RepresentationEligibility` — (eligible: boolean, nextEligibleDate, maxAttempts, attemptsUsed)

**Invariants:**
- A bounce must be linked to a valid mandate (UMRN) and loan account
- Re-presentation cannot exceed maximum attempts per RBI/NPCI guidelines (typically 1 re-presentation for NACH)
- Bounce charges can only be levied as per board-approved schedule of charges

### 7.4 Domain Events

| Event ID | Event Name | Published When | Payload (Key Fields) | Consumed By |
|---|---|---|---|---|
| DE-REC-001 | `ReconciliationBatchReceived` | New batch file/stream received from a payment rail | batchId, rail, transactionCount, totalAmount, receivedAt | Internal (triggers matching) |
| DE-REC-002 | `TransactionAutoMatched` | Auto-matching engine matches a transaction to EMI | matchedPaymentId, loanAccountId, emiNumber, amount, rail, confidence | Delinquency (BC-02), Account (BC-06) |
| DE-REC-003 | `TransactionManuallyMatched` | Ops user manually matches a transaction | matchedPaymentId, loanAccountId, emiNumber, matchedBy | Delinquency (BC-02), Account (BC-06) |
| DE-REC-004 | `TransactionMovedToSuspense` | Transaction cannot be auto-matched, moved to suspense | suspenseEntryId, transactionId, amount, reason | Compliance (BC-10) |
| DE-REC-005 | `SuspenseEntryResolved` | Suspense entry matched or otherwise resolved | suspenseEntryId, resolution, loanAccountId (if matched) | Delinquency (BC-02), Analytics (BC-11) |
| DE-REC-006 | `BounceReceived` | NACH/ECS/UPI return file processed, bounce identified | bounceRecordId, loanAccountId, umrn, returnCode, amount, bounceDate | Delinquency (BC-02), Workflow (BC-03), Account (BC-06) |
| DE-REC-007 | `BounceChargesApplied` | Bounce/return charges debited to loan account | loanAccountId, chargeAmount, chargeType | Account (BC-06) |
| DE-REC-008 | `RepresentationScheduled` | Bounced mandate scheduled for re-presentation | loanAccountId, umrn, representationDate, amount | Mandate (BC-06) |
| DE-REC-009 | `ReconciliationBreakIdentified` | Mismatch found between two sources during reconciliation | breakId, sourceA, sourceB, breakAmount, breakType | Compliance (BC-10) |
| DE-REC-010 | `PaymentReversed` | A previously matched payment is reversed (chargeback, returned cheque) | matchedPaymentId, loanAccountId, reversalReason | Delinquency (BC-02), Account (BC-06) |

### 7.5 Domain Services

#### 7.5.1 ReconciliationMatchingService

**Purpose:** Core matching engine that applies configurable rules to match incoming transactions to EMIs.

**Matching Algorithm (Waterfall):**

```
Step 1: EXACT REFERENCE MATCH
  - Match by UMRN (NACH) or Mandate ID (UPI AutoPay) → loan account
  - Match by UTR → pre-registered UTR from payment link
  - Match by virtual account number → loan-specific VA
  - If exact match found AND amount matches EMI due → AUTO-MATCH (confidence: 100%)

Step 2: REFERENCE + AMOUNT MATCH
  - Match by customer ID / loan account number in payer reference field
  - Validate amount against due amount (exact, or within tolerance of ±INR 1)
  - If matched → AUTO-MATCH (confidence: 95%)

Step 3: AMOUNT + DATE HEURISTIC
  - Group unmatched transactions by amount
  - Match against EMIs due on or around the transaction date (±3 days)
  - Cross-reference payer bank account with registered bank account
  - If matched → AUTO-MATCH if confidence > configurable threshold (default: 85%)

Step 4: FUZZY MATCH
  - Parse payer name, remarks, reference fields for embedded loan/account info
  - Use historical payment patterns (same payer, same amount, same date pattern)
  - If matched → PROPOSED MATCH for manual review (confidence: 60-84%)

Step 5: SUSPENSE
  - No match found → Move to suspense with reason code
```

**Business Rules:**

| Rule ID | Rule | Example |
|---|---|---|
| COL-REC-BR-001 | NACH transactions must be matched first by UMRN, then by amount + account | UMRN "NACH000001234567" → Loan ACC/2024/HL/001234 |
| COL-REC-BR-002 | Excess payment (amount > total due) must be credited to loan account as advance and the excess flagged for refund review | EMI due: ₹15,000. Received: ₹20,000. Match ₹15,000 to EMI, flag ₹5,000 as excess |
| COL-REC-BR-003 | Short payment (amount < EMI but within tolerance) can be auto-matched with short-payment flag | EMI due: ₹15,000. Received: ₹14,990. Auto-match with shortfall of ₹10 |
| COL-REC-BR-004 | Partial payment (amount significantly < EMI) must follow payment allocation waterfall | EMI due: ₹15,000. Received: ₹8,000. Allocate per waterfall: fees → penal → overdue interest → overdue principal → current interest → current principal |
| COL-REC-BR-005 | Duplicate transaction detection: same amount, same payer, same date, same rail within 24 hours triggers duplicate alert | Two NEFT of ₹15,000 from same account on same day → flag second as potential duplicate |
| COL-REC-BR-006 | Cheque payments must not be marked as matched until cheque clearing confirmation (T+1 for CTS, T+3 for non-CTS) | Cheque deposited Day 0 → Provisional match. CTS clear Day 1 → Confirmed match |

#### 7.5.2 BounceProcessingService

**Purpose:** Processes bounce/return files from NACH, ECS, and UPI AutoPay and triggers downstream actions.

**NACH Return Code Handling:**

| Return Code | Description | Category | Action |
|---|---|---|---|
| 01 | Insufficient Funds | Financial | Mark bounce; trigger DPD event; schedule re-presentation if eligible; send bounce notification |
| 02 | Account Closed | Administrative | Mark bounce; cancel mandate; trigger mandate refresh; escalate to collections |
| 03 | No Such Account | Administrative | Mark bounce; cancel mandate; verify KYC; raise data quality alert |
| 04 | Mandate not registered / Cancelled | Technical | Mark bounce; re-initiate mandate registration; escalate |
| 10 | Account Frozen/Blocked | Administrative | Mark bounce; legal hold check; escalate |
| 12 | Mandate Expired | Technical | Mark bounce; initiate mandate renewal; escalate |
| 51 | Amount exceeds mandate limit | Technical | Mark bounce; check if EMI increased (rate reset); update mandate amount |
| 68 | Account Debited but NACH Return | Technical | Mark provisional bounce; initiate trace; may auto-reverse if funds confirmed |

#### 7.5.3 GLReconciliationService

**Purpose:** Reconciles collection sub-ledger (loan account level postings) with GL entries.

**Reconciliation Points:**
- Total collections posted to individual loan accounts (sub-ledger) = Total credit to collections GL account
- Suspense GL balance = Sum of all open suspense entries
- Bounce reversal entries balance across sub-ledger and GL
- Interest income GL = Sum of interest components across all matched payments
- Principal collection GL = Sum of principal components across all matched payments

### 7.6 Application Services (Use Cases)

| Use Case ID | Use Case Name | Actor | Description |
|---|---|---|---|
| COL-REC-UC-001 | Process NACH Response File | System (Scheduled) | Receive NACH response file from sponsor bank, parse transactions, separate successes and bounces, trigger matching and bounce processing |
| COL-REC-UC-002 | Process UPI AutoPay Callback | System (Event) | Receive real-time UPI AutoPay success/failure callback, trigger matching or bounce processing |
| COL-REC-UC-003 | Process NEFT/RTGS/IMPS Credit | System (Event) | Receive credit notification from CBS, trigger matching engine |
| COL-REC-UC-004 | Process Cheque Collection | Ops User | Enter cheque details, create provisional match, confirm on clearing |
| COL-REC-UC-005 | Process Cash Collection | Branch User | Enter cash receipt, immediately match to EMI, generate receipt |
| COL-REC-UC-006 | Process Payment Gateway Settlement | System (Scheduled) | Receive settlement file from payment gateway, reconcile with individual payment transactions |
| COL-REC-UC-007 | Resolve Suspense Entry | Ops User | Review unmatched transaction, manually match or return to payer |
| COL-REC-UC-008 | Run GL Reconciliation | System (Scheduled/EOD) | Compare sub-ledger totals to GL, identify and report breaks |
| COL-REC-UC-009 | Generate Reconciliation Report | Ops User / System | Generate daily/weekly/monthly reconciliation MIS |
| COL-REC-UC-010 | Schedule NACH Re-presentation | System | For eligible bounces, schedule re-presentation on next eligible date |
| COL-REC-UC-011 | Process BBPS Collection | System (Event) | Receive BBPS payment confirmation, match to loan account biller ID |
| COL-REC-UC-012 | Reverse Matched Payment | Ops Supervisor | Reverse a confirmed match due to chargeback, returned cheque, or operational error |

### 7.7 Functional Requirements

#### 7.7.1 Multi-Rail Transaction Ingestion

| Req ID | Requirement | Priority | Acceptance Criteria |
|---|---|---|---|
| COL-REC-001 | System shall ingest NACH response files (ACH Credit/Debit Response — NPCI format) via SFTP on a scheduled basis (minimum 4 times per day: morning, afternoon, evening, night cycles). | P0 | NACH files parsed within 15 minutes of receipt; 100% transactions extracted |
| COL-REC-002 | System shall process real-time UPI AutoPay callbacks via API (NPCI UPI 2.0 recurring payment response). | P0 | Callback processed within 5 seconds; success/failure recorded |
| COL-REC-003 | System shall ingest NEFT/RTGS/IMPS credit notifications from CBS via API or message queue in real-time. | P0 | Credit notification processed within 30 seconds of receipt |
| COL-REC-004 | System shall support cheque collection entry (CTS and non-CTS) with provisional matching and clearing-based confirmation. | P1 | Provisional match on deposit; auto-confirm on CTS clearing (T+1) |
| COL-REC-005 | System shall support cash collection entry at branch with instant receipt generation. | P1 | Cash matched instantly; receipt generated with QR code for verification |
| COL-REC-006 | System shall ingest payment gateway settlement files (Razorpay, PayU, BillDesk, CCAvenue) and reconcile with individual transactions. | P0 | Settlement file reconciled within 30 minutes; breaks identified |
| COL-REC-007 | System shall process BBPS payment confirmations for loans registered as BBPS billers. | P1 | BBPS payment matched to loan account within 5 minutes |
| COL-REC-008 | System shall support wallet-based payments (Paytm, PhonePe, etc.) with settlement reconciliation. | P2 | Wallet payments reconciled within 1 hour of settlement |
| COL-REC-009 | System shall support QR code-based payments where QR is linked to specific loan account. | P1 | QR payment auto-matched via virtual account mapping |
| COL-REC-010 | System shall support standing instruction (SI) collections from savings accounts within the same bank. | P1 | SI success/failure processed same day |

#### 7.7.2 Matching Engine

| Req ID | Requirement | Priority | Acceptance Criteria |
|---|---|---|---|
| COL-REC-011 | System shall implement configurable matching rules engine with waterfall execution (exact → reference → heuristic → fuzzy → suspense). | P0 | Rules configurable via admin UI; waterfall executes in sequence |
| COL-REC-012 | System shall achieve auto-match rate of ≥97% for NACH transactions (matched by UMRN). | P0 | Monthly auto-match rate ≥97% for NACH |
| COL-REC-013 | System shall achieve auto-match rate of ≥90% across all payment rails combined. | P0 | Monthly combined auto-match rate ≥90% |
| COL-REC-014 | System shall support configurable confidence threshold for auto-matching (default: 85%). | P0 | Threshold configurable per rail and per product type |
| COL-REC-015 | System shall detect and flag potential duplicate transactions (same amount, same payer, same rail, within 24 hours). | P0 | Duplicates flagged with <1% false positive rate |
| COL-REC-016 | System shall handle partial payments by applying payment allocation waterfall (configurable per product). | P0 | Partial payment allocated correctly per waterfall rules |
| COL-REC-017 | System shall handle excess payments by matching to due amount and flagging excess for refund/advance processing. | P0 | Excess amount correctly identified and routed |
| COL-REC-018 | System shall handle combined payments (single transaction covering multiple EMIs) by splitting and matching to multiple installments. | P1 | Multi-EMI match with correct allocation per installment |
| COL-REC-019 | System shall support third-party payments (payment from non-borrower) with proper identification and AML flagging if above threshold. | P1 | Third-party payment flagged; matched if reference valid |
| COL-REC-020 | System shall maintain complete audit trail of matching decisions (rule applied, confidence score, matched/proposed by). | P0 | Every match decision auditable with full context |

#### 7.7.3 Suspense Management

| Req ID | Requirement | Priority | Acceptance Criteria |
|---|---|---|---|
| COL-REC-021 | System shall automatically move unmatched transactions to suspense with categorized reason code. | P0 | All unmatched transactions in suspense within 1 hour of batch completion |
| COL-REC-022 | System shall track suspense aging (0-7, 8-15, 16-30, 31-60, 60+ days) and auto-escalate based on aging. | P0 | Escalation triggered at configured aging thresholds |
| COL-REC-023 | System shall provide a suspense resolution workbench for ops users with suggested matches (AI-assisted). | P0 | Workbench shows top 3 suggested matches per suspense entry |
| COL-REC-024 | System shall support bulk resolution of suspense entries for common patterns (e.g., all entries from same payer). | P1 | Bulk resolution with maker-checker approval |
| COL-REC-025 | Suspense GL account balance must be reconciled daily with system suspense entries. | P0 | Daily reconciliation with zero tolerance; breaks reported immediately |
| COL-REC-026 | System shall support return-to-payer for permanently unresolvable suspense entries (with maker-checker). | P1 | Return initiated within 2 business days of decision |

#### 7.7.4 Bounce Processing

| Req ID | Requirement | Priority | Acceptance Criteria |
|---|---|---|---|
| COL-REC-027 | System shall process NACH return files and categorize bounces by return code (financial, technical, administrative). | P0 | 100% return codes mapped; categorized within 30 minutes |
| COL-REC-028 | System shall apply bounce charges as per board-approved schedule of charges (configurable per product). | P0 | Charges applied correctly; compliant with RBI fair lending norms |
| COL-REC-029 | System shall evaluate re-presentation eligibility based on return code, attempt count, and institution policy. | P0 | Re-presentation only for eligible codes; max attempts enforced |
| COL-REC-030 | System shall schedule re-presentation on configurable date (default: next business day + 3 calendar days). | P0 | Re-presentation file generated on schedule |
| COL-REC-031 | System shall publish BounceReceived event to downstream contexts within 1 minute of bounce identification. | P0 | Event published with <1 minute latency |
| COL-REC-032 | System shall track bounce history per mandate (UMRN) and per loan account for pattern analysis. | P0 | Complete bounce history accessible; supports analytics |
| COL-REC-033 | System shall auto-cancel mandates after N consecutive bounces (configurable, default: 3) and trigger mandate refresh workflow. | P1 | Mandate cancelled after threshold; refresh initiated |

#### 7.7.5 GL Reconciliation

| Req ID | Requirement | Priority | Acceptance Criteria |
|---|---|---|---|
| COL-REC-034 | System shall perform daily sub-ledger to GL reconciliation for all collection-related GL accounts. | P0 | Reconciliation completed within EOD batch window |
| COL-REC-035 | System shall identify and report GL breaks with categorization (timing difference, data error, system error). | P0 | All breaks identified; report generated by 8 AM next day |
| COL-REC-036 | System shall support auto-resolution of timing differences (e.g., NEFT credited in CBS but batch not yet processed in LMS). | P1 | Timing differences auto-resolved within T+1 |
| COL-REC-037 | System shall generate GL reconciliation report with sign-off workflow for Finance team. | P0 | Report with break details; sign-off captured |
| COL-REC-038 | System shall reconcile interest income GL with interest components of all matched payments. | P0 | Interest income reconciled to the rupee |
| COL-REC-039 | System shall reconcile principal collection GL with principal components of all matched payments. | P0 | Principal collection reconciled to the rupee |

#### 7.7.6 Reconciliation Dashboards & Reports

| Req ID | Requirement | Priority | Acceptance Criteria |
|---|---|---|---|
| COL-REC-040 | System shall provide real-time reconciliation dashboard showing: total received, matched, pending, suspense, bounced — per rail, per product, per date. | P0 | Dashboard refreshes within 5 minutes of data change |
| COL-REC-041 | System shall generate daily reconciliation summary report (auto-emailed to configured recipients). | P0 | Report delivered by 8 AM |
| COL-REC-042 | System shall generate suspense aging report with drill-down to individual entries. | P0 | Report available on-demand and in daily MIS |
| COL-REC-043 | System shall generate bounce analysis report (by return code, by product, by sponsor bank, trend). | P0 | Report available daily and monthly |
| COL-REC-044 | System shall generate payment rail performance report (success rate, processing time, cost per transaction). | P1 | Monthly report with trend analysis |

### 7.8 Integration Events Published

| Event | Target Context | Payload |
|---|---|---|
| `PaymentCredited` | Delinquency (BC-02), Account (BC-06) | loanAccountId, amount, rail, matchedPaymentId, emiNumber, allocationBreakup, creditDate |
| `BounceConfirmed` | Delinquency (BC-02), Workflow (BC-03), Account (BC-06) | loanAccountId, umrn, returnCode, bounceCategory, bounceDate, amount |
| `SuspenseEntryCreated` | Compliance (BC-10) | suspenseEntryId, amount, rail, reason, transactionDate |
| `SuspenseCleared` | Delinquency (BC-02) | suspenseEntryId, loanAccountId, amount, clearingDate |
| `ReconciliationBreakFound` | Compliance (BC-10) | breakId, sourceA, sourceB, breakAmount, breakDate |
| `PaymentReversed` | Delinquency (BC-02), Account (BC-06) | matchedPaymentId, loanAccountId, reversalAmount, reversalReason |

### 7.9 Sample Calculation -- Payment Allocation Waterfall

**Scenario:** Loan account ACC/2024/PL/005678 has following dues:

| Component | Amount |
|---|---|
| Bounce charges (fee) | ₹500 |
| Penal interest (overdue) | ₹1,200 |
| Overdue interest (previous EMI) | ₹3,500 |
| Overdue principal (previous EMI) | ₹8,500 |
| Current month interest | ₹3,200 |
| Current month principal | ₹8,800 |
| **Total due** | **₹25,700** |

**Payment received:** ₹15,000 via NEFT

**Allocation (default waterfall):**

| Step | Component | Allocated | Remaining |
|---|---|---|---|
| 1 | Bounce charges (fees) | ₹500 | ₹14,500 |
| 2 | Penal interest | ₹1,200 | ₹13,300 |
| 3 | Overdue interest | ₹3,500 | ₹9,800 |
| 4 | Overdue principal | ₹8,500 | ₹1,300 |
| 5 | Current interest | ₹1,300 | ₹0 |
| 6 | Current principal | ₹0 | — |

**Result:** Previous EMI fully cleared (account upgrades from overdue). Current EMI partially paid (₹1,300 of ₹3,200 interest paid; ₹1,900 interest + ₹8,800 principal still due).

---

## END OF PART 02

Next: **COL-BRD-Part03-Delinquency.md** -- Delinquency Bounded Context with DPD Engine, Bucket Management, NPA Classification.
