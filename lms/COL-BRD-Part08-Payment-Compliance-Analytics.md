# PART 08 -- PAYMENT FACILITATION, COMPLIANCE & AUDIT, ANALYTICS BOUNDED CONTEXTS (BC-09, BC-10, BC-11)

---

## 15. PAYMENT FACILITATION CONTEXT (BC-09)

### 15.1 Context Overview

| Attribute | Details |
|---|---|
| Context Name | Payment Facilitation Context |
| Context ID | BC-09 |
| Domain Classification | Supporting Domain |
| Owner Team | Digital Payments |
| Upstream Dependencies | Account (BC-06) — outstanding data; Workflow (BC-03) — payment link triggers; Communication (BC-04) — delivery channel |
| Downstream Consumers | Reconciliation (BC-01), Delinquency (BC-02), Analytics (BC-11) |
| Primary Objective | Maximize self-service payment options for delinquent borrowers, reducing cost of collection and improving recovery rates |

### 15.2 Aggregates

#### 15.2.1 Aggregate: PaymentLink

| Aspect | Details |
|---|---|
| Aggregate Root | `PaymentLink` |
| Identity | `PaymentLinkId` (UUID) |
| Description | A time-bound, amount-specific payment link generated for a delinquent borrower. |

**Entities:**
- `PaymentLink` — Root. Contains loan account reference, amount (pre-filled or open), expiry, payment modes supported, status.
- `PaymentAttempt` — Each attempt made by customer via this link.

**Value Objects:**
- `LinkStatus` — Enum: ACTIVE, EXPIRED, USED, CANCELLED
- `PaymentMode` — Enum: UPI, NETBANKING, DEBIT_CARD, CREDIT_CARD, WALLET, NEFT
- `LinkConfig` — (preFilledAmount: boolean, allowPartial: boolean, expiryHours: int, supportedModes: List<PaymentMode>)
- `QRCode` — (qrData, upiDeepLink, imageUrl)

#### 15.2.2 Aggregate: RepaymentPlan

| Aspect | Details |
|---|---|
| Aggregate Root | `RepaymentPlan` |
| Identity | `RepaymentPlanId` (UUID) |
| Description | A negotiated plan to clear overdue amounts in installments (different from loan restructuring; this is a short-term collection arrangement). |

**Entities:**
- `RepaymentPlan` — Root. Total overdue, plan installments, approval status, compliance with plan.
- `PlanInstallment` — Individual installment with due date, amount, payment status.

**Value Objects:**
- `PlanStatus` — Enum: PROPOSED, CUSTOMER_ACCEPTED, APPROVED, ACTIVE, COMPLETED, DEFAULTED, CANCELLED
- `PlanType` — Enum: OVERDUE_SPLIT (split overdue into parts), CATCH_UP (pay extra per month), BALLOON (small installments + final lump sum)

#### 15.2.3 Aggregate: MandateRepresentation

| Aspect | Details |
|---|---|
| Aggregate Root | `MandateRepresentation` |
| Identity | `RepresentationId` (UUID) |
| Description | A scheduled re-presentation of a bounced NACH/ECS mandate. |

**Value Objects:**
- `RepresentationStatus` — Enum: SCHEDULED, SUBMITTED, SUCCESS, BOUNCED, CANCELLED
- `RepresentationConfig` — (maxAttempts, daysBetweenAttempts, eligibleReturnCodes)

### 15.3 Functional Requirements

| Req ID | Requirement | Priority | Acceptance Criteria |
|---|---|---|---|
| COL-PF-001 | System shall generate unique, time-bound payment links for delinquent accounts, embeddable in SMS, WhatsApp, and email. | P0 | Link generated in <1 second; opens mobile-friendly payment page |
| COL-PF-002 | Payment link page shall show: borrower name, loan account (masked), overdue amount breakup (principal, interest, charges), total payable, and support multiple payment modes (UPI, netbanking, debit card). | P0 | Payment page loads in <3 seconds; shows accurate breakup |
| COL-PF-003 | System shall generate dynamic QR codes (UPI QR) linked to specific loan accounts for in-person and remote payment. | P0 | QR code scannable; amount pre-filled; payment credited to correct account |
| COL-PF-004 | System shall support partial payment acceptance (customer pays what they can) with real-time allocation per waterfall and DPD recalculation. | P0 | Partial payment accepted; allocated correctly; DPD updated |
| COL-PF-005 | System shall support repayment plan creation: split overdue amount into 2-6 installments over 30-180 days. With approval workflow. | P0 | Plan created; customer digitally accepts; installments tracked |
| COL-PF-006 | System shall auto-send payment reminders for repayment plan installments (T-3, T-1, due date). | P0 | Reminders sent on schedule |
| COL-PF-007 | System shall schedule NACH re-presentation for eligible bounced mandates per configurable rules (eligible return codes, waiting period, max attempts). | P0 | Re-presentation scheduled correctly; file generated |
| COL-PF-008 | System shall provide foreclosure/settlement calculator accessible to collection agents and customers (via self-service portal). | P1 | Calculator shows accurate closure amount with validity period |
| COL-PF-009 | System shall send digital payment receipt (SMS + email + WhatsApp) immediately upon successful payment via any channel. | P0 | Receipt sent within 2 minutes of payment confirmation |
| COL-PF-010 | System shall track payment link performance (generated, clicked, initiated, completed, dropped — funnel analysis). | P0 | Funnel metrics on dashboard |

