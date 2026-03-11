# PART 07 -- AGENCY & FIELD + LEGAL & RECOVERY BOUNDED CONTEXTS (BC-07 & BC-08)

---

## 13. AGENCY & FIELD CONTEXT (BC-07)

### 13.1 Context Overview

| Attribute | Details |
|---|---|
| Context Name | Agency & Field Context |
| Context ID | BC-07 |
| Domain Classification | Core Domain |
| Owner Team | Agency Management / Field Operations |
| Upstream Dependencies | Delinquency (BC-02) — CollectionCaseOpened, BucketChanged; Workflow (BC-03) — allocation triggers; Sentiment (BC-05) — RiskCategoryUpdated; Account (BC-06) — account/customer data |
| Downstream Consumers | Legal (BC-08), Communication (BC-04), Compliance (BC-10), Analytics (BC-11) |
| Primary Objective | Manage end-to-end lifecycle of collection agency relationships, account allocation, field visit operations, and agency performance monitoring |

### 13.2 Aggregates

#### 13.2.1 Aggregate: CollectionAgency

| Aspect | Details |
|---|---|
| Aggregate Root | `CollectionAgency` |
| Identity | `AgencyId` (UUID) |
| Description | An external collection agency empanelled for recovery operations. |

**Entities:**
- `CollectionAgency` — Root. Name, registration details, empanelment status, contract terms, geographic coverage, product specialization.
- `AgencyContract` — Contract terms including commission rates, SLA targets, compliance requirements, tenure.
- `AgencyPerformanceRecord` — Monthly performance snapshots.
- `AgencyComplianceRecord` — Compliance audit findings and corrective actions.
- `AgencyUser` — Users from agency who access the system (with role-based access).

**Value Objects:**
- `EmpanelmentStatus` — Enum: PROSPECTIVE, UNDER_EVALUATION, EMPANELLED, ACTIVE, SUSPENDED, BLACKLISTED, TERMINATED
- `CommissionStructure` — (commissionType [FLAT/PERCENTAGE/SLAB], rates per bucket/product, paymentTerms, incentiveRules)
- `GeographicCoverage` — (states, cities, pincodes)
- `PerformanceScore` — (recoveryRate, ptpConversionRate, contactRate, costPerRecovery, complianceScore, overallRating)
- `SLATargets` — (contactWithin48Hours, ptpConversionTarget, recoveryRateTarget, complianceAuditPassRate)

#### 13.2.2 Aggregate: AccountAllocation

| Aspect | Details |
|---|---|
| Aggregate Root | `AccountAllocation` |
| Identity | `AllocationId` (UUID) |
| Description | Assignment of a delinquent loan account to a specific collection agency or internal team for recovery. |

**Entities:**
- `AccountAllocation` — Root. Links loan account to agency/team with allocation date, target, status.
- `AllocationActivity` — Activities performed by the allocated agency against this account.

**Value Objects:**
- `AllocationStatus` — Enum: ALLOCATED, ACTIVE, RECALLED, TRANSFERRED, RESOLVED, EXPIRED
- `AllocationTarget` — (targetAmount, targetDate, minimumContactAttempts)
- `RecallReason` — Enum: PERFORMANCE_BELOW_SLA, COMPLIANCE_VIOLATION, CUSTOMER_COMPLAINT, STRATEGY_CHANGE, LEGAL_REFERRAL, ACCOUNT_RESOLVED

**Invariants:**
- An account can be allocated to at most one agency at a time per collection level (tele-calling vs field)
- Recalled accounts must have a 24-hour cooling period before reallocation
- Agency cannot access account data after recall
- Allocation must respect agency's geographic coverage and product specialization

#### 13.2.3 Aggregate: FieldVisit

| Aspect | Details |
|---|---|
| Aggregate Root | `FieldVisit` |
| Identity | `FieldVisitId` (UUID) |
| Description | A physical visit to the borrower's address by a field collection agent. |

**Entities:**
- `FieldVisit` — Root. Contains scheduled date, agent, address, purpose, status, outcome.
- `VisitEvidence` — Photos, videos, GPS coordinates, customer signatures captured during visit.

