# PART 03 -- DELINQUENCY BOUNDED CONTEXT (BC-02)

---

## 8. DELINQUENCY CONTEXT -- DPD ENGINE, BUCKET MANAGEMENT, NPA CLASSIFICATION

### 8.1 Context Overview

| Attribute | Details |
|---|---|
| Context Name | Delinquency Context |
| Context ID | BC-02 |
| Domain Classification | Core Domain |
| Owner Team | Risk & Collections |
| Upstream Dependencies | Reconciliation Context (BC-01) — PaymentCredited, BounceConfirmed events; External LMS — EMI schedule, loan master |
| Downstream Consumers | Workflow (BC-03), Sentiment (BC-05), Agency (BC-07), Legal (BC-08), Compliance (BC-10), Analytics (BC-11) |
| Primary Objective | Maintain real-time, accurate DPD for every loan account; classify accounts into regulatory buckets; drive NPA recognition; trigger downstream collection actions |

### 8.2 Ubiquitous Language -- Delinquency-Specific Terms

| Term | Definition |
|---|---|
| **DPD (Days Past Due)** | Calendar days since the oldest unpaid installment's due date. Calculated daily as: `DPD = today - oldest_unpaid_emi_due_date`. If no installment is overdue, DPD = 0. |
| **Bucket** | DPD range-based classification. Standard buckets per RBI IRAC: Current (0), SMA-0 (1-30), SMA-1 (31-60), SMA-2 (61-90), NPA-SS (91-365), NPA-D1 (366-730), NPA-D2 (731-1095), NPA-D3 (1096-1460), Loss (1461+). |
| **Bucket Movement** | Transition between buckets. Forward roll (upgrade to worse bucket) or backward roll (downgrade to better bucket). |
| **Asset Classification** | RBI IRAC norm-based classification: Standard (performing), Sub-standard, Doubtful (D1/D2/D3), Loss. |
| **Provisioning** | Reserve amount set aside against potential losses. Rates per RBI: Standard 0.40% (0.25% for agriculture/MSE), Sub-standard 15% (25% unsecured), Doubtful D1 25%, D2 40%, D3 100%, Loss 100%. |
| **NPA Date** | The date when DPD first reaches 91 (for banks/NBFCs post-harmonization). This becomes the NPA recognition date. |
| **NPA Upgrade** | An NPA account returning to Standard status. Requires: entire arrears cleared (principal + interest + penal charges) AND account remains current for one full quarter. |
| **SMA Reporting** | Special Mention Account reporting to CRILC. SMA-1 and SMA-2 must be reported to RBI weekly (as per revised CRILC framework). |
| **Technical Write-off** | NPA removed from balance sheet but recovery efforts continue. Tracking in memorandum account. |
| **Wilful Defaulter** | Borrower who has capacity to pay but deliberately defaults. Identified per RBI Master Circular on Wilful Defaulters dated July 2024. |

### 8.3 Aggregates

#### 8.3.1 Aggregate: DelinquencyProfile

| Aspect | Details |
|---|---|
| Aggregate Root | `DelinquencyProfile` |
| Identity | `DelinquencyProfileId` (derived from LoanAccountId) |
| Description | The master delinquency record for a loan account. Tracks current DPD, bucket, asset classification, provisioning, and history. |

**Entities:**
- `DelinquencyProfile` — Root. Contains loanAccountId, currentDPD, currentBucket, assetClassification, provisioningRate, npaDate, lastPaymentDate, oldestUnpaidEMI.
- `BucketMovementHistory` — Ordered list of all bucket transitions for audit trail.
- `ProvisioningHistory` — History of provisioning changes.

**Value Objects:**
- `DPDBucket` — (bucketCode, minDPD, maxDPD, label, assetClass, provisioningRate)
- `AssetClassification` — Enum: STANDARD, SUB_STANDARD, DOUBTFUL_D1, DOUBTFUL_D2, DOUBTFUL_D3, LOSS
- `BucketTransition` — (fromBucket, toBucket, transitionDate, triggerEvent, direction [UPGRADE/DOWNGRADE])
- `ProvisionAmount` — (amount, rate, computationDate, method)

**Invariants:**
- DPD must be recalculated daily for every active loan account
- DPD cannot be negative
- Bucket must always be consistent with DPD (e.g., DPD=45 must be in SMA-1 bucket)
- NPA date, once set, cannot be backdated (except for regulatory corrections with auditor approval)
- Asset classification must follow RBI IRAC progression (cannot skip from Standard to Doubtful)
- Provisioning rate must comply with RBI minimum norms (institution can provision higher, not lower)

