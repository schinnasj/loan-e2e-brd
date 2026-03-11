# BUSINESS REQUIREMENTS DOCUMENT
# LOAN MANAGEMENT SYSTEM (LMS)
# FOR INDIAN BANKS, NBFCs, SFBs & COOPERATIVE BANKS

---

## DOCUMENT CONTROL

| Field | Details |
|---|---|
| Document Title | Business Requirements Document -- Loan Management System |
| Document ID | LMS-BRD-2026-001 |
| Version | 1.0 |
| Status | Draft for Review |
| Classification | Confidential |
| Created Date | 10-Mar-2026 |
| Last Updated | 10-Mar-2026 |
| Author | Product Management -- Lending Technology |
| Department | Digital Banking & Lending Transformation |

### Version History

| Version | Date | Author | Change Description |
|---|---|---|---|
| 0.1 | 10-Mar-2026 | Product Management | Initial draft -- GL, Payment Processing, Interest Computation |
| 0.2 | -- | -- | Reconciliation, Late Payment, Early Closure modules |
| 0.3 | -- | -- | Interest-Principal Allocation, Origination, Collateral |
| 0.4 | -- | -- | Regulatory, NFR, Integration, Data Model |
| 1.0 | -- | -- | Final consolidated version after stakeholder review |

### Approvers and Sign-Off Matrix

| Role | Name | Department | Sign-Off Status | Date |
|---|---|---|---|---|
| Chief Technology Officer | [TBD] | Technology | Pending | -- |
| Chief Risk Officer | [TBD] | Risk Management | Pending | -- |
| Head -- Retail Lending | [TBD] | Retail Banking | Pending | -- |
| Head -- MSME/Corporate Lending | [TBD] | Wholesale Banking | Pending | -- |
| Chief Compliance Officer | [TBD] | Compliance | Pending | -- |
| Head -- Finance & Accounts | [TBD] | Finance | Pending | -- |
| Head -- Operations | [TBD] | Loan Operations | Pending | -- |
| Head -- Collections | [TBD] | Collections & Recovery | Pending | -- |
| CISO | [TBD] | Information Security | Pending | -- |
| Head -- Internal Audit | [TBD] | Audit | Pending | -- |

### Distribution List

| Name | Role | Distribution Type |
|---|---|---|
| CTO Office | Technology Decision Makers | Full Document |
| Development Teams | Implementation | Full Document |
| QA Team | Testing & Validation | Full Document |
| Business Teams | Requirement Validation | Relevant Sections |
| Compliance Team | Regulatory Review | Compliance Sections |
| External Vendor (if applicable) | System Integrator | Full Document (under NDA) |

---

## TABLE OF CONTENTS

1. Executive Summary
2. Business Context and Objectives
3. Scope Definition
4. Module 1: General Ledger (GL) Integration
5. Module 2: Loan Payment Processing
6. Module 3: Reconciliation
7. Module 4: Interest Computation
8. Module 5: Late Payment / Overdue Management
9. Module 6: Early Closure / Prepayment
10. Module 7: Interest vs Principal Allocation
11. Module 8: Loan Origination Lifecycle
12. Module 9: Collateral and Security Management
13. Module 10: Insurance Integration
14. Module 11: Regulatory Reporting
15. Module 12: Credit Bureau Integration
16. Module 13: Digital Lending Compliance
17. Module 14: Customer Communication
18. Module 15: Audit Trail and Maker-Checker Workflows
19. Module 16: Multi-Entity Architecture
20. Module 17: API-First Architecture
21. Module 18: Data Privacy and Security
22. Module 19: Disaster Recovery and Business Continuity
23. Non-Functional Requirements
24. Data Model Highlights
25. Integration Architecture
26. Regulatory Compliance Matrix
27. Assumptions, Risks, and Dependencies
28. Glossary
29. Appendices

---

## 1. EXECUTIVE SUMMARY

### 1.1 Purpose