**Value Objects:**
- `VisitStatus` — Enum: SCHEDULED, EN_ROUTE, AT_LOCATION, COMPLETED, CUSTOMER_NOT_FOUND, ADDRESS_INVALID, RESCHEDULED, CANCELLED
- `VisitOutcome` — (disposition, ptpCaptured, paymentCollected, evidenceCollected, addressVerified, customerMet)
- `GPSCoordinate` — (latitude, longitude, accuracy, timestamp)
- `VisitEvidence` — (evidenceType [PHOTO/VIDEO/DOCUMENT/SIGNATURE], fileReference, capturedAt, gpsLocation)

### 13.3 Domain Events

| Event ID | Event Name | Payload | Consumed By |
|---|---|---|---|
| DE-AG-001 | `AccountAllocated` | allocationId, loanAccountId, agencyId, bucket, outstandingAmount | Account (BC-06), Communication (BC-04) |
| DE-AG-002 | `AccountRecalled` | allocationId, loanAccountId, agencyId, recallReason | Account (BC-06) |
| DE-AG-003 | `FieldVisitScheduled` | visitId, loanAccountId, agentId, scheduledDate, address | Workflow (BC-03) |
| DE-AG-004 | `FieldVisitCompleted` | visitId, loanAccountId, outcome, disposition, ptpCaptured, evidenceCount | Delinquency (BC-02), Account (BC-06), Sentiment (BC-05) |
| DE-AG-005 | `AgencyPerformanceEvaluated` | agencyId, period, performanceScore, rankAmongAgencies | Analytics (BC-11) |
| DE-AG-006 | `AgencyComplianceViolation` | agencyId, violationType, severity, accountsAffected | Compliance (BC-10) |
| DE-AG-007 | `LegalReferralRequested` | loanAccountId, agencyId, referralReason, outstandingAmount | Legal (BC-08) |
| DE-AG-008 | `CashCollected` | visitId, loanAccountId, amount, receiptNumber | Reconciliation (BC-01) |
| DE-AG-009 | `AddressFound` | loanAccountId, newAddress, verificationMethod | Account (BC-06) |

### 13.4 Domain Services

#### 13.4.1 AllocationEngineService

**Purpose:** Optimally allocates delinquent accounts to collection agencies based on configurable rules.

**Allocation Algorithm:**

```
Input: Pool of unallocated delinquent accounts + Pool of active agencies

Step 1: ELIGIBILITY FILTER
  - Filter agencies by: empanelment status = ACTIVE
  - Filter by geographic match (account's state/city within agency coverage)
  - Filter by product match (account's product type within agency specialization)
  - Filter by bucket match (account's DPD bucket within agency's contracted buckets)
  - Filter by capacity (agency hasn't exceeded max allocation limit)

Step 2: SCORING
  For each eligible agency, compute allocation score:
    Score = w1 × recovery_rate_score      (historical recovery rate for similar accounts)
          + w2 × contact_rate_score        (historical contact success rate)
          + w3 × compliance_score          (audit compliance rating)
          + w4 × cost_score                (inverse of commission rate)
          + w5 × capacity_score            (available capacity vs limit)
          + w6 × specialization_bonus      (agency specializes in this product/bucket)

  Default weights: w1=0.30, w2=0.15, w3=0.20, w4=0.15, w5=0.10, w6=0.10

Step 3: ALLOCATION
  - Sort agencies by score (descending)
  - Allocate accounts to highest-scoring agency up to capacity
  - Distribute remaining to next-best agencies
  - Ensure minimum allocation per agency (contractual minimum)
  - Round-robin for equal scores

Step 4: REBALANCING (Monthly)
  - Review allocation vs performance
  - Recall underperforming allocations
  - Reallocate to better-performing agencies
```

#### 13.4.2 FieldVisitPlanningService

**Purpose:** Optimizes field visit scheduling and routing.

**Capabilities:**
- Beat planning: Group accounts by geography for efficient route coverage
- Route optimization: TSP (Traveling Salesman Problem) approximation for daily visit routes
- Priority scoring: High-value overdue + high propensity to pay = highest visit priority
- Workload balancing: Even distribution across field agents
- Timing optimization: Visit during hours when customer is most likely home/at office
- Safety alerts: Flag high-risk areas; require buddy system for large outstanding amounts

