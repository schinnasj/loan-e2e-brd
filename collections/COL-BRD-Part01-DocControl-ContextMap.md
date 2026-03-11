# BUSINESS REQUIREMENTS DOCUMENT
# LOAN COLLECTION MANAGEMENT SYSTEM (LCMS)
# DOMAIN-DRIVEN DESIGN ARCHITECTURE
# FOR INDIAN BANKS, NBFCs, HFCs, SFBs & COOPERATIVE BANKS

---

## DOCUMENT CONTROL

| Field | Details |
|---|---|
| Document Title | Business Requirements Document -- Loan Collection Management System (DDD Architecture) |
| Document ID | LCMS-BRD-2026-001 |
| Version | 1.0 |
| Status | Draft for Review |
| Classification | Confidential |
| Created Date | 11-Mar-2026 |
| Last Updated | 11-Mar-2026 |
| Author | Product Management -- Lending Technology |
| Department | Digital Banking & Lending Transformation |
| Parent Document | LMS-BRD-2026-001 (Loan Management System BRD) |
| Architecture Approach | Domain-Driven Design (DDD) with Bounded Contexts |

### Version History

| Version | Date | Author | Change Description |
|---|---|---|---|
| 0.1 | 11-Mar-2026 | Product Management | Initial draft -- Reconciliation, Delinquency, Workflow bounded contexts |
| 0.2 | -- | -- | Communication, Sentiment, Mandate bounded contexts |
| 0.3 | -- | -- | Agency, Field, Legal, Payment, Compliance, Analytics contexts |
| 0.4 | -- | -- | NFR, Data Model, API Contracts, Glossary |
| 1.0 | -- | -- | Final consolidated version after stakeholder review |

### Approvers and Sign-Off Matrix

| Role | Name | Department | Sign-Off Status | Date |
|---|---|---|---|---|
| Chief Technology Officer | [TBD] | Technology | Pending | -- |
| Chief Risk Officer | [TBD] | Risk Management | Pending | -- |
| Head -- Collections & Recovery | [TBD] | Collections | Pending | -- |
| Head -- Retail Lending | [TBD] | Retail Banking | Pending | -- |
| Chief Compliance Officer | [TBD] | Compliance | Pending | -- |
| Head -- Finance & Accounts | [TBD] | Finance | Pending | -- |
| Head -- Operations | [TBD] | Loan Operations | Pending | -- |
| Head -- Legal | [TBD] | Legal | Pending | -- |
| CISO | [TBD] | Information Security | Pending | -- |
| Head -- Internal Audit | [TBD] | Audit | Pending | -- |
| Head -- Data Science / Analytics | [TBD] | Analytics | Pending | -- |
| Head -- Customer Service | [TBD] | Customer Experience | Pending | -- |

### Distribution List

| Name | Role | Distribution Type |
|---|---|---|
| CTO Office | Technology Decision Makers | Full Document |
| Collections Leadership | Business Owners | Full Document |
| Development Teams | Implementation | Full Document |
| QA Team | Testing & Validation | Full Document |
| Data Science Team | ML/AI Implementation | Parts 05, 08, 09 |
| Legal Department | Legal Module Review | Parts 07, 08 |
| Compliance Team | Regulatory Review | Parts 03, 08 |
| External Vendor (if applicable) | System Integrator | Full Document (under NDA) |

### Related Documents

| Document ID | Title | Relationship |
|---|---|---|
| LMS-BRD-2026-001 | Loan Management System BRD | Parent system; LCMS consumes loan lifecycle data |
| LMS-BRD-Part03 | LMS Reconciliation Module | Foundation for COL Reconciliation Context |
| LMS-BRD-Part05 | LMS Overdue & Early Closure Module | Foundation for COL Delinquency Context |
| LMS-BRD-Part07 | LMS Regulatory & Integrations | Shared regulatory framework |

---

## TABLE OF CONTENTS