This Business Requirements Document defines the complete functional, non-functional, regulatory, and integration requirements for building or procuring a production-grade Loan Management System (LMS) suitable for deployment across Indian financial institutions including Scheduled Commercial Banks (SCBs), Small Finance Banks (SFBs), Urban Cooperative Banks (UCBs), Regional Rural Banks (RRBs), Non-Banking Financial Companies (NBFCs including NBFC-UL, NBFC-ML, NBFC-BL per RBI's Scale-Based Regulation), and Housing Finance Companies (HFCs) regulated by the National Housing Bank (NHB).

### 1.2 Business Context

The Indian lending ecosystem has undergone fundamental transformation driven by:

- **RBI's Digital Lending Guidelines (September 2, 2022)**: Mandating direct disbursement to borrower accounts, KFS (Key Fact Statement) disclosure, cooling-off periods, and restrictions on First Loss Default Guarantee (FLDG) arrangements (capped at 5% per RBI circular DOR.FIN.REC.No.20/03.10.038/2023-24 dated June 8, 2023).
- **Scale-Based Regulation (SBR) for NBFCs**: Effective October 1, 2022, classifying NBFCs into Base Layer, Middle Layer, Upper Layer, and Top Layer with progressively stringent requirements.
- **IND AS Implementation**: IND AS 109 (Financial Instruments) requiring Expected Credit Loss (ECL) model replacing the incurred loss model, impacting provisioning, staging, and GL accounting.
- **Account Aggregator Framework**: Enabling consent-based financial data sharing under RBI's NBFC-AA guidelines.
- **MCLR to EBLR Transition**: RBI's mandate for external benchmark-linked lending rates for specific loan categories (effective October 1, 2019 for new floating rate personal, retail, and MSME loans by banks).
- **CERSAI Digitization**: Mandatory registration of equitable mortgages and charges on the Central Registry.
- **Enhanced SMA/NPA Reporting**: CRILC (Central Repository of Information on Large Credits) reporting thresholds and SMA classification requirements.

### 1.3 Strategic Objectives

| Objective ID | Objective | Measurable Target |
|---|---|---|
| SO-01 | Automate end-to-end loan lifecycle from origination to closure | 90% straight-through processing for eligible products |
| SO-02 | Ensure 100% regulatory compliance across RBI, NHB, and SEBI norms | Zero regulatory penalties; 100% timely return filing |
| SO-03 | Support multi-product lending across retail, MSME, corporate, and agricultural segments | Minimum 15 loan product types configurable without code changes |
| SO-04 | Enable real-time GL integration with double-entry accounting | T+0 GL posting for all loan events; zero reconciliation breaks |
| SO-05 | Achieve sub-second response times for customer-facing operations | 95th percentile response time under 2 seconds |
| SO-06 | Support regulatory reporting (CRILC, BSR, XBRL, SMA) with automated data extraction | 100% automated data extraction; T+1 reporting readiness |
| SO-07 | Enable digital lending partnerships with LSPs/fintech platforms via APIs | API response times under 500ms; 99.9% API availability |
| SO-08 | Implement IND AS 109 compliant ECL provisioning engine | Automated ECL computation with daily staging assessment |

### 1.4 Problem Statement

Current legacy loan management systems in many Indian banks and NBFCs suffer from:

1. **Fragmented Architecture**: Separate systems for origination (LOS), management (LMS), and collections with manual data handoffs and reconciliation gaps.
2. **Rigid Product Configuration**: Adding new loan products or modifying business rules requires months of development effort.
3. **Regulatory Lag**: Manual processes for regulatory return generation leading to delayed filings and inaccuracies.
4. **Poor Interest Computation Accuracy**: Inability to handle complex interest scenarios such as benchmark rate resets, moratorium interest, IoI calculations, and IND AS 109 effective interest rate (EIR) computations.
5. **Weak GL Integration**: Batch-mode GL posting causing end-of-day reconciliation issues and delayed financial reporting.
6. **Limited Digital Capabilities**: Lack of APIs for digital lending partnerships, Account Aggregator integration, and real-time bureau checks.
7. **Manual Reconciliation**: Heavy manual effort for NACH return processing, payment gateway reconciliation, and suspense account clearing.

### 1.5 Target Audience

This BRD is intended for:

- Technology teams (architects, developers, QA) responsible for system implementation
- System integrators and vendor partners
- Business stakeholders in lending, operations, finance, risk, and compliance
- Regulatory and audit teams for compliance validation
- Project management teams for planning and execution

---

## 2. BUSINESS CONTEXT AND OBJECTIVES

### 2.1 Current State (AS-IS) Pain Points

| Pain Point ID | Description | Impact | Affected Stakeholders |
|---|---|---|---|
| PP-01 | Manual GL posting with batch processing causes T+1 delay in financial reporting | Delayed P&L reporting; reconciliation breaks | Finance, Audit |
| PP-02 | Interest computation engine cannot handle EBLR resets mid-cycle | Incorrect interest charges; customer complaints; regulatory risk | Operations, Customers, Compliance |
| PP-03 | No automated SMA classification and CRILC reporting | Manual data compilation prone to errors; delayed CRILC submissions | Risk, Compliance |
| PP-04 | Payment allocation rules are hardcoded, not configurable per product | Different products need different allocation waterfalls; change requests take weeks | Operations, Technology |
| PP-05 | No integrated digital lending compliance framework | KFS not auto-generated; cooling-off period not system-enforced; FLDG tracking manual | Compliance, Partnerships |
| PP-06 | Collateral management on spreadsheets | No automated CERSAI filing; collateral revaluation tracking manual; security release delays | Operations, Legal |
| PP-07 | Manual NACH mandate management and bounce processing | High bounce rate management effort; delayed follow-up; poor recovery | Collections, Operations |
| PP-08 | No integrated ECL engine for IND AS 109 | Quarterly provisioning done in Excel; staging assessment manual | Finance, Risk, Audit |

### 2.2 Target State (TO-BE) Architecture Vision

The target LMS shall be a modular, microservices-based (or modular monolith) platform with the following characteristics:

1. **Product-Agnostic Core Engine**: A configurable rules engine that supports any loan product type through parameter-driven configuration rather than code changes.
2. **Real-Time GL Integration**: Event-driven GL posting with real-time sub-ledger updates and automated reconciliation.
3. **Regulatory-First Design**: Built-in compliance for RBI, NHB, SEBI, and DPDP Act requirements with automated regulatory return generation.
4. **API-First Architecture**: RESTful APIs for every business function enabling integration with LOS, digital lending platforms, Account Aggregators, bureau systems, payment systems, and core banking.
5. **Multi-Entity Support**: Single platform deployment supporting multiple legal entities, branches, products, and currencies.
6. **Cloud-Ready Deployment**: Deployable on private cloud, public cloud (with RBI data localization compliance), or hybrid infrastructure.

### 2.3 Regulatory Landscape Summary

| Regulation | Applicable To | Key Requirements for LMS | Reference |
|---|---|---|---|
| RBI Master Direction on IRAC Norms | Banks, NBFCs | 90-day NPA recognition; asset classification; provisioning norms | DOR.STR.REC.4/21.04.048/2021-22 |
| RBI Digital Lending Guidelines | Banks, NBFCs, LSPs | KFS disclosure; direct disbursement; cooling-off; FLDG cap | DOR.CRE.REC.66/21.07.001/2022-23 |
| IND AS 109 | Banks (Phase-wise), NBFCs (as applicable) | ECL model; 3-stage classification; EIR method; fair value measurement | MoCA notification; RBI implementation roadmap |
| NHB Directions | HFCs | Housing loan specific IRAC; LTV norms; NHB return filing | NHB(ND)/DRS/Policy Circular |
| CERSAI Registration | All secured lenders | Mandatory filing within 30 days of creation of security interest | CERSAI Act 2011; RBI circular |
| SARFAESI Act | Banks, select NBFCs (asset > 100 Cr) | Enforcement of security interest for NPAs > Rs 1 lakh (Rs 20 lakh for NBFCs) | SARFAESI Act 2002 |
| RBI KYC Master Direction | All REs | CKYC; V-CIP; Aadhaar eKYC; periodic KYC updates | DOR.AML.REC.10/14.01.001/2023-24 |
| Scale-Based Regulation | NBFCs | Layer-wise regulatory requirements; capital adequacy; governance | DOR.CRE.REC.60/03.10.001/2021-22 |
| Fair Practices Code | NBFCs | Loan agreement terms; prepayment transparency; grievance redressal | DOR.FIN.HFC.CC.No.120/03.10.136/2020-21 |
| RBI Framework on Resolution of Stressed Assets | Banks, NBFCs | Resolution timeline; ICA provisions; additional provisioning for delays | DOR.STR.REC.21/21.04.048/2018-19 (updated) |
| TDS on Interest (Sec 194A) | All | TDS deduction on interest paid exceeding threshold | Income Tax Act, 1961 |
| GST on Financial Services | All | GST on processing fees, penal charges (post April 2024 penal charges circular), documentation charges | CGST Act; RBI circular on penal charges |
| RBI Circular on Penal Charges | Banks, NBFCs | Penal charges (not penal interest) on loan defaults; no compounding; reasonable and transparent | DoR.MCS.REC.28/01.01.001/2023-24 dated Aug 18, 2023; effective April 1, 2024 |

---

## 3. SCOPE DEFINITION

### 3.1 In-Scope

| Scope ID | Module/Feature | Description |
|---|---|---|
| SC-01 | General Ledger Integration | Complete double-entry GL posting for all loan events; IND AS 109 entries; sub-ledger reconciliation |
| SC-02 | Loan Payment Processing | EMI calculation; amortization; payment allocation; all payment modes; moratorium; restructuring |
| SC-03 | Reconciliation | Bank statement, NACH, payment gateway, GL, suspense, inter-branch, escrow reconciliation |
| SC-04 | Interest Computation | All interest methods; floating rate management; accrual; NPA reversal; subvention; TDS |
| SC-05 | Late Payment/Overdue Management | DPD engine; NPA classification; provisioning; SARFAESI; collections; OTS; write-off |
| SC-06 | Early Closure/Prepayment | Foreclosure; part-prepayment; penalty; NOC; CERSAI satisfaction; security release |
| SC-07 | Interest vs Principal Allocation | Amortization split; tax certificates; SOA generation; regulatory reporting split |
| SC-08 | Loan Origination Lifecycle | Application to disbursement; credit appraisal; sanction; documentation |
| SC-09 | Collateral/Security Management | Property, gold, FD, shares; valuation; CERSAI; lien marking; release |
| SC-10 | Insurance Integration | Credit life; property; crop; premium tracking; claims |
| SC-11 | Regulatory Reporting | BSR, XBRL, SMA, CRILC, CPC, NHB returns, CERSAI reports |
| SC-12 | Credit Bureau Integration | CIBIL, Experian, CRIF, Equifax; inquiry and reporting |
| SC-13 | Digital Lending Compliance | KFS; cooling-off; FLDG tracking; LSP management |
| SC-14 | Customer Communication | Statements; rate change notices; demand letters; SMS/email/WhatsApp |
| SC-15 | Audit Trail & Maker-Checker | Complete audit logging; configurable approval workflows |
| SC-16 | Multi-Entity Architecture | Multi-branch, multi-product, multi-currency support |
| SC-17 | API Architecture | RESTful APIs; webhook support; API gateway; rate limiting |
| SC-18 | Security & Privacy | Encryption; access control; DPDP Act; RBI cyber security framework |
| SC-19 | DR/BCP | Disaster recovery; business continuity; RPO/RTO targets |

### 3.2 Out of Scope

| Item | Rationale |
|---|---|
| Core Banking System (CBS) replacement | LMS integrates with existing CBS; does not replace it |
| Treasury management and investment portfolio | Separate system; LMS provides GL feeds only |
| HR and payroll systems | Not related to loan management |
| ATM/POS/Card management | Separate channel systems |
| Trade finance and LC/BG management | Separate trade finance system |
| Fixed deposit and savings account management | CBS functionality |
| Anti-money laundering (AML) transaction monitoring | Separate AML system; LMS provides data feeds |
| Customer Relationship Management (CRM) | Separate CRM; LMS integrates via APIs |

### 3.3 Loan Product Types in Scope

| Category | Products |
|---|---|
| Retail Lending | Home Loan, Home Loan Balance Transfer, Loan Against Property (LAP), Personal Loan, Consumer Durable Loan, Education Loan, Vehicle Loan (Car, Two-Wheeler, Commercial Vehicle), Gold Loan, Loan Against FD, Loan Against Securities, Credit Card EMI Conversion |
| MSME Lending | MSME Term Loan, MSME Working Capital (Cash Credit/Overdraft), Mudra Loans (Shishu/Kishore/Tarun), CGTMSE-backed Loans, Invoice Financing, Supply Chain Finance |
| Agricultural Lending | Kisan Credit Card (KCC), Crop Loan, Agricultural Term Loan, Allied Activity Loan, Warehouse Receipt Finance |
| Corporate Lending | Term Loan, Working Capital, Consortium Lending, Syndicated Loan, External Commercial Borrowing (ECB) servicing |
| Co-Lending | Co-Lending Model (CLM) as per RBI guidelines; NBFC-Bank partnership; 80:20 split management |
| Digital Lending | BNPL (Buy Now Pay Later), Digital Personal Loan, Pre-approved Loans, Sachet Loans (small ticket) |
| Microfinance | JLG (Joint Liability Group) Loans, SHG (Self Help Group) Loans as per RBI Microfinance Guidelines (March 2022) |

---

## 4. MODULE 1: GENERAL LEDGER (GL) INTEGRATION

### 4.1 Overview

The GL Integration module ensures that every financial event in the loan lifecycle results in accurate, auditable, real-time double-entry accounting entries posted to the institution's General Ledger. This module must comply with IND AS 109 (Financial Instruments), RBI's guidelines on accounting standards, and the institution's internal accounting policies.

### 4.2 GL Account Structure

#### 4.2.1 Chart of Accounts Framework

**Requirement ID: LMS-GL-001**
**Title:** Configurable Chart of Accounts for Loan Products
**Priority:** Must Have
**Description:** The system shall maintain a hierarchical Chart of Accounts (CoA) structure that maps every loan product and event type to specific GL account codes. The CoA must be configurable without code deployment.

**Business Rules:**
- GL account codes shall follow a structured format: [Entity Code]-[Branch Code]-[Product Code]-[Account Type]-[Sub-Account]-[Currency]
- Example: 01-0001-HL01-PROUT-00-INR (Entity 01, Branch 0001, Home Loan Product 01, Principal Outstanding, INR)
- The system must support a minimum 20-digit GL account code structure
- Each loan product must have a complete GL account set mapped at the time of product configuration
- GL accounts must be tagged with: Profit Center, Cost Center, Business Segment, Regulatory Classification (PSL/Non-PSL)

**Acceptance Criteria:**
1. Administrator can create and modify GL account mappings through a configuration UI
2. System prevents loan disbursement if required GL accounts are not mapped for the product
3. GL account structure supports at least 6 hierarchical levels
4. Audit trail captures all GL mapping changes with maker-checker approval

#### 4.2.2 Standard GL Account Heads for Loan Products

**Requirement ID: LMS-GL-002**
**Title:** Mandatory GL Account Heads per Loan Product
**Priority:** Must Have
**Description:** Every loan product must have the following GL account heads configured. The system shall not allow product activation without complete GL mapping.

**GL Account Head Master:**

| GL Head Category | GL Head Name | Nature | Normal Balance | Description |
|---|---|---|---|---|
| **Asset -- Loan Outstanding** | | | | |
| A01 | Principal Outstanding -- Performing | Asset | Debit | Outstanding principal for standard/performing loans |
| A02 | Principal Outstanding -- SMA | Asset | Debit | Outstanding principal for SMA-0/1/2 accounts |
| A03 | Principal Outstanding -- NPA Sub-Standard | Asset | Debit | Outstanding principal for sub-standard NPA |
| A04 | Principal Outstanding -- NPA Doubtful (D1/D2/D3) | Asset | Debit | Outstanding principal for doubtful category |
| A05 | Principal Outstanding -- NPA Loss | Asset | Debit | Outstanding principal for loss category |
| A06 | Interest Receivable -- Performing | Asset | Debit | Accrued interest on performing loans |
| A07 | Interest Receivable -- NPA (Memorandum) | Off-BS | Debit | Interest accrued on NPA -- tracked off-balance sheet |
| A08 | Penal Charges Receivable | Asset | Debit | Outstanding penal charges (post Apr 2024 RBI norms) |
| A09 | Fees Receivable | Asset | Debit | Outstanding processing/documentation fees |
| A10 | Pre-EMI Interest Receivable | Asset | Debit | PEMI receivable for under-construction HL |
| A11 | Suspense -- Loan Collections | Asset | Debit | Unidentified/unallocated loan collections |
| A12 | Sundry Advances -- Loan Related | Asset | Debit | Insurance advances, legal fee advances |
| **Contra-Asset -- Provisions** | | | | |
| P01 | Provision for Standard Assets | Contra-Asset | Credit | Provision on performing book (0.40% for banks; as per NBFC norms) |
| P02 | Provision for SMA | Contra-Asset | Credit | Additional provision if applicable |
| P03 | Provision for NPA -- Sub-Standard | Contra-Asset | Credit | 15% (secured) / 25% (unsecured) for banks |
| P04 | Provision for NPA -- Doubtful D1 | Contra-Asset | Credit | 25% of secured + 100% of unsecured |
| P05 | Provision for NPA -- Doubtful D2 | Contra-Asset | Credit | 40% of secured + 100% of unsecured |
| P06 | Provision for NPA -- Doubtful D3 | Contra-Asset | Credit | 100% |
| P07 | Provision for NPA -- Loss | Contra-Asset | Credit | 100% |
| P08 | ECL Provision -- Stage 1 | Contra-Asset | Credit | 12-month ECL (IND AS 109) |
| P09 | ECL Provision -- Stage 2 | Contra-Asset | Credit | Lifetime ECL (IND AS 109) |
| P10 | ECL Provision -- Stage 3 | Contra-Asset | Credit | Lifetime ECL -- credit impaired (IND AS 109) |
| **Liability** | | | | |
| L01 | EMI Collection Clearing | Liability | Credit | Temporary holding for EMI collections before allocation |
| L02 | Excess Collection / Overpayment | Liability | Credit | Customer overpayments pending refund or adjustment |
| L03 | Insurance Premium Payable | Liability | Credit | Collected insurance premiums pending remittance |
| L04 | TDS Payable on Interest | Liability | Credit | TDS deducted under 194A pending deposit |
| L05 | GST Payable on Fees | Liability | Credit | GST collected on processing/documentation fees |
| L06 | Undisbursed Loan Commitment | Off-BS | Credit | Sanctioned but undisbursed loan amounts |
| **Income** | | | | |
| I01 | Interest Income -- Performing Loans | Income | Credit | Interest income recognized on performing book |
| I02 | Processing Fee Income | Income | Credit | Upfront processing fee income (or amortized under IND AS) |
| I03 | Documentation Charge Income | Income | Credit | Documentation and legal charge income |
| I04 | Penal Charge Income | Income | Credit | Penal charges on defaults (non-interest, post Apr 2024) |
| I05 | Prepayment/Foreclosure Fee Income | Income | Credit | Foreclosure penalty income |
| I06 | Late Payment Fee Income | Income | Credit | Late payment charges |
| I07 | Interest Income -- Recovered from NPA | Income | Credit | Interest recovered from NPA accounts |
| I08 | Miscellaneous Fee Income | Income | Credit | Cheque bounce charges, statement charges, etc. |
| I09 | Recovery from Written-Off Accounts | Income | Credit | Recoveries post write-off |
| **Expense** | | | | |
| E01 | Provision Expense -- Standard Assets | Expense | Debit | Provisioning charge for standard book |
| E02 | Provision Expense -- NPA | Expense | Debit | Provisioning charge for NPA classification |
| E03 | ECL Provision Expense | Expense | Debit | ECL provisioning charge (IND AS 109) |
| E04 | Write-Off Expense | Expense | Debit | Loan write-off charge |
| E05 | Interest Subvention Claim Receivable | Asset | Debit | Government subvention amount receivable (PMAY, etc.) |
| E06 | Interest Reversal on NPA | Expense (or contra-income) | Debit | Reversal of interest income on NPA classification |

**Acceptance Criteria:**
1. System validates completeness of GL mapping before product activation
2. Missing GL heads trigger a validation error listing all unmapped accounts
3. GL heads support tagging with regulatory classification (PSL, non-PSL, weaker section, etc.)
4. System supports entity-specific GL head overrides for multi-entity deployment

#### 4.2.3 Sub-Ledger Architecture

**Requirement ID: LMS-GL-003**
**Title:** Loan Sub-Ledger with Real-Time GL Posting
**Priority:** Must Have
**Description:** Each individual loan account shall function as a sub-ledger account. The system must maintain detailed transactional records at the individual loan level and aggregate postings to GL account heads in real-time or near-real-time (within configurable threshold, default 5 seconds).

**Business Rules:**
- Every financial transaction on a loan account generates a sub-ledger entry with a unique transaction reference
- Sub-ledger entries are aggregated by GL head, branch, product, and currency for GL posting
- GL posting can be configured as: (a) Real-time per transaction, (b) Micro-batched (configurable interval, e.g., every 5 minutes), or (c) End-of-day batch
- The sub-ledger balance for any GL head must always equal the sum of individual loan account balances for that head
- Reconciliation between sub-ledger and GL must be automated and run at minimum daily

**Acceptance Criteria:**
1. Zero-break tolerance between sub-ledger totals and GL balances at end of each business day
2. System generates automatic reconciliation reports with exception flagging
3. Any discrepancy triggers an alert to the operations team within 30 minutes of detection
4. Sub-ledger supports drill-down from GL balance to individual loan transactions

### 4.3 Double-Entry Accounting for Loan Events

**Requirement ID: LMS-GL-004**
**Title:** Double-Entry GL Posting Rules for All Loan Lifecycle Events
**Priority:** Must Have
**Description:** The system shall execute double-entry accounting entries for every financial event in the loan lifecycle. Each posting rule must be pre-configured per product and event type, with the ability to override at entity level.

#### 4.3.1 Loan Disbursement

**Event: Full Disbursement (Direct to Customer Account)**

| Leg | Account | Debit (Dr) | Credit (Cr) | Description |
|---|---|---|---|---|
| 1 | Principal Outstanding -- Performing (A01) | Loan Amount | | Loan asset created |
| 2 | Customer's Bank Account / Disbursement Clearing | | Loan Amount | Funds released to customer |

**Example Calculation:**
- Loan Amount Sanctioned: Rs 50,00,000 (Home Loan)
- Processing Fee: Rs 25,000 + GST (18%) = Rs 29,500
- Documentation Charges: Rs 5,000 + GST = Rs 5,900
- CERSAI Charges: Rs 500
- Stamp Duty (collected for e-stamping): Rs 10,000

**Disbursement Entries:**

| Leg | Account | Debit | Credit |
|---|---|---|---|
| 1 | Principal Outstanding -- Performing | Rs 50,00,000 | |
| 2 | Disbursement Clearing / Customer Account | | Rs 49,54,100 |
| 3 | Processing Fee Income | | Rs 25,000 |
| 4 | GST Payable -- CGST | | Rs 2,250 |
| 5 | GST Payable -- SGST | | Rs 2,250 |
| 6 | Documentation Charge Income | | Rs 5,000 |
| 7 | GST Payable -- CGST (Doc Charges) | | Rs 450 |
| 8 | GST Payable -- SGST (Doc Charges) | | Rs 450 |
| 9 | CERSAI Charges Recoverable | | Rs 500 |
| 10 | Stamp Duty Payable | | Rs 10,000 |

Note: Net disbursement to customer = Rs 50,00,000 - Rs 29,500 - Rs 5,900 - Rs 500 - Rs 10,000 = Rs 49,54,100

**IND AS 109 Adjustment -- Upfront Fee Amortization:**

Under IND AS 109, processing fees must be amortized over the loan tenure using the Effective Interest Rate (EIR) method rather than recognized upfront. The system must support both methods (regulatory books may use upfront recognition while IND AS books use EIR amortization).

| Leg | Account | Debit | Credit | Book |
|---|---|---|---|---|
| 1 | Processing Fee Income | Rs 25,000 | | IND AS Book -- reversal of upfront recognition |
| 2 | Deferred Fee Income (Liability) | | Rs 25,000 | IND AS Book -- to be amortized over tenure |

Monthly EIR amortization entry (assuming 240 months, simplified straight-line for illustration; actual EIR calculation required):

| Leg | Account | Debit | Credit |
|---|---|---|---|
| 1 | Deferred Fee Income | Rs 104.17 | |
| 2 | Interest Income (EIR adjustment) | | Rs 104.17 |

**Event: Partial/Tranche Disbursement (Under-Construction Home Loan)**

**Requirement ID: LMS-GL-005**
**Title:** Tranche Disbursement Accounting
**Priority:** Must Have

For under-construction property loans, disbursement happens in tranches linked to construction milestones.

| Leg | Account | Debit | Credit |
|---|---|---|---|
| 1 | Principal Outstanding -- Performing | Tranche Amount | |
| 2 | Disbursement Clearing / Builder Account | | Tranche Amount |

**Business Rules for Tranche Disbursement:**
- Undisbursed commitment amount (Sanctioned - Disbursed) tracked in off-balance sheet GL (L06)
- Pre-EMI interest charged only on disbursed amount
- Full EMI commencement only after final tranche or as per loan agreement
- System must track construction stage-wise disbursement with milestone mapping
- Maximum number of tranches configurable per product (typically 4-8 for home loans)

#### 4.3.2 Interest Accrual

**Requirement ID: LMS-GL-006**
**Title:** Daily Interest Accrual with Periodic GL Posting
**Priority:** Must Have

**Daily Accrual Entry (for performing loans):**

| Leg | Account | Debit | Credit |
|---|---|---|---|
| 1 | Interest Receivable -- Performing (A06) | Daily Interest | |
| 2 | Interest Income -- Performing (I01) | | Daily Interest |

**Example:**
- Loan Outstanding: Rs 48,50,000
- Interest Rate: 8.50% p.a.
- Day Count Convention: Actual/365
- Daily Interest = Rs 48,50,000 x 8.50% / 365 = Rs 1,129.45

**Business Rules:**
- Interest accrual runs as part of End-of-Day (EOD) batch for all active loan accounts
- Accrual date is the business date (not system date) to handle holiday adjustments
- For NPA accounts, interest accrual continues but is posted to memorandum (off-balance sheet) account, not income
- Interest income recognized to P&L only for performing accounts
- Month-end accrual must account for exact number of days in the month

**NPA Account -- Interest Accrual Reversal:**

**Requirement ID: LMS-GL-007**
**Title:** Interest Income Reversal on NPA Classification
**Priority:** Must Have

When a loan is classified as NPA (90+ DPD as per IRAC norms), all unrealized interest income must be reversed.

| Leg | Account | Debit | Credit | Description |
|---|---|---|---|---|
| 1 | Interest Income -- Performing (I01) | Unrealized Interest | | Reversal of income recognized |
| 2 | Interest Receivable -- Performing (A06) | | Unrealized Interest | Remove from performing receivable |
| 3 | Interest Receivable -- NPA Memorandum (A07) | Unrealized Interest | | Track in off-BS memorandum |

**Business Rules:**
- Interest already collected (cash basis) is not reversed
- Only accrued but unrealized interest is reversed
- Reversal applies to both the current period interest and any previous period uncollected interest
- For agricultural loans, NPA norms differ: classified as NPA at the end of the crop season for short-duration crops (refer RBI Master Direction on IRAC, Section 4.2.4)
- For NBFCs in the Base Layer: NPA recognition was 180 days until March 31, 2024; now aligned to 90 days per RBI circular DOR.ACC.REC.No.20/21.04.018/2021-22

#### 4.3.3 EMI Repayment

**Requirement ID: LMS-GL-008**
**Title:** GL Entries for EMI Receipt and Allocation
**Priority:** Must Have

**Step 1: EMI Collection Receipt**

| Leg | Account | Debit | Credit |
|---|---|---|---|
| 1 | EMI Collection Clearing (L01) or Bank Account | EMI Amount | |
| 2 | Customer Loan Account (Sub-Ledger) | | EMI Amount |

**Step 2: EMI Allocation (Interest + Principal)**

Assuming EMI = Rs 44,138 (for Rs 50L loan at 8.5% for 20 years), Month 1 split:
- Interest Component: Rs 35,417
- Principal Component: Rs 8,721

| Leg | Account | Debit | Credit |
|---|---|---|---|
| 1 | Interest Receivable -- Performing (A06) | | Rs 35,417 |
| 2 | Principal Outstanding -- Performing (A01) | | Rs 8,721 |
| 3 | EMI Collection Clearing / Bank | Rs 44,138 | |

Note: If interest was already accrued daily during the month, the interest receivable already has the accrued balance. The collection simply reduces the receivable.

#### 4.3.4 NPA Provisioning

**Requirement ID: LMS-GL-009**
**Title:** GL Entries for NPA Provisioning as per IRAC Norms
**Priority:** Must Have

**Standard Asset Provisioning (performing book):**

| Asset Category | Provision Rate -- Banks (SCBs) | Provision Rate -- NBFCs (NBFC-ML/UL) |
|---|---|---|
| Standard -- General | 0.40% | 0.40% |
| Standard -- CRE (Commercial Real Estate) | 1.00% | 0.50% |
| Standard -- CRE-RH (Residential Housing) | 0.75% | 0.40% |
| Standard -- Restructured | 5.00% (for first 2 years) | 5.00% |
| Standard -- MSME Restructured (special dispensation) | As per applicable RBI circular | As per applicable RBI circular |

**NPA Provisioning Rates -- Banks (SCBs):**

| NPA Category | Period | Secured Provision | Unsecured Provision |
|---|---|---|---|
| Sub-Standard | Up to 12 months from NPA date | 15% | 25% |
| Doubtful -- D1 | 12-24 months from NPA date | 25% of secured + 100% unsecured | 100% |
| Doubtful -- D2 | 24-48 months from NPA date | 40% of secured + 100% unsecured | 100% |
| Doubtful -- D3 | Beyond 48 months from NPA date | 100% | 100% |
| Loss | As identified | 100% | 100% |

**Provisioning GL Entry (example: loan moving to Sub-Standard NPA):**

Loan Outstanding: Rs 48,00,000 (Secured Home Loan)
Required Provision: 15% of Rs 48,00,000 = Rs 7,20,000
Existing Standard Asset Provision (0.40%): Rs 19,200
Additional Provision Required: Rs 7,20,000 - Rs 19,200 = Rs 7,00,800

| Leg | Account | Debit | Credit |
|---|---|---|---|
| 1 | Provision Expense -- NPA (E02) | Rs 7,00,800 | |
| 2 | Provision for NPA -- Sub-Standard (P03) | | Rs 7,00,800 |
| 3 | Provision for Standard Assets (P01) | Rs 19,200 | |
| 4 | Provision Expense -- Standard (E01) | | Rs 19,200 |

Simultaneously, reclassify the principal outstanding:

| Leg | Account | Debit | Credit |
|---|---|---|---|
| 1 | Principal Outstanding -- NPA Sub-Standard (A03) | Rs 48,00,000 | |
| 2 | Principal Outstanding -- Performing (A01) | | Rs 48,00,000 |

**Acceptance Criteria:**
1. System automatically computes provision amount based on asset classification, secured/unsecured status, and collateral value
2. Provision entries are generated during EOD batch when asset classification changes
3. System maintains provision movement schedule (opening, additions, write-backs, write-offs, closing)
4. Provision rates are configurable per product, entity, and regulatory type (RBI norms vs internal policy if more conservative)
5. System supports both IRAC-based provisioning and IND AS 109 ECL-based provisioning in parallel (dual reporting)

#### 4.3.5 Loan Write-Off

**Requirement ID: LMS-GL-010**
**Title:** GL Entries for Loan Write-Off (Technical and Actual)
**Priority:** Must Have

**Technical Write-Off (removed from books but recovery efforts continue):**

| Leg | Account | Debit | Credit |
|---|---|---|---|
| 1 | Write-Off Expense (E04) | Net Outstanding | |
| 2 | Principal Outstanding -- NPA Loss (A05) | | Principal Outstanding |
| 3 | Provision for NPA -- Loss (P07) | Outstanding Provision | |
| 4 | Write-Off Expense (E04) | | Outstanding Provision |

Net write-off expense = Outstanding - Provision already held

Simultaneously, create off-balance sheet tracking entry:

| Leg | Account | Debit | Credit |
|---|---|---|---|
| 1 | Written-Off Assets -- Memorandum (Off-BS) | Full Written-Off Amount | |

**Recovery Post Write-Off:**

| Leg | Account | Debit | Credit |
|---|---|---|---|
| 1 | Bank Account / Collection Account | Recovery Amount | |
| 2 | Recovery from Written-Off Accounts (I09) | | Recovery Amount |
| 3 | Written-Off Assets -- Memorandum (Off-BS) | | Recovery Amount |

**Business Rules:**
- Board-approved write-off policy must be configured in the system
- Write-off authority matrix (based on amount thresholds) must be enforced
- Written-off accounts continue to be tracked for recovery in a separate module
- Written-off accounts must continue to be reported to credit bureaus as "Written-Off"
- Tax implications of write-off (Section 36(1)(vii) of Income Tax Act) must be tracked
- System must prevent write-off without minimum required provisioning (100% provision for loss assets)

#### 4.3.6 IND AS 109 -- ECL Staging and Fair Value Entries

**Requirement ID: LMS-GL-011**
**Title:** IND AS 109 Expected Credit Loss Entries
**Priority:** Must Have (for entities that have adopted IND AS)

**Stage Classification:**

| Stage | Criteria | ECL Measurement | Income Recognition |
|---|---|---|---|
| Stage 1 | No significant increase in credit risk since origination; 0-30 DPD (rebuttable presumption) | 12-month ECL | EIR on gross carrying amount |
| Stage 2 | Significant increase in credit risk; 31-90 DPD (rebuttable presumption); or qualitative triggers | Lifetime ECL | EIR on gross carrying amount |
| Stage 3 | Credit-impaired; 90+ DPD; or specific triggers (death, fraud, SARFAESI initiated) | Lifetime ECL | EIR on net carrying amount (gross - ECL) |

**ECL Provision Increase Entry (Stage 1 to Stage 2 migration):**

| Leg | Account | Debit | Credit |
|---|---|---|---|
| 1 | ECL Provision Expense (E03) | Incremental ECL | |
| 2 | ECL Provision -- Stage 1 (P08) | Previous Stage 1 ECL | |
| 3 | ECL Provision -- Stage 2 (P09) | | New Stage 2 ECL |

**Business Rules:**
- ECL calculation requires: Probability of Default (PD), Loss Given Default (LGD), Exposure at Default (EAD)
- PD models must be calibrated using historical default data (minimum 5-year history recommended)
- Forward-looking macroeconomic scenarios must be incorporated (GDP growth, unemployment rate, property price index, etc.)
- System must support multiple ECL scenarios (base, optimistic, pessimistic) with probability-weighted average
- Staging assessment must run daily as part of EOD
- System must support both: (a) automated staging based on DPD/quantitative triggers, and (b) manual staging override with maker-checker approval and documented rationale
- Stage 2 triggers (qualitative) include: restructuring, 30+ DPD, significant adverse change in borrower's financial condition, industry downturn
- Backstop indicator: 30 DPD for Stage 2 is a rebuttable presumption per IND AS 109 para 5.5.11

**Acceptance Criteria:**
1. System computes ECL for all three stages using configurable PD, LGD, EAD models
2. Staging migration is automated with daily assessment
3. Manual override supported with full audit trail and approval workflow
4. Parallel computation: IRAC provisioning and IND AS 109 ECL
5. Regulatory reporting extracts available for both frameworks

### 4.4 Day-End and Month-End GL Posting Rules

**Requirement ID: LMS-GL-012**
**Title:** EOD and Month-End GL Processing Rules
**Priority:** Must Have

**End-of-Day (EOD) Processing Sequence:**

| Step | Process | Description | Dependency |
|---|---|---|---|
| 1 | Business Date Validation | Confirm business date; process holiday calendar | None |
| 2 | Transaction Cut-Off | No new transactions after EOD initiation | Step 1 |
| 3 | Interest Accrual | Daily interest accrual for all active loans | Step 2 |
| 4 | DPD Update | Update Days Past Due for all accounts | Step 2 |
| 5 | Asset Classification | Run NPA classification engine; identify new NPAs, upgrades, downgrades | Step 4 |
| 6 | Provisioning | Compute provisions (IRAC and ECL) for classification changes | Step 5 |
| 7 | GL Posting | Post all accrual, classification, and provisioning entries to GL | Steps 3-6 |
| 8 | Sub-Ledger Reconciliation | Reconcile sub-ledger totals with GL balances | Step 7 |
| 9 | Exception Report Generation | Generate reports for breaks, unprocessed transactions, suspense entries | Step 8 |
| 10 | Business Date Roll | Advance business date to next working day | Step 9 (all clear) |

**Month-End Additional Processing:**

| Process | Description |
|---|---|
| Interest Application | Apply accrued interest for the month (for products with monthly interest application) |
| Floating Rate Reset | Process any pending benchmark rate changes effective during the month |
| Provision True-Up | Month-end provision reconciliation and adjustment |
| P&L Closure | Interest income, fee income, provisioning expense aggregation |
| Regulatory Return Data Extraction | Extract data for monthly regulatory returns (BSR-1, BSR-2, etc.) |
| IND AS Adjustments | EIR amortization, fair value adjustments, stage migration finalization |
| MIS Report Generation | Portfolio quality reports, product-wise profitability, segment-wise analysis |

**Business Rules:**
- EOD must be idempotent: re-running EOD for the same date must not create duplicate entries
- Failed EOD steps must not block subsequent steps unless there is a hard dependency (as defined in the sequence)
- System must support EOD roll-back to previous business date with full reversal of entries (with appropriate authorization)
- Month-end closing must have a configurable hard-close date (typically T+2 to T+5 from month end)
- Prior-period adjustments post hard-close must be routed through a separate adjustment entry workflow with enhanced authorization

### 4.5 Multi-Currency GL Handling

**Requirement ID: LMS-GL-013**
**Title:** Multi-Currency Loan Accounting
**Priority:** Should Have
**Description:** For ECB (External Commercial Borrowing) servicing, FCNR loans, and cross-border lending scenarios, the system must support multi-currency GL accounting.

**Business Rules:**
- Loan can be denominated in foreign currency (USD, EUR, GBP, JPY, etc.)
- GL entries posted in both transaction currency and reporting currency (INR)
- Exchange rate source: RBI reference rate or FEDAI rates (configurable)
- Revaluation of foreign currency loan balances at month-end/quarter-end
- Translation gains/losses posted to a designated GL account
- Hedging entries (if any) tracked separately

**Revaluation Entry (example: USD loan):**

| Leg | Account | Debit | Credit |
|---|---|---|---|
| 1 | Principal Outstanding -- FCY (translated) | Revaluation Gain/Loss | |
| 2 | Foreign Currency Translation Reserve / P&L | | Revaluation Gain/Loss |

### 4.6 Profit Center and Cost Center Allocation

**Requirement ID: LMS-GL-014**
**Title:** Profit Center and Cost Center Tagging for Loan GL Entries
**Priority:** Should Have

**Business Rules:**
- Every GL entry must carry Profit Center and Cost Center tags
- Profit Centers typically mapped to: Branch, Region, Zone, Product Line, Segment (Retail/MSME/Corporate)
- Cost Center allocation rules for shared costs (e.g., centralized processing center costs allocated to originating branch)
- System must support Fund Transfer Pricing (FTP) tagging for each loan account to enable profitability analysis
- FTP rate assigned at origination based on matched-maturity funding cost; can be updated periodically

**Acceptance Criteria:**
1. All GL entries carry mandatory Profit Center and Cost Center codes
2. System prevents posting if PC/CC codes are missing
3. FTP rate is recorded at loan level and used for profitability reporting
4. Management reporting available at PC/CC level for interest income, fee income, provisioning cost, and net contribution

---