### 13.5 Functional Requirements

#### 13.5.1 Agency Management

| Req ID | Requirement | Priority | Acceptance Criteria |
|---|---|---|---|
| COL-AG-001 | System shall support agency onboarding with: registration details, contract terms, commission structure, geographic coverage, product specialization, SLA targets, user provisioning. | P0 | Complete agency profile captured; users provisioned with role-based access |
| COL-AG-002 | System shall support agency empanelment lifecycle: prospective → evaluation → empanelled → active → suspended/terminated. With maker-checker for status changes. | P0 | Lifecycle managed; state transitions audited |
| COL-AG-003 | System shall compute agency performance scorecard monthly: recovery rate, PTP conversion rate, contact rate, cost per recovery, compliance score, overall rating. | P0 | Scorecard generated by 5th of each month; agency-wise ranking |
| COL-AG-004 | System shall support commission calculation and settlement per agency contract terms (flat, percentage, slab-based; per bucket/product). | P0 | Commission calculated accurately; settlement report generated |
| COL-AG-005 | System shall monitor agency compliance: call recording audits (random sample), fair practices adherence, customer complaint correlation, visit verification (GPS). | P0 | Random audit sample ≥5% of interactions; violations flagged |
| COL-AG-006 | System shall support agency suspension with immediate access revocation and account reallocation. | P0 | Suspension effective within 1 hour; accounts recalled |
| COL-AG-007 | System shall provide agency portal with: allocated accounts, customer data (role-masked), disposition capture, PTP recording, payment recording, visit scheduling, performance dashboard. | P0 | Agency portal functional; data segregated per agency |
| COL-AG-008 | System shall support multi-tier agency model (national agencies for high-value; regional for mid; local for small-ticket and field-intensive). | P1 | Allocation rules support agency tier |

#### 13.5.2 Account Allocation

| Req ID | Requirement | Priority | Acceptance Criteria |
|---|---|---|---|
| COL-AG-009 | System shall auto-allocate delinquent accounts to agencies based on configurable allocation rules (geography, product, bucket, agency score). | P0 | Auto-allocation runs daily (or on-demand); 100% eligible accounts allocated |
| COL-AG-010 | System shall support manual allocation/reallocation by collection supervisors with justification. | P0 | Manual allocation with audit trail |
| COL-AG-011 | System shall support account recall from agency (poor performance, compliance violation, customer complaint, strategy change). | P0 | Recall executed; agency access revoked within 4 hours |
| COL-AG-012 | System shall support account transfer between agencies with full history preservation. | P0 | History preserved; new agency can see previous interaction summary |
| COL-AG-013 | System shall auto-recall accounts when: payment clears all dues, account moves to legal referral, customer complaint received about agency. | P0 | Auto-recall triggered by events |
| COL-AG-014 | System shall enforce allocation caps per agency (maximum accounts, maximum exposure). | P0 | Caps enforced; overflow to next-best agency |

#### 13.5.3 Field Collection & Visit Management

| Req ID | Requirement | Priority | Acceptance Criteria |
|---|---|---|---|
| COL-AG-015 | System shall provide mobile app for field agents with: daily visit list, customer details, route map, disposition capture, GPS check-in/out, photo/video capture, digital receipt, offline mode. | P0 | App functional on Android (min) and iOS; works offline with sync |
| COL-AG-016 | System shall support beat planning: group accounts by pincode/area, assign to field agents, optimize routes. | P0 | Beats planned daily; route optimization reduces travel by ≥20% vs manual |
| COL-AG-017 | System shall track field visits via GPS: check-in at customer location, time spent, route taken. | P0 | GPS coordinates captured at check-in/out; deviation from planned route flagged |
| COL-AG-018 | System shall support cash collection during field visits with: amount entry, digital receipt generation (SMS + PDF), immediate reconciliation trigger. | P0 | Cash recorded; receipt sent to customer within 5 minutes; appears in reconciliation |
| COL-AG-019 | System shall capture visit evidence: photos of property, customer meeting photo (with consent), document photos. Evidence tagged with GPS and timestamp. | P0 | Evidence captured; tamper-proof (metadata verification) |
| COL-AG-020 | System shall capture customer e-signature for PTP commitments during field visits. | P1 | Digital signature captured; linked to PTP record |
| COL-AG-021 | System shall support safety features: panic button, live location sharing, supervisor alerts for high-risk visits. | P1 | Safety features functional; panic alert reaches supervisor within 30 seconds |
| COL-AG-022 | System shall generate field visit report automatically from captured data (disposition, evidence, PTP, address verification) and post to collection case. | P0 | Report auto-generated; posted to case within 5 minutes of visit completion |