#### 8.3.2 Aggregate: CollectionCase

| Aspect | Details |
|---|---|
| Aggregate Root | `CollectionCase` |
| Identity | `CollectionCaseId` (UUID) |
| Description | Represents an active collection effort for a delinquent account. Created when DPD > 0. Closed when DPD returns to 0 and remains at 0. |

**Entities:**
- `CollectionCase` — Root. Contains case status, assigned strategy, priority, resolution target date.
- `CaseActivity` — Individual collection actions taken (call, SMS, visit, legal notice, etc.).
- `PTPromise` — Promise to Pay records with expected date and amount.
- `DisputeRecord` — Customer-raised disputes linked to the case.

**Value Objects:**
- `CaseStatus` — Enum: OPEN, IN_PROGRESS, PTP_RECEIVED, ESCALATED, LEGAL_REFERRED, SETTLED, CLOSED, WRITTEN_OFF
- `CasePriority` — Enum: LOW, MEDIUM, HIGH, CRITICAL (derived from DPD, amount, product type)
- `Disposition` — (dispositionCode, notes, capturedBy, capturedAt, channel)
- `PTPromiseStatus` — Enum: PENDING, KEPT, BROKEN, PARTIALLY_KEPT, EXPIRED

**Invariants:**
- Only one active CollectionCase per loan account at any time
- PTP date must be in the future when captured
- Case cannot be CLOSED while DPD > 0 (unless settled/written off)
- All case activities must have a valid disposition

### 8.4 Domain Events

| Event ID | Event Name | Published When | Payload | Consumed By |
|---|---|---|---|---|
| DE-DEL-001 | `DPDUpdated` | Daily DPD recalculation changes account's DPD | loanAccountId, previousDPD, newDPD, oldestUnpaidEMIDate | Workflow (BC-03), Analytics (BC-11) |
| DE-DEL-002 | `BucketChanged` | Account moves from one bucket to another | loanAccountId, fromBucket, toBucket, direction, dpd, transitionDate | Workflow (BC-03), Agency (BC-07), Compliance (BC-10), Analytics (BC-11) |
| DE-DEL-003 | `NPAClassified` | Account crosses 90 DPD and classified as NPA | loanAccountId, npaDate, outstandingAmount, productType, assetClass | Workflow (BC-03), Legal (BC-08), Compliance (BC-10) |
| DE-DEL-004 | `NPAUpgraded` | NPA account returns to Standard after full arrears clearance | loanAccountId, upgradeDate, previousAssetClass | Compliance (BC-10), Analytics (BC-11) |
| DE-DEL-005 | `SMAReportTriggered` | Account reaches SMA-1 or SMA-2, triggering CRILC reporting | loanAccountId, smaCategory, outstandingAmount, reportDate | Compliance (BC-10) |
| DE-DEL-006 | `ProvisioningChanged` | Asset classification change triggers provisioning change | loanAccountId, previousRate, newRate, provisionAmount | Analytics (BC-11) |
| DE-DEL-007 | `CollectionCaseOpened` | New collection case created when DPD > 0 | caseId, loanAccountId, dpd, bucket, priority | Workflow (BC-03), Agency (BC-07) |
| DE-DEL-008 | `CollectionCaseClosed` | Case closed — account returned to current or settled | caseId, loanAccountId, resolutionType, closureDate | Analytics (BC-11) |
| DE-DEL-009 | `PTPromiseMade` | Customer makes a Promise to Pay | caseId, loanAccountId, promisedAmount, promisedDate, channel | Workflow (BC-03), Agency (BC-07) |
| DE-DEL-010 | `PTPromiseBroken` | PTP date passed without payment | caseId, loanAccountId, promisedAmount, promisedDate | Workflow (BC-03), Sentiment (BC-05) |
| DE-DEL-011 | `WriteOffInitiated` | Account recommended for technical/prudential write-off | loanAccountId, writeOffType, writeOffAmount, recommendedBy | Legal (BC-08), Compliance (BC-10) |
| DE-DEL-012 | `WilfulDefaulterIdentified` | Account flagged as wilful defaulter per RBI norms | loanAccountId, customerId, criteria, identifiedDate | Legal (BC-08), Compliance (BC-10) |

### 8.5 Domain Services

#### 8.5.1 DPDCalculationService

**Purpose:** Calculates DPD for every active loan account daily (and on-demand when payment events occur).