---

## 16. COMPLIANCE & AUDIT CONTEXT (BC-10)

### 16.1 Context Overview

| Attribute | Details |
|---|---|
| Context Name | Compliance & Audit Context |
| Context ID | BC-10 |
| Domain Classification | Generic (but Critical) |
| Owner Team | Compliance & Internal Audit |
| Upstream Dependencies | ALL bounded contexts (conformist — consumes events from all contexts) |
| Downstream Consumers | Analytics (BC-11), External regulators |
| Primary Objective | Ensure 100% regulatory compliance, maintain complete audit trails, detect violations, and automate regulatory reporting |

### 16.2 Aggregates

#### 16.2.1 Aggregate: ComplianceRule

| Aspect | Details |
|---|---|
| Aggregate Root | `ComplianceRule` |
| Identity | `ComplianceRuleId` |
| Description | A regulatory or internal compliance rule that the system must enforce. |

**Value Objects:**
- `RuleCategory` — Enum: RBI_FAIR_PRACTICES, TRAI_DND, SARFAESI, IRAC, PRIVACY, AML, INTERNAL_POLICY
- `EnforcementType` — Enum: PREVENTIVE (blocks action), DETECTIVE (flags after action), CORRECTIVE (auto-remediation)
- `ViolationSeverity` — Enum: LOW, MEDIUM, HIGH, CRITICAL

#### 16.2.2 Aggregate: AuditTrail

| Aspect | Details |
|---|---|
| Aggregate Root | `AuditTrail` |
| Identity | `AuditEntryId` (UUID) |
| Description | Immutable record of every significant action in the system. |

**Value Objects:**
- `AuditEntry` — (timestamp, actor, actorType [SYSTEM/USER/AGENCY/AI_AGENT], action, entity, entityId, beforeState, afterState, ipAddress, sessionId)

#### 16.2.3 Aggregate: RegulatoryReport

| Aspect | Details |
|---|---|
| Aggregate Root | `RegulatoryReport` |
| Identity | `ReportId` (UUID) |
| Description | A regulatory report generated for submission to RBI, NHB, or other regulators. |

**Value Objects:**
- `ReportType` — Enum: CRILC_SMA, CRILC_QUARTERLY, NPA_MOVEMENT, WRITE_OFF_REPORT, WILFUL_DEFAULTER, COLLECTION_PRACTICES_AUDIT, BUREAU_UPDATE, OMBUDSMAN_COMPLAINT
- `ReportStatus` — Enum: GENERATED, REVIEWED, APPROVED, SUBMITTED, ACKNOWLEDGED

### 16.3 Domain Services

#### 16.3.1 FairPracticesComplianceEngine

**Purpose:** Real-time enforcement of RBI Fair Practices Code for collection activities.

**Rules Enforced:**