---

## 14. LEGAL & RECOVERY CONTEXT (BC-08)

### 14.1 Context Overview

| Attribute | Details |
|---|---|
| Context Name | Legal & Recovery Context |
| Context ID | BC-08 |
| Domain Classification | Core Domain |
| Owner Team | Legal & Recovery |
| Upstream Dependencies | Delinquency (BC-02) — NPAClassified; Agency (BC-07) — LegalReferralRequested; Workflow (BC-03) — legal workflow triggers |
| Downstream Consumers | Account (BC-06), Compliance (BC-10), Analytics (BC-11) |
| Primary Objective | Manage all legal proceedings, SARFAESI actions, settlements (OTS), write-offs, and post-write-off recovery |

### 14.2 Aggregates

#### 14.2.1 Aggregate: LegalCase

| Aspect | Details |
|---|---|
| Aggregate Root | `LegalCase` |
| Identity | `LegalCaseId` (UUID) |
| Description | A legal action initiated against a borrower for loan recovery. |

**Entities:**
- `LegalCase` — Root. Case type, court/forum, filing date, status, assigned advocate, next hearing.
- `LegalNotice` — Notices served (demand notice, SARFAESI 13(2), arbitration, civil suit, recovery notice).
- `Hearing` — Court hearing records with date, court, judge, outcome, next date.
- `CourtOrder` — Orders received (interim, final, execution decree).

**Value Objects:**
- `CaseType` — Enum: SARFAESI, DRT (Debt Recovery Tribunal), CIVIL_SUIT, ARBITRATION, LOK_ADALAT, SECTION_138_CHEQUE_BOUNCE, INSOLVENCY (IBC), CRIMINAL
- `CaseStatus` — Enum: NOTICE_ISSUED, NOTICE_SERVED, OBJECTION_RECEIVED, CASE_FILED, HEARING_IN_PROGRESS, ORDER_RECEIVED, EXECUTION_IN_PROGRESS, SETTLED, DISPOSED, CLOSED
- `NoticeType` — Enum: DEMAND_NOTICE, SARFAESI_13_2, SARFAESI_13_4, ARBITRATION_NOTICE, CIVIL_SUIT_NOTICE, SECTION_138_NOTICE, LEGAL_HEIR_NOTICE
- `HearingOutcome` — (outcome [ADJOURNED/ARGUED/ORDER_RESERVED/ORDER_PRONOUNCED/SETTLED], nextDate, notes)

#### 14.2.2 Aggregate: SARFAESIAction

| Aspect | Details |
|---|---|
| Aggregate Root | `SARFAESIAction` |
| Identity | `SARFAESIActionId` (UUID) |
| Description | Tracks the SARFAESI process workflow from Section 13(2) notice through possession and auction. |

**Entities:**
- `SARFAESIAction` — Root. Links to loan account and collateral. Tracks SARFAESI stage.
- `Section13_2Notice` — Notice details, service date, borrower response, objection handling.
- `PossessionAction` — Physical/symbolic possession details (Section 13(4)).
- `AuctionRecord` — Auction notice, e-auction details, bidders, reserve price, sale price.