### Part 01 -- Document Control & Context Map (this file)
1. Executive Summary
2. Business Context and Objectives
3. Scope Definition
4. Domain-Driven Design Overview
5. DDD Context Map -- Bounded Contexts and Relationships
6. Ubiquitous Language -- System-Wide Glossary

### Part 02 -- Reconciliation Bounded Context
7. Reconciliation Context -- Aggregates, Events, Services, Requirements

### Part 03 -- Delinquency Bounded Context
8. Delinquency Context -- DPD Engine, Bucket Management, NPA Classification

### Part 04 -- Workflow & Communication Bounded Contexts
9. Workflow & Campaign Context -- Notification Workflows, Escalation, Campaigns
10. Communication Context -- Multi-Channel Engine, Agentic Conversations

### Part 05 -- Sentiment & Intelligence Bounded Context
11. Sentiment & Intelligence Context -- Sentiment Analysis, Propensity Models, EWS

### Part 06 -- Mandate & Account 360 Bounded Context
12. Mandate & Account Context -- Mandate Management, Customer 360, EMI Management

### Part 07 -- Agency, Field & Legal Bounded Contexts
13. Agency & Field Context -- Collection Agency, Field Visits, Allocation
14. Legal & Recovery Context -- Legal Proceedings, Settlements, Write-offs

### Part 08 -- Payment, Compliance & Analytics Bounded Contexts
15. Payment Facilitation Context -- Payment Links, Restructuring, Re-presentation
16. Compliance & Audit Context -- Regulatory Compliance, Audit Trails
17. Analytics Context -- MIS, Dashboards, ML Models

### Part 09 -- Non-Functional Requirements, Data Model & Glossary
18. Non-Functional Requirements
19. Data Model Highlights per Context
20. Sample API Contracts
21. Integration Architecture
22. Regulatory Compliance Matrix
23. Assumptions, Risks, and Dependencies
24. Full Glossary with Ubiquitous Language

---

## 1. EXECUTIVE SUMMARY

### 1.1 Purpose

This Business Requirements Document defines the complete functional, non-functional, regulatory, and integration requirements for building a production-grade **Loan Collection Management System (LCMS)** designed using **Domain-Driven Design (DDD)** principles. The system is architected around eleven bounded contexts, each representing a distinct subdomain of the collections lifecycle.