| Rule ID | Rule | Enforcement | Regulatory Reference |
|---|---|---|---|
| COL-CP-R001 | No calls before 8:00 AM or after 7:00 PM (borrower's local time) | Preventive — blocks call/IVR/AI voice scheduling | RBI FPC; Supreme Court guidelines |
| COL-CP-R002 | No more than 3 communication attempts per day per borrower across all channels | Preventive — blocks excess communication | Internal policy (RBI FPC spirit) |
| COL-CP-R003 | No abusive, threatening, or coercive language in any communication | Detective — AI monitors call recordings and text; flags violations | RBI FPC; Indian Penal Code |
| COL-CP-R004 | Collection agent must identify themselves and the institution at start of every interaction | Detective — call recording audit | RBI FPC |
| COL-CP-R005 | No third-party disclosure of borrower's default (except to authorized parties) | Preventive — system restricts data access | RBI FPC; IT Act privacy provisions |
| COL-CP-R006 | Borrower's place of employment must not be visited for collection purposes | Detective — field visit address validation | RBI FPC |
| COL-CP-R007 | Recovery agent must carry authorization letter during field visits | Detective — visit checklist enforcement | RBI FPC |
| COL-CP-R008 | SARFAESI Section 13(2) notice must provide 60 days for borrower response | Preventive — system blocks premature action | SARFAESI Act Section 13(3A) |
| COL-CP-R009 | Collection communications must be in language understood by borrower | Preventive — language preference check before sending | RBI FPC |
| COL-CP-R010 | DND/NCPR registry must be checked before promotional communications | Preventive — TRAI DND API check | TRAI regulations |

### 16.4 Functional Requirements

| Req ID | Requirement | Priority | Acceptance Criteria |
|---|---|---|---|
| COL-CP-001 | System shall maintain immutable audit trail for every action (user action, system action, AI action, agency action) with timestamp, actor, before/after state. | P0 | Audit trail immutable (append-only); queryable; retained for minimum 8 years |
| COL-CP-002 | System shall enforce all RBI Fair Practices Code rules in real-time (preventive) for communication timing, frequency, and content. | P0 | Zero violations of preventive rules |
| COL-CP-003 | System shall monitor call recordings (sample-based and AI-powered full scan) for compliance violations (threatening language, non-disclosure, harassment). | P0 | 100% AI scan; 5% manual audit; violations flagged within 24 hours |
| COL-CP-004 | System shall generate CRILC SMA reports (weekly) and CRILC quarterly reports per RBI format and submit via secure channel. | P0 | Reports generated on schedule; validated before submission |
| COL-CP-005 | System shall generate NPA movement report (monthly), write-off report (quarterly), and wilful defaulter report per RBI formats. | P0 | Reports accurate; reconciled with GL |
| COL-CP-006 | System shall auto-update credit bureau records (CIBIL, Equifax, Experian, CRIF High Mark) monthly for all delinquent accounts — DPD, asset classification, settlement status, write-off status. | P0 | Bureau files generated per bureau format; submitted within RBI timelines |
| COL-CP-007 | System shall track customer complaints related to collections with SLA (acknowledge within 24 hours, resolve within 15 days per RBI). | P0 | Complaint lifecycle tracked; SLA monitored; escalation to ombudsman handled |
| COL-CP-008 | System shall maintain maker-checker workflow for all sensitive operations: strategy changes, agency empanelment, settlement approvals, write-off approvals, template changes. | P0 | No sensitive operation without maker-checker |
| COL-CP-009 | System shall support data privacy compliance: data masking (PAN, Aadhaar, bank account), consent management, data access logging, right to information requests. | P0 | PII masked in UI/logs; access logged; consent tracked |
| COL-CP-010 | System shall comply with RBI's data localization requirements — all customer data stored within India. | P0 | All data centers in India; no cross-border data transfer |
| COL-CP-011 | System shall maintain role-based access control (RBAC) with least-privilege principle: collection agents, supervisors, legal team, agency users, auditors, admins each with distinct access levels. | P0 | RBAC enforced; access reviews quarterly |
| COL-CP-012 | System shall support regulatory examination readiness: on-demand generation of any compliance report, account history, communication history with full audit trail for regulatory inspection. | P0 | Any data retrievable within 4 hours of regulator request |

---

## 17. ANALYTICS CONTEXT (BC-11)

### 17.1 Context Overview

| Attribute | Details |
|---|---|
| Context Name | Analytics Context |
| Context ID | BC-11 |
| Domain Classification | Supporting Domain |
| Owner Team | Business Intelligence & Analytics |
| Upstream Dependencies | ALL bounded contexts (conformist — consumes events and data from all contexts via CDC + event stream) |
| Downstream Consumers | Management dashboards, Board reports, Regulatory reports |
| Primary Objective | Provide real-time and historical analytics, dashboards, and predictive insights for collections performance management |

### 17.2 Functional Requirements

#### 17.2.1 Portfolio Dashboards

| Req ID | Requirement | Priority | Acceptance Criteria |
|---|---|---|---|
| COL-AN-001 | System shall provide real-time portfolio health dashboard: total outstanding, bucket-wise distribution (count & amount), NPA ratio, SMA distribution, collection efficiency ratio (CER). | P0 | Dashboard refreshes within 5 minutes; drill-down to branch/product/region |
| COL-AN-002 | System shall provide roll-rate analysis: month-over-month transition matrix showing % of accounts moving between buckets (forward roll, backward roll, static). | P0 | Roll-rate matrix generated monthly; trend comparison available |
| COL-AN-003 | System shall provide flow-rate analysis: tracking cohorts of accounts as they move through delinquency stages over time. | P0 | Flow-rate charts with configurable time periods |
| COL-AN-004 | System shall provide vintage analysis: delinquency performance grouped by origination month/quarter, showing how each vintage ages into delinquency. | P0 | Vintage curves with comparison across vintages |
| COL-AN-005 | System shall provide concentration risk dashboards: top 10/20/50 delinquent exposures, geographic concentration, product concentration, employer concentration (for salaried). | P0 | Concentration reports on-demand |

#### 17.2.2 Collection Performance Analytics

| Req ID | Requirement | Priority | Acceptance Criteria |
|---|---|---|---|
| COL-AN-006 | System shall compute Collection Efficiency Ratio (CER): `Total collected / Total demand (due + overdue)` — daily, monthly, quarterly, by product, by bucket, by branch, by region. | P0 | CER dashboard with trend analysis |
| COL-AN-007 | System shall compute resolution rate: % of collection cases resolved (DPD back to 0) within 30/60/90 days of case opening. | P0 | Resolution rate by bucket, product, agency |
| COL-AN-008 | System shall compute PTP conversion rate: % of PTPs that resulted in actual payment within promised date + 7 days grace. | P0 | PTP metrics per agent, agency, channel |
| COL-AN-009 | System shall provide agent/team productivity report: accounts handled, contacts made, PTPs obtained, collections achieved, disposition breakdown. | P0 | Per-agent daily/weekly/monthly report |
| COL-AN-010 | System shall provide agency comparative performance: ranking of agencies by recovery rate, contact rate, PTP conversion, compliance score, cost per recovery. | P0 | Agency leaderboard with trend |

#### 17.2.3 Channel & Campaign Analytics

| Req ID | Requirement | Priority | Acceptance Criteria |
|---|---|---|---|
| COL-AN-011 | System shall analyze channel effectiveness: for each channel (SMS, WhatsApp, IVR, AI Voice, AI Chat, email, field visit), measure: delivery rate, response rate, payment conversion rate, cost per contact, cost per recovery. | P0 | Channel comparison dashboard |
| COL-AN-012 | System shall provide campaign analytics: for each campaign, measure: reach, response rate, payment conversion, ROI. | P0 | Per-campaign dashboard |
| COL-AN-013 | System shall track AI agent performance: AI voice/chat conversation completion rate, PTP capture rate, customer satisfaction (if measured), escalation rate, cost comparison vs human agent. | P0 | AI vs human comparison metrics |

#### 17.2.4 Predictive & Advanced Analytics

| Req ID | Requirement | Priority | Acceptance Criteria |
|---|---|---|---|
| COL-AN-014 | System shall provide recovery forecasting using ML models: predicted collection amount for next 30/60/90 days by bucket, product, region. | P1 | Forecast accuracy within ±10% of actual (validated monthly) |
| COL-AN-015 | System shall compute ECL (Expected Credit Loss) projections for IND AS 109 reporting with forward-looking macro-economic scenario analysis. | P0 | ECL projection per quarter; multiple scenarios (base, stress, optimistic) |
| COL-AN-016 | System shall provide cost-of-collection analysis: total collection cost (staff, agency commissions, technology, communication, legal) vs amount recovered, by bucket and product. | P0 | Cost analysis monthly |
| COL-AN-017 | System shall support custom report builder allowing business users to create ad-hoc reports by selecting dimensions (product, bucket, region, channel, agency, time period) and measures (count, amount, rates). | P1 | Self-service report builder; SQL-free |

#### 17.2.5 Board & Management MIS

| Req ID | Requirement | Priority | Acceptance Criteria |
|---|---|---|---|
| COL-AN-018 | System shall generate monthly Board MIS pack: portfolio quality, NPA trends, provision movement, CER, agency performance summary, legal recovery summary, write-off summary, EWS summary. | P0 | Board pack auto-generated by 10th of each month |
| COL-AN-019 | System shall generate weekly management flash report: week-over-week collection performance, significant bucket movements, new NPA additions, EWS alerts summary. | P0 | Flash report auto-generated every Monday by 9 AM |
| COL-AN-020 | System shall provide executive dashboard: single screen showing key metrics (NPA%, CER, total overdue, total collected MTD, top risks). | P0 | Executive dashboard with mobile-responsive design |

---

## END OF PART 08

Next: **COL-BRD-Part09-NFR-DataModel-Glossary.md** -- Non-Functional Requirements, Data Model, API Contracts, Glossary.