**Value Objects:**
- `SARFAESIStage` — Enum: NOTICE_13_2_ISSUED, NOTICE_13_2_SERVED, OBJECTION_PERIOD (60 days), OBJECTION_RECEIVED, OBJECTION_REJECTED, POSSESSION_NOTICE_13_4, SYMBOLIC_POSSESSION, PHYSICAL_POSSESSION, AUCTION_NOTICE_PUBLISHED, E_AUCTION_CONDUCTED, SALE_CONFIRMED, SALE_CERTIFICATE_ISSUED, TITLE_TRANSFERRED
- `AuctionDetails` — (reservePrice, auctionDate, platform, totalBidders, highestBid, winningBidder, salePrice, saleCertificateDate)
- `CollateralDetails` — (type, description, address, currentValuation, lastValuationDate, mortgageType, CERSAI_chargeId)

**Invariants:**
- Section 13(2) notice must allow 60 days for borrower response before further action
- SARFAESI not applicable for loans <₹1 lakh or where collateral value < ₹1 lakh (Section 13(1) threshold)
- SARFAESI not applicable for agricultural land
- Auction reserve price must be ≥ assessed value per SARFAESI rules
- If first auction fails, second auction reserve price can be reduced by up to 25%

#### 14.2.3 Aggregate: Settlement

| Aspect | Details |
|---|---|
| Aggregate Root | `Settlement` |
| Identity | `SettlementId` (UUID) |
| Description | One-time settlement (OTS) negotiation and execution. |

**Entities:**
- `Settlement` — Root. Contains settlement type, proposed terms, approval workflow, execution status.
- `SettlementNegotiation` — History of offers and counter-offers.
- `SettlementPayment` — Payment schedule for settlement amount (lump sum or installments).

**Value Objects:**
- `SettlementType` — Enum: OTS_FULL (one-time lump sum), OTS_INSTALLMENT (settlement in parts), TECHNICAL_WRITE_OFF, COMPROMISE_SETTLEMENT, INSURANCE_CLAIM
- `SettlementTerms` — (totalOutstanding, settlementAmount, haircut percentage, paymentSchedule, conditions, expiryDate)
- `ApprovalStatus` — Enum: PROPOSED, UNDER_REVIEW, L1_APPROVED, L2_APPROVED, COMMITTEE_APPROVED, BOARD_APPROVED, REJECTED, EXPIRED
- `SettlementExecution` — (amountPaid, amountPending, paymentsDone, paymentsRemaining, isCompleted)

**Invariants:**
- Settlement must follow board-approved OTS policy (delegation of authority matrix)
- Haircut above threshold (e.g., >25%) requires committee/board approval
- Settlement offer has finite validity (configurable, default: 30 days)
- Write-off requires multi-level approval per RBI prudential norms
- All settlement decisions must be documented with justification (CAG/audit trail)

### 14.3 Functional Requirements

#### 14.3.1 Legal Case Management

| Req ID | Requirement | Priority | Acceptance Criteria |
|---|---|---|---|
| COL-LG-001 | System shall support legal case creation with case type, court/forum, parties, claim amount, assigned advocate, and linked loan account(s). | P0 | Case created with all mandatory fields |
| COL-LG-002 | System shall track case lifecycle from notice through filing, hearings, orders, and disposition with complete audit trail. | P0 | Every status change logged; timeline visible |
| COL-LG-003 | System shall generate legal notices from templates: demand notice, SARFAESI 13(2), SARFAESI 13(4), arbitration notice, Section 138 notice, civil suit notice. | P0 | Templates configurable; auto-populated with account/customer data; multi-language |
| COL-LG-004 | System shall track notice service: dispatch date, service method (registered post, courier, affixture, publication), service date, acknowledgement/receipt, proof of service. | P0 | Service tracking with digital proof storage |
| COL-LG-005 | System shall maintain hearing calendar with reminders (T-7, T-3, T-1 days) to assigned advocate and legal team. | P0 | Calendar integrated; reminders sent |
| COL-LG-006 | System shall track limitation periods and alert 90 days before limitation expires (Limitation Act, 1963: 3 years for recovery of money). | P0 | Limitation tracking; alert raised; no expired limitations |
| COL-LG-007 | System shall support advocate empanelment, case assignment, performance tracking (success rate, cost per case, timeline adherence), and fee management. | P1 | Advocate panel managed; performance visible |
| COL-LG-008 | System shall track legal costs per case (filing fees, advocate fees, valuation costs, publication costs, auction costs) and per loan account. | P0 | Cost tracking accurate; reports available |
| COL-LG-009 | System shall support Lok Adalat case management with: case identification, Lok Adalat scheduling, settlement recording, decree tracking. | P1 | Lok Adalat workflow supported |
| COL-LG-010 | System shall support DRT (Debt Recovery Tribunal) case management for exposures ≥₹20 lakh per Recovery of Debts Act, 1993. | P0 | DRT workflow supported |