**Algorithm:**

```
DPD Calculation Logic:
─────────────────────
Input: LoanAccount, EMISchedule, PaymentHistory

1. Identify ALL unpaid or partially-paid installments
   - An EMI is "unpaid" if total payments allocated to it < EMI amount
   - A "partially-paid" EMI is treated as unpaid for DPD purposes

2. Find the OLDEST unpaid installment due date
   oldest_unpaid_date = MIN(due_date) of all unpaid EMIs

3. Calculate DPD
   IF no unpaid EMIs exist:
       DPD = 0
   ELSE:
       DPD = today - oldest_unpaid_date (in calendar days)
       DPD = MAX(DPD, 0)  // Cannot be negative

4. Special Cases:
   a. Moratorium: If EMI due date falls within approved moratorium period,
      exclude that EMI from DPD calculation
   b. Restructured loans: Use revised EMI schedule for DPD calculation
      (DPD resets on restructuring implementation date per RBI norms)
   c. Agricultural loans: NPA at 2 harvest seasons (or 2 half-years)
      for short-duration crops; specific crop cycle for long-duration crops
   d. Part payment: If part of EMI paid, remaining amount still makes EMI
      "unpaid" for DPD purposes
```

**Sample DPD Calculation:**

```
Loan: ACC/2024/PL/001234
EMI Amount: ₹15,000 (due on 5th of every month)

EMI Schedule:
  EMI #12: Due 05-Jan-2026 → Paid ₹15,000 on 07-Jan-2026 ✓
  EMI #13: Due 05-Feb-2026 → Paid ₹8,000 on 10-Feb-2026 (partial) ✗
  EMI #14: Due 05-Mar-2026 → Not paid ✗

Date of calculation: 11-Mar-2026

Oldest unpaid EMI: EMI #13 (due 05-Feb-2026) — partial payment doesn't clear it
DPD = 11-Mar-2026 minus 05-Feb-2026 = 34 days
Bucket = SMA-1 (31-60 DPD)
```

#### 8.5.2 BucketClassificationService

**Purpose:** Maps DPD to regulatory buckets and manages bucket transitions.

**Standard Bucket Configuration (RBI IRAC Norms):**

| Bucket Code | Label | DPD Range | Asset Classification | Provisioning Rate (Secured) | Provisioning Rate (Unsecured) |
|---|---|---|---|---|---|
| CURRENT | Current | 0 | Standard | 0.40% | 0.40% |
| SMA-0 | Special Mention 0 | 1-30 | Standard | 0.40% | 0.40% |
| SMA-1 | Special Mention 1 | 31-60 | Standard | 0.40% | 0.40% |
| SMA-2 | Special Mention 2 | 61-90 | Standard | 0.40% | 0.40% |
| NPA-SS | Sub-Standard | 91-365 | Sub-Standard | 15% | 25% |
| NPA-D1 | Doubtful ≤1 year | 366-730 | Doubtful D1 | 25% of unsecured + 25% of secured | 100% |
| NPA-D2 | Doubtful 1-3 years | 731-1095 | Doubtful D2 | 40% of unsecured + 40% of secured | 100% |
| NPA-D3 | Doubtful >3 years | 1096+ | Doubtful D3 | 100% | 100% |
| LOSS | Loss | Any (identified) | Loss | 100% | 100% |

**NPA Upgrade Rules (per RBI IRAC norms):**
- Entire arrears of interest and principal must be paid (not just current EMI)
- For accounts classified as NPA, upgrade to Standard requires:
  - Payment of all overdue amounts
  - Account should remain "standard" for a minimum of one quarter (observation period)
  - For restructured NPA accounts: satisfactory performance under revised terms for one year

#### 8.5.3 CRILCReportingService

**Purpose:** Generates CRILC (Central Repository of Information on Large Credits) reports as per RBI requirements.

**Reporting Requirements:**

| Report Type | Frequency | Threshold | Content |
|---|---|---|---|
| Large Credit Exposure | Quarterly | ≥ ₹5 crore aggregate exposure | All borrowers with exposure ≥ ₹5 crore |
| SMA-1 Reporting | Weekly (every Friday) | ≥ ₹5 crore | Accounts moving to SMA-1 |
| SMA-2 Reporting | Weekly (every Friday) | ≥ ₹5 crore | Accounts moving to SMA-2 |
| NPA Reporting | Monthly | All NPAs | All NPA accounts regardless of amount |
| Wilful Defaulter | As identified | All | Per RBI Master Circular |