The LCMS is designed for deployment across Indian financial institutions including Scheduled Commercial Banks (SCBs), Small Finance Banks (SFBs), Non-Banking Financial Companies (NBFCs -- including NBFC-UL, NBFC-ML, NBFC-BL per RBI's Scale-Based Regulation framework dated October 2021), Housing Finance Companies (HFCs) regulated by the National Housing Bank (NHB), and Urban Cooperative Banks (UCBs).

The system serves as a companion to the core Loan Management System (LMS-BRD-2026-001), consuming loan lifecycle data and providing specialized collections intelligence, workflow automation, multi-channel communication, sentiment-driven strategy, legal management, and regulatory compliance capabilities.

### 1.2 Business Context

Indian lending institutions face significant challenges in collections management:

- **Scale**: India's retail credit outstanding exceeded INR 50 lakh crore by Q3 FY2025-26, with delinquency rates varying from 1.5% (home loans) to 5%+ (unsecured personal loans and microfinance).
- **Regulatory intensity**: RBI's IRAC norms mandate daily DPD tracking, real-time SMA/CRILC reporting, strict NPA classification timelines, and adherence to Fair Practices Code for collections.
- **Digital transformation**: The proliferation of digital lending (estimated 60%+ of new personal loans originated digitally) demands equally digital, intelligent collection capabilities.
- **Customer experience**: RBI's Fair Practices Code and increasing consumer awareness demand respectful, compliant, omni-channel collection approaches. Aggressive or harassing collection practices attract regulatory penalties and reputational damage.
- **Data-driven decisions**: Traditional rule-based collection strategies are giving way to AI/ML-driven approaches leveraging sentiment analysis, payment behavior prediction, and propensity modeling.
- **Agency management complexity**: Most institutions deploy 10-50+ collection agencies across geographies, requiring robust allocation, monitoring, compliance enforcement, and performance management.

### 1.3 Strategic Objectives

| Objective ID | Objective | Success Metric |
|---|---|---|
| SO-01 | Reduce delinquency rates across all product portfolios | 15-25% reduction in 90+ DPD accounts within 12 months |
| SO-02 | Improve collection efficiency ratio (CER) | CER improvement from industry average 85% to 92%+ |
| SO-03 | Achieve 100% regulatory compliance | Zero RBI observations on collections practices |
| SO-04 | Reduce cost of collection | 20% reduction in per-account collection cost |
| SO-05 | Improve customer experience during collections | NPS score for resolved delinquencies above -10 |
| SO-06 | Enable real-time portfolio monitoring | Sub-second portfolio health dashboards |
| SO-07 | Automate 80%+ of routine collection interactions | 80% of early bucket (0-30 DPD) handled without human agent |
| SO-08 | Achieve 99%+ reconciliation accuracy | Auto-match rate above 97% across all payment rails |
| SO-09 | Reduce time to NPA recognition and reporting | Real-time NPA classification; CRILC within regulatory timelines |
| SO-10 | Build predictive collection capabilities | EWS identifies 70%+ of future NPAs 30 days in advance |

### 1.4 Key Stakeholders

| Stakeholder Group | Primary Concerns | LCMS Interaction |
|---|---|---|
| Collections Head / VP Collections | Overall recovery rates, strategy effectiveness, regulatory compliance | Strategy definition, dashboard consumer, escalation receiver |
| Collection Supervisors | Team productivity, queue management, PTP tracking | Daily operations, workflow management |
| Tele-calling Agents | Call scripts, customer data, disposition capture | Primary system users for outbound calling |
| Field Collection Agents | Visit scheduling, route optimization, receipt generation | Mobile app users |
| Legal Team | Notice generation, case tracking, SARFAESI compliance | Legal module users |
| Risk Management | NPA trends, provisioning impact, EWS signals | Dashboard consumers, rule configurators |
| Finance / Accounts | Reconciliation accuracy, GL impact, write-off accounting | Reconciliation module users |
| Compliance / Audit | Regulatory adherence, fair practices, audit trails | Compliance dashboard, audit log consumers |
| IT / Engineering | System performance, integration health, data quality | System administration, monitoring |
| Collection Agencies (External) | Account allocation, performance targets, commission | Agency portal users |
| Customers (Borrowers) | Fair treatment, payment convenience, dispute resolution | Receive communications, make payments |

---

## 2. BUSINESS CONTEXT AND OBJECTIVES

### 2.1 Current State Analysis (Typical AS-IS Pain Points)

| Area | Typical Pain Point | Impact |
|---|---|---|
| DPD Calculation | Manual or batch-only DPD computation; errors in partial payment handling | Incorrect SMA/NPA classification; regulatory risk |
| Reconciliation | Manual matching of collections from 8+ payment channels | Aged suspense balances; delayed EMI credit; customer complaints |
| Communication | Siloed channels; no unified view; manual campaign execution | Low contact rates; compliance violations (wrong timing, frequency) |
| Agency Management | Spreadsheet-based allocation; no real-time monitoring | Suboptimal recovery; agency malpractice undetected |
| Customer Intelligence | No sentiment analysis; one-size-fits-all collection strategy | Missed recovery opportunities; customer harassment complaints |
| Legal Process | Paper-based legal tracking; no SARFAESI workflow automation | Delayed legal action; missed limitation periods |
| Reporting | Manual MIS; delayed portfolio health visibility | Delayed management decisions; audit findings |
| Payment Facilitation | Limited self-service payment options for delinquent customers | Lost recovery opportunities; higher cost of collection |

### 2.2 Target State Vision

The LCMS represents a paradigm shift from reactive, manual collections to **proactive, intelligent, omni-channel, compliance-first** collections management. Key characteristics of the target state:

1. **Event-driven architecture**: Every collection-relevant occurrence (payment, bounce, bucket change, sentiment shift) is a domain event that triggers appropriate downstream actions automatically.
2. **Intelligence-first**: AI/ML models drive strategy selection, channel optimization, timing decisions, and agent allocation.
3. **Omni-channel with memory**: Customers experience consistent, context-aware interactions regardless of channel (SMS, WhatsApp, IVR, voice agent, field visit).
4. **Compliance by design**: Regulatory guardrails are embedded in the domain model, not bolted on as afterthoughts. Fair practices code, call timing restrictions, communication frequency limits, and harassment detection are core system capabilities.
5. **Real-time everything**: DPD calculation, bucket movement, reconciliation, sentiment scoring, and portfolio dashboards operate in real-time or near-real-time.

---

## 3. SCOPE DEFINITION

### 3.1 In-Scope

| Module | Description |
|---|---|
| Collection Reconciliation Engine | Multi-rail transaction matching, suspense management, bounce processing |
| DPD/Bucket Engine | Daily DPD calculation, asset classification, bucket management |
| Workflow & Campaign Engine | Rule-based notification sequences, escalation matrices, campaign management |
| Multi-Channel Communication | SMS, IVR, WhatsApp, Email, Agentic Voice, Agentic Text |
| Sentiment & Intelligence | Sentiment scoring, propensity models, Early Warning System |
| Mandate & Account 360 | Mandate management, customer 360 view, EMI lifecycle |
| Agency & Field Management | Agency allocation, performance, field visits, route optimization |
| Legal & Recovery | SARFAESI, OTS, write-off, legal case tracking |
| Payment Facilitation | Payment links, QR codes, restructuring, re-presentation |
| Compliance & Audit | Fair practices, DND, audit trails, regulatory reporting |
| Analytics & MIS | Dashboards, roll-rate analysis, recovery forecasting |

### 3.2 Out-of-Scope

| Item | Reason |
|---|---|
| Loan origination and underwriting | Covered in LMS-BRD-2026-001 |
| Core Banking System (CBS) functions | LCMS integrates with CBS; does not replace it |
| Customer onboarding and KYC | Covered in LMS; LCMS consumes customer data |
| Disbursement processing | Covered in LMS |
| Interest computation engine | Covered in LMS; LCMS consumes computed interest/EMI data |
| Collateral management (creation/valuation) | Covered in LMS; LCMS consumes collateral data for legal actions |
| Treasury and fund management | Separate system |
| HR management for collection staff | Separate HR system |

### 3.3 Product Applicability Matrix

| Product Type | Applicable Contexts | Special Rules |
|---|---|---|
| Home Loan | All 11 contexts | SARFAESI applicable; NHB norms for HFCs; longer legal timelines |
| Personal Loan (Unsecured) | All except SARFAESI in Legal | Higher provisioning (25% sub-standard); faster escalation |
| Vehicle Loan | All 11 contexts | Hypothecation; vehicle repossession workflow; RTO integration |
| Business Loan / MSME | All 11 contexts | MSME restructuring norms; Kalanidhi Maran committee norms |
| Loan Against Property (LAP) | All 11 contexts | SARFAESI applicable; property auction workflow |
| Gold Loan | Reconciliation, Delinquency, Communication, Payment | Auction of gold; shorter NPA cycle; LTV monitoring |
| Microfinance / JLG | All except Legal (limited) | Group collection dynamics; doorstep collection; weekly repayment |
| Education Loan | All 11 contexts | Moratorium period handling; CSIS norms |
| Agricultural / KCC | All 11 contexts | Crop season-based NPA; government waiver handling |
| Co-Lending (CLM) | All 11 contexts | Split collection; escrow reconciliation; partner reporting |
| BNPL / Digital Lending | All 11 contexts | LSP integration; FLDG tracking; digital-first communication |

### 3.4 Entity Applicability

| Entity Type | RBI Classification | Special Considerations |
|---|---|---|
| Scheduled Commercial Banks (SCBs) | RBI regulated | Full IRAC, CRILC, CRR/SLR; SARFAESI Act applicable |
| Small Finance Banks (SFBs) | RBI regulated | Priority sector 75%; microfinance focus; SARFAESI applicable |
| NBFC-UL (Upper Layer) | RBI SBR - Upper Layer | Enhanced governance; 90 DPD NPA (aligned); CRILC applicable |
| NBFC-ML (Middle Layer) | RBI SBR - Middle Layer | 90 DPD NPA; CRILC for INR 5 Cr+ exposures |
| NBFC-BL (Base Layer) | RBI SBR - Base Layer | 90 DPD NPA; limited CRILC requirements |
| HFCs | NHB regulated | NHB IRAC norms; SARFAESI for housing loans; NHB returns |
| UCBs | RBI regulated | Limited SARFAESI; state cooperative laws may apply |
| Microfinance Institutions | RBI (if NBFC-MFI) | Microfinance lending norms; cap on interest rates |

---

## 4. DOMAIN-DRIVEN DESIGN OVERVIEW

### 4.1 Why DDD for Collections Management

Loan collection management is a complex domain with multiple interacting subdomains, each with its own language, rules, and lifecycle. Domain-Driven Design is chosen because:

1. **Complex domain logic**: Collection rules vary by product, bucket, entity type, regulatory regime, and customer segment. DDD's emphasis on modeling domain complexity through aggregates and domain events captures this naturally.
2. **Multiple team ownership**: Different teams own different aspects (reconciliation team, collection operations, legal, compliance, analytics). Bounded contexts align with team boundaries.
3. **Regulatory isolation**: Compliance rules must be isolated and independently auditable. A dedicated Compliance bounded context ensures regulatory logic is not scattered across the system.
4. **Event-driven nature**: Collections is inherently event-driven (payment received, bounce occurred, bucket changed, sentiment shifted). Domain events are first-class citizens in DDD.
5. **Evolutionary architecture**: New collection strategies, channels, and regulatory requirements emerge frequently. Bounded contexts enable independent evolution without system-wide impact.

### 4.2 DDD Building Blocks Used in This BRD

| DDD Concept | Definition in LCMS Context |
|---|---|
| **Bounded Context** | A logical boundary within which a particular domain model applies. Each bounded context has its own ubiquitous language, aggregates, and services. |
| **Aggregate** | A cluster of domain objects treated as a single unit for data changes. Each aggregate has a root entity and enforces invariants. |
| **Entity** | An object with a distinct identity that persists over time (e.g., LoanAccount, CollectionCase). |
| **Value Object** | An object without identity, defined by its attributes (e.g., Money, DPDBucket, Address). |
| **Domain Event** | A record of something significant that happened in the domain (e.g., PaymentReceived, BucketChanged). Events are immutable and timestamped. |
| **Domain Service** | Logic that does not naturally belong to any single entity or value object (e.g., ReconciliationMatchingService). |
| **Application Service** | Orchestrates use cases by coordinating domain objects and infrastructure (e.g., ProcessBounceFileUseCase). |
| **Integration Event** | An event published to other bounded contexts via an event bus/message broker. Enables loose coupling. |
| **Anti-Corruption Layer (ACL)** | A translation layer that prevents one bounded context's model from leaking into another. |
| **Repository** | Abstracts data persistence for aggregates. |

### 4.3 Architectural Principles

1. **Each bounded context owns its data store** -- no shared databases between contexts.
2. **Inter-context communication via integration events** -- asynchronous by default, synchronous (API) only when necessary.
3. **Eventual consistency** between contexts is acceptable; strong consistency within an aggregate.
4. **Each context can be independently deployed, scaled, and versioned**.
5. **Shared Kernel** is used sparingly -- only for truly shared value objects (Money, LoanAccountId, CustomerId).
6. **Anti-Corruption Layers** protect each context from external system changes (CBS, bureau, payment networks).

---

## 5. DDD CONTEXT MAP -- BOUNDED CONTEXTS AND RELATIONSHIPS

### 5.1 Bounded Context Inventory

| # | Bounded Context | Core Domain / Supporting / Generic | Owner Team |
|---|---|---|---|
| BC-01 | **Reconciliation Context** | Supporting | Finance Operations |
| BC-02 | **Delinquency Context** | Core | Risk & Collections |
| BC-03 | **Workflow & Campaign Context** | Core | Collection Strategy |
| BC-04 | **Communication Context** | Supporting | Communication Platform |
| BC-05 | **Sentiment & Intelligence Context** | Core | Data Science |
| BC-06 | **Mandate & Account Context** | Supporting | Loan Operations |
| BC-07 | **Agency & Field Context** | Core | Agency Management |
| BC-08 | **Legal & Recovery Context** | Core | Legal & Recovery |
| BC-09 | **Payment Facilitation Context** | Supporting | Digital Payments |
| BC-10 | **Compliance & Audit Context** | Generic (but critical) | Compliance |
| BC-11 | **Analytics Context** | Supporting | Business Intelligence |

### 5.2 Context Map -- Relationships

```
+------------------------------------------------------------------+
|                     LCMS CONTEXT MAP                              |
+------------------------------------------------------------------+

  UPSTREAM                              DOWNSTREAM
  --------                              ----------

  [External: CBS / LMS]
       |
       | (ACL - Anti-Corruption Layer)
       v
  +-------------------+    PaymentReceived     +-------------------+
  | BC-01             |----------------------->| BC-02             |
  | RECONCILIATION    |    BounceProcessed     | DELINQUENCY       |
  | CONTEXT           |----------------------->|  CONTEXT          |
  +-------------------+                        +-------------------+
       |                                            |
       | ReconciliationCompleted                    | BucketChanged
       | SuspenseEntryCreated                       | NPAClassified
       v                                            | SMAReported
  +-------------------+                            v
  | BC-10             |                        +-------------------+
  | COMPLIANCE &      |<---------ALL--------  | BC-03             |
  | AUDIT CONTEXT     |   (Conformist)         | WORKFLOW &        |
  +-------------------+                        | CAMPAIGN CONTEXT  |
       ^                                        +-------------------+
       |                                            |
       | AuditEvents from all contexts              | NotificationTriggered
       |                                            | EscalationRaised
  +-------------------+                            v
  | BC-11             |                        +-------------------+
  | ANALYTICS         |<------- ALL ------     | BC-04             |
  | CONTEXT           |   (Conformist)         | COMMUNICATION     |
  +-------------------+                        | CONTEXT           |
                                                +-------------------+
                                                    |
  +-------------------+    SentimentUpdated         | MessageDelivered
  | BC-05             |<---------------------------|  ConversationEvent
  | SENTIMENT &       |                            |
  | INTELLIGENCE      |---StrategyRecommended----->|
  +-------------------+                        +-------------------+
       |                                        | BC-06             |
       | RiskScoreUpdated                       | MANDATE &         |
       | EWSAlertRaised                         | ACCOUNT CONTEXT   |
       v                                        +-------------------+
  +-------------------+                            ^
  | BC-07             |     AccountAllocated        |
  | AGENCY & FIELD    |<---------------------------|
  | CONTEXT           |                            |
  +-------------------+                        +-------------------+
       |                                        | BC-09             |
       | LegalReferralMade                      | PAYMENT           |
       v                                        | FACILITATION      |
  +-------------------+                        +-------------------+
  | BC-08             |
  | LEGAL & RECOVERY  |
  | CONTEXT           |
  +-------------------+
```

### 5.3 Context Relationships -- Detailed

| Upstream Context | Downstream Context | Relationship Type | Integration Mechanism | Key Events Exchanged |
|---|---|---|---|---|
| External CBS/LMS | Reconciliation | ACL (Anti-Corruption Layer) | API + File (SFTP) | Loan data, payment data, bank statements |
| External CBS/LMS | Mandate & Account | ACL | API + Event Stream | Loan account data, EMI schedule, customer data |
| Reconciliation | Delinquency | Customer-Supplier | Integration Events | PaymentCredited, BounceConfirmed, SuspenseCleared |
| Delinquency | Workflow & Campaign | Customer-Supplier | Integration Events | BucketChanged, NPAClassified, DPDUpdated |
| Workflow & Campaign | Communication | Customer-Supplier | Integration Events (Command) | SendNotification, ExecuteCampaign |
| Communication | Sentiment & Intelligence | Customer-Supplier | Integration Events | ConversationCompleted, MessageResponseReceived |
| Sentiment & Intelligence | Workflow & Campaign | Partnership | Integration Events | StrategyRecommended, SentimentScoreUpdated |
| Sentiment & Intelligence | Agency & Field | Customer-Supplier | Integration Events | RiskCategoryUpdated, EWSAlertRaised |
| Mandate & Account | Workflow & Campaign | Customer-Supplier | API (Query) | Customer360Data, MandateStatus |
| Mandate & Account | Agency & Field | Customer-Supplier | API (Query) | AccountDetails, OutstandingAmount |
| Agency & Field | Legal & Recovery | Customer-Supplier | Integration Events | LegalReferralRequested, FieldVisitEvidenceSubmitted |
| Payment Facilitation | Reconciliation | Customer-Supplier | Integration Events | PaymentLinkUsed, PaymentReceived |
| All Contexts | Compliance & Audit | Conformist | Event Stream | All domain events published to compliance |
| All Contexts | Analytics | Conformist | Event Stream + CDC | All domain events + data snapshots |

### 5.4 Shared Kernel -- Common Value Objects

The following value objects are shared across bounded contexts via a lightweight shared library. Changes to these require consensus across all context owners.

| Value Object | Attributes | Used By |
|---|---|---|
| `Money` | amount (BigDecimal), currency (INR default) | All contexts |
| `LoanAccountId` | accountNumber (String), entityCode (String) | All contexts |
| `CustomerId` | customerNumber (String), entityCode (String) | All contexts |
| `DPDBucket` | bucketCode (enum), minDPD (int), maxDPD (int), label (String) | Delinquency, Workflow, Analytics |
| `DateRange` | fromDate (LocalDate), toDate (LocalDate) | All contexts |
| `ContactInfo` | mobileNumber, email, address, preferredLanguage | Communication, Account |
| `AuditMetadata` | createdBy, createdAt, modifiedBy, modifiedAt, version | All contexts |

---

## 6. UBIQUITOUS LANGUAGE -- SYSTEM-WIDE CORE TERMS

This section defines terms used across multiple bounded contexts. Each bounded context part (Parts 02-08) defines additional context-specific terms.

| Term | Definition |
|---|---|
| **DPD (Days Past Due)** | Number of calendar days elapsed since the due date of the oldest unpaid installment (or part thereof) for a loan account. Minimum value is 0. |
| **Bucket** | A classification category for a loan account based on its DPD range. Standard buckets align with RBI IRAC norms: Current (0 DPD), SMA-0 (1-30), SMA-1 (31-60), SMA-2 (61-90), NPA Sub-Standard (91-365), Doubtful (366+), Loss. |
| **Bucket Movement** | A transition of a loan account from one DPD bucket to another. Upgrade = movement to a lower DPD bucket. Downgrade = movement to a higher DPD bucket. |
| **EMI (Equated Monthly Installment)** | A fixed monthly payment comprising principal and interest portions, due on a pre-defined date each month. |
| **NACH (National Automated Clearing House)** | NPCI-operated electronic payment system for bulk recurring debits/credits. Primary mechanism for EMI auto-debit in India. |
| **UMRN (Unique Mandate Reference Number)** | A unique identifier assigned by NPCI to each NACH mandate. |
| **UPI AutoPay** | Recurring payment mandate on UPI platform (per NPCI UPI 2.0 specification). |
| **Bounce / Return** | Failure of an attempted auto-debit (NACH/ECS/UPI AutoPay) with a return reason code. |
| **PTP (Promise to Pay)** | A verbal or written commitment by a borrower to make payment by a specific date and amount. |
| **OTS (One Time Settlement)** | A negotiated settlement where the borrower pays a lump sum (typically less than total outstanding) to close the loan. Requires board-approved OTS policy. |
| **SARFAESI** | Securitisation and Reconstruction of Financial Assets and Enforcement of Security Interest Act, 2002. Enables secured creditors to enforce security interest without court intervention. |
| **NPA (Non-Performing Asset)** | A loan account classified as non-performing per RBI IRAC norms -- typically 90+ DPD for banks and NBFCs (post harmonization). |
| **SMA (Special Mention Account)** | A pre-NPA classification for accounts showing signs of stress (SMA-0: 1-30 DPD, SMA-1: 31-60 DPD, SMA-2: 61-90 DPD). |
| **CRILC (Central Repository of Information on Large Credits)** | RBI's database for tracking large credit exposures (INR 5 crore and above). SMA reporting is mandatory. |
| **IRAC (Income Recognition and Asset Classification)** | RBI's Master Direction on income recognition, asset classification, and provisioning norms. Current reference: RBI Master Direction DOR.STR.REC.4/21.04.048/2024-25 dated September 12, 2024. |
| **CER (Collection Efficiency Ratio)** | Total amount collected in a period divided by total amount due (including overdue) in that period, expressed as percentage. |
| **Roll Rate** | Percentage of accounts that move (roll) from one DPD bucket to the next worse bucket in a given period. |
| **Flow Rate** | Similar to roll rate; specifically tracks the flow of accounts across delinquency stages over time. |
| **Write-off** | Accounting treatment where an NPA is removed from the balance sheet. Technical write-off: removed from books but recovery efforts continue. Prudential write-off: fully written off. |
| **Suspense Account** | A temporary holding account for unmatched or unidentified transactions pending resolution. |
| **Disposition** | The outcome recorded after a collection interaction (e.g., PTP, Refused to Pay, Wrong Number, Not Reachable, Dispute Raised). |
| **Collection Case** | A logical grouping of all collection activities for a delinquent loan account, from first delinquency through resolution. |
| **Allocation** | Assignment of delinquent accounts to internal collection teams or external collection agencies for recovery action. |
| **Demand Notice** | A formal written communication to the borrower demanding payment of overdue amounts. Precedes legal action. |
| **FLDG (First Loss Default Guarantee)** | A guarantee arrangement in co-lending/LSP models where the originator provides a first-loss guarantee. RBI guidelines dated June 8, 2023 govern FLDG arrangements. |
| **Fair Practices Code** | RBI's code of conduct for recovery agents and collection practices. Reference: RBI Master Direction DoR.FIN.REC.61/03.10.038/2024-25 (Fair Practices Code for NBFCs) and RBI circular on use of recovery agents dated April 24, 2008. |
| **DND (Do Not Disturb)** | TRAI's Do Not Disturb registry. Customers registered on DND must not receive unsolicited commercial communications. Collection reminders for existing obligations are generally exempt but must comply with format requirements. |
| **DLT (Distributed Ledger Technology) Registration** | TRAI-mandated registration of SMS templates, headers, and senders on blockchain-based DLT platform for commercial messaging. |
| **EWS (Early Warning System)** | A predictive system that identifies accounts likely to become delinquent before actual default occurs. |
| **IndAS** | Indian Accounting Standards. IndAS 109 governs expected credit loss (ECL) provisioning for financial instruments. |
| **ECL (Expected Credit Loss)** | Forward-looking credit loss estimate under IndAS 109, replacing the incurred loss model. Relevant for provisioning calculations. |

---

## END OF PART 01

Next: **COL-BRD-Part02-Reconciliation.md** -- Reconciliation Bounded Context with full DDD structure.