#### 14.3.2 SARFAESI Process

| Req ID | Requirement | Priority | Acceptance Criteria |
|---|---|---|---|
| COL-LG-011 | System shall automate SARFAESI workflow: eligibility check → 13(2) notice generation → service tracking → 60-day waiting → objection handling → 13(4) possession → auction. | P0 | Full workflow automated with compliance checks at each stage |
| COL-LG-012 | System shall validate SARFAESI eligibility: loan classified as NPA, secured asset, outstanding ≥₹1 lakh, not agricultural land, lender is bank/NBFC with asset size ≥₹100 crore (for NBFCs). | P0 | Eligibility validated before notice generation |
| COL-LG-013 | System shall enforce 60-day response period after Section 13(2) notice service. No action permitted before period expires (unless borrower waives). | P0 | System blocks premature action; countdown visible |
| COL-LG-014 | System shall track borrower objections and institution's response (mandatory response within 15 days per RBI guidelines). | P0 | Objection logged; response deadline tracked |
| COL-LG-015 | System shall support e-auction process: reserve price setting, auction notice publication (2 newspapers including vernacular), e-auction platform integration, bid management, sale confirmation, sale certificate generation. | P0 | E-auction workflow supported end-to-end |
| COL-LG-016 | System shall manage post-auction process: sale certificate issuance, CERSAI charge satisfaction, title transfer support, surplus amount refund to borrower. | P0 | Post-auction tasks tracked to completion |

#### 14.3.3 Settlement (OTS) Management

| Req ID | Requirement | Priority | Acceptance Criteria |
|---|---|---|---|
| COL-LG-017 | System shall support OTS proposal creation with: total outstanding, proposed settlement amount, haircut calculation, payment terms, conditions, and justification. | P0 | Proposal created with full financial analysis |
| COL-LG-018 | System shall enforce delegation of authority matrix for settlement approvals (e.g., <₹10L branch manager, <₹50L zonal, <₹5Cr committee, >₹5Cr board). Configurable per institution. | P0 | DOA enforced; no bypass possible |
| COL-LG-019 | System shall track settlement payment execution (lump sum or installments) with auto-closure on full payment and auto-revival on default. | P0 | Payment tracking; auto-actions on completion/default |
| COL-LG-020 | System shall generate settlement letter, NOC, and CERSAI charge satisfaction filing on successful settlement completion. | P0 | Documents auto-generated on completion |

#### 14.3.4 Write-off & Post Write-off Recovery

| Req ID | Requirement | Priority | Acceptance Criteria |
|---|---|---|---|
| COL-LG-021 | System shall support technical write-off: remove from balance sheet but continue tracking in memorandum account. | P0 | Write-off GL entries correct; memorandum account active |
| COL-LG-022 | System shall continue collection efforts on written-off accounts with separate tracking and reporting. | P0 | Post-write-off recovery tracked; credited to P&L on receipt |
| COL-LG-023 | System shall generate write-off proposal with: account details, recovery efforts history, reason for write-off, expected recovery, tax implications. | P0 | Proposal with complete history |
| COL-LG-024 | System shall enforce write-off approval matrix (multi-level per RBI/board-approved policy). | P0 | Approval matrix enforced |
| COL-LG-025 | System shall report written-off accounts to credit bureaus with appropriate status (written-off/settled). | P0 | Bureau update within 30 days of write-off |

---

## END OF PART 07

Next: **COL-BRD-Part08-Payment-Compliance-Analytics.md** -- Payment Facilitation, Compliance & Audit, Analytics Bounded Contexts.