### 8.6 Application Services (Use Cases)

| Use Case ID | Use Case Name | Actor | Description |
|---|---|---|---|
| COL-DEL-UC-001 | Run Daily DPD Calculation | System (EOD) | Calculate DPD for all active loan accounts. Run after all reconciliation batches are processed. |
| COL-DEL-UC-002 | On-Demand DPD Recalculation | System (Event) | Recalculate DPD for specific account when PaymentCredited or BounceConfirmed event received. |
| COL-DEL-UC-003 | Process Bucket Movements | System | After DPD calculation, identify all accounts that changed buckets. Publish BucketChanged events. |
| COL-DEL-UC-004 | Classify NPA | System | Identify accounts crossing 91 DPD. Mark NPA with date. Reverse unrealized interest income. Publish NPAClassified. |
| COL-DEL-UC-005 | Process NPA Upgrade | System (Event) | When payment clears all arrears for an NPA account, initiate upgrade process (observation period). |
| COL-DEL-UC-006 | Generate CRILC Report | System (Scheduled) | Generate weekly SMA report and quarterly large credit report per RBI format. |
| COL-DEL-UC-007 | Open Collection Case | System | When DPD > 0 and no active case exists, create CollectionCase with priority based on DPD/amount/product. |
| COL-DEL-UC-008 | Record PTP | Agent / System | Capture Promise to Pay from customer interaction. Set follow-up workflow. |
| COL-DEL-UC-009 | Process Broken PTP | System (Scheduled) | Daily check for PTPs where date has passed without corresponding payment. Escalate. |
| COL-DEL-UC-010 | Initiate Write-off | Risk User | Recommend account for write-off. Requires multi-level approval (committee). |
| COL-DEL-UC-011 | Generate Provisioning Report | System (Month-end) | Calculate provision requirements per asset classification. Report for finance team. |
| COL-DEL-UC-012 | Handle Restructured Account DPD | System | Apply restructuring terms to DPD calculation. Reset DPD per RBI restructuring norms. |

### 8.7 Functional Requirements

#### 8.7.1 DPD Calculation Engine

| Req ID | Requirement | Priority | Acceptance Criteria |
|---|---|---|---|
| COL-DEL-001 | System shall calculate DPD for ALL active loan accounts daily as part of EOD processing. | P0 | DPD calculated for 100% of active accounts; completed within EOD batch window (max 2 hours for 10M accounts) |
| COL-DEL-002 | System shall recalculate DPD on-demand for individual accounts when PaymentCredited or BounceConfirmed events are received (near-real-time). | P0 | DPD recalculated within 30 seconds of event |
| COL-DEL-003 | DPD calculation must consider partial payments — an EMI is "unpaid" until full EMI amount is received (or exceeded). | P0 | Partial payment of ₹8,000 against ₹15,000 EMI → EMI remains unpaid |
| COL-DEL-004 | DPD calculation must handle moratorium periods — EMIs falling within approved moratorium are excluded. | P0 | Moratorium EMIs excluded; DPD counts from first non-moratorium unpaid EMI |
| COL-DEL-005 | DPD calculation must handle restructured loans — DPD resets to 0 on restructuring implementation date, follows revised schedule thereafter. | P0 | DPD reset on restructuring; new schedule used for subsequent calculation |
| COL-DEL-006 | System shall handle agricultural loan NPA norms — NPA based on crop season (2 crop seasons or 2 half-years for short-duration; crop cycle for long-duration). | P1 | Agricultural NPA per RBI crop-cycle norms |
| COL-DEL-007 | System shall handle bullet repayment loans — DPD counted from maturity date for principal, from due date for interest. | P1 | Bullet loan DPD correctly computed |
| COL-DEL-008 | System shall handle overdraft/cash credit facilities — NPA if out-of-order for >90 days (credits < interest debited, or no credits for 90 days, or outstanding > sanctioned limit continuously for 90 days). | P1 | OD/CC NPA per RBI "out of order" norms |

#### 8.7.2 Bucket Management

| Req ID | Requirement | Priority | Acceptance Criteria |
|---|---|---|---|
| COL-DEL-009 | System shall classify every account into a DPD bucket based on calculated DPD, per configurable bucket definitions. | P0 | Every account has exactly one bucket at all times |
| COL-DEL-010 | System shall detect and record all bucket movements (upgrades and downgrades) with timestamp and trigger event. | P0 | Complete bucket movement audit trail |
| COL-DEL-011 | System shall support configurable bucket definitions per product type and entity type (to accommodate different regulatory norms). | P0 | Bucket config via admin UI; product-specific rules supported |
| COL-DEL-012 | System shall publish `BucketChanged` event within 1 minute of detecting a bucket transition. | P0 | Event published with <1 min latency |
| COL-DEL-013 | System shall provide real-time portfolio bucket distribution dashboard (count and amount per bucket, per product, per branch, per region). | P0 | Dashboard with <5 minute data lag |
| COL-DEL-014 | System shall compute roll-forward and roll-backward rates (month-over-month, quarter-over-quarter) per bucket. | P0 | Roll rates available in monthly MIS |
| COL-DEL-015 | System shall generate bucket-wise aging report with granular drill-down (to individual accounts). | P0 | Report on-demand and scheduled |

#### 8.7.3 NPA Classification & Provisioning

| Req ID | Requirement | Priority | Acceptance Criteria |
|---|---|---|---|
| COL-DEL-016 | System shall automatically classify account as NPA when DPD reaches 91 (configurable per entity/product type). | P0 | NPA classified on exact date; no lag |
| COL-DEL-017 | On NPA classification, system shall reverse unrealized interest income per RBI IRAC norms (interest recognized on accrual basis but not received must be reversed). | P0 | Interest income reversed from P&L on NPA date |
| COL-DEL-018 | System shall compute provisioning amount based on asset classification and RBI minimum provisioning norms. | P0 | Provisioning computed correctly per classification |
| COL-DEL-019 | For NPA upgrade, system shall enforce observation period (minimum 1 quarter of standard performance after arrears clearance) before upgrading asset classification. | P0 | Observation period tracked; upgrade only after clean quarter |
| COL-DEL-020 | System shall generate NPA movement report (additions, reductions, upgrades, write-offs) — monthly for management, quarterly for board. | P0 | NPA movement report per RBI format |
| COL-DEL-021 | System shall support technical write-off (off-balance-sheet) with continued tracking in memorandum accounts. | P0 | Write-off entries correct; memorandum tracking active; recovery posted correctly |
| COL-DEL-022 | System shall identify potential wilful defaulters based on configurable criteria (has capacity to pay from bureau/financial data, no response to notices, diversion of funds). | P1 | Candidates flagged for committee review |
| COL-DEL-023 | System shall compute ECL (Expected Credit Loss) provisions per IND AS 109 staging (Stage 1, 2, 3) in addition to IRAC provisions. | P0 | ECL computed; IND AS staging aligned with IRAC |

#### 8.7.4 Collection Case Management

| Req ID | Requirement | Priority | Acceptance Criteria |
|---|---|---|---|
| COL-DEL-024 | System shall auto-create a Collection Case when any account's DPD crosses configurable threshold (default: DPD > 0). | P0 | Case created with priority; linked to account |
| COL-DEL-025 | System shall auto-assign case priority based on configurable rules (DPD, outstanding amount, product type, customer segment, vintage). | P0 | Priority correctly assigned; recalculated on DPD change |
| COL-DEL-026 | System shall track all activities (calls, messages, visits, notices) against the collection case with dispositions. | P0 | Every activity logged with disposition, timestamp, agent |
| COL-DEL-027 | System shall manage PTP lifecycle (capture → track → kept/broken → escalate if broken). | P0 | PTP adherence tracked; broken PTPs auto-escalated |
| COL-DEL-028 | System shall support dispute management within collection cases (customer disputes amount, charges, etc.). | P1 | Dispute recorded; collection actions paused if disputed |
| COL-DEL-029 | System shall auto-close collection case when DPD returns to 0 and remains at 0 for configurable period (default: 7 days). | P0 | Case closed; resolution type recorded |

### 8.8 Integration Events Consumed

| Event | From Context | Action Taken |
|---|---|---|
| `PaymentCredited` | Reconciliation (BC-01) | Recalculate DPD; check bucket change; update collection case; check PTP fulfillment |
| `BounceConfirmed` | Reconciliation (BC-01) | Confirm non-payment; DPD continues; update collection case; trigger bounce-specific workflow |
| `SuspenseCleared` | Reconciliation (BC-01) | If cleared to this account, recalculate DPD |
| `PaymentReversed` | Reconciliation (BC-01) | Re-instate unpaid EMI; recalculate DPD; potential bucket downgrade |

---

## END OF PART 03

Next: **COL-BRD-Part04-Workflow-Communication.md** -- Workflow & Campaign + Communication Bounded Contexts.
