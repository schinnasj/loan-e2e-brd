# MODULE 10: INSURANCE INTEGRATION
# MODULE 11: REGULATORY REPORTING
# MODULE 12: CREDIT BUREAU INTEGRATION
# MODULE 13: DIGITAL LENDING COMPLIANCE
# MODULE 14: CUSTOMER COMMUNICATION

---

## 13. INSURANCE INTEGRATION

### 13.1 Overview

**Requirement ID: LMS-INS-001**
**Title:** Comprehensive Insurance Lifecycle Management
**Priority:** Must Have

**Insurance Types for Lending:**

| Insurance Type | Purpose | Typical Products | Premium Structure |
|---|---|---|---|
| Credit Life Insurance | Covers outstanding loan on death/disability of borrower | Home Loan, Personal Loan, Vehicle Loan | Single premium (upfront) or annual renewable |
| Property Insurance | Covers property damage (fire, flood, earthquake) | Home Loan, LAP | Annual premium |
| Mortgage Guarantee | Covers lender's loss on default (IMGC/NHB) | Home Loan (high LTV) | Single premium, 1-2% of loan amount |
| Vehicle Insurance | Comprehensive cover for financed vehicle | Vehicle Loan | Annual premium |
| Crop Insurance | PMFBY (Pradhan Mantri Fasal Bima Yojana) | Agricultural/KCC Loans | Seasonal premium (subsidized) |
| Key Man Insurance | Covers loss of key person in business | Business Loan, MSME | Annual premium |
| Stock/Inventory Insurance | Covers hypothecated stock | Working Capital, CC/OD | Annual premium |
| Personal Accident Insurance | Covers accidental death/disability | Personal Loan, Microfinance | Annual premium |

### 13.2 Insurance Lifecycle

**Requirement ID: LMS-INS-002**
**Title:** Insurance Policy Lifecycle Tracking
**Priority:** Must Have

**Lifecycle Stages:**

| Stage | System Action |
|---|---|
| Policy Proposal | Generate proposal with borrower/loan details; submit to insurer |
| Premium Collection | Deduct from disbursement (single premium) or include in EMI (annual) |
| Policy Issuance | Record policy number, coverage amount, tenure, nominee details |
| Premium Remittance | Remit collected premium to insurance company |
| Renewal Tracking | Alert 30 days before policy expiry; auto-debit renewal premium |
| Claim Initiation | On death/disability/property damage: initiate claim with insurer |
| Claim Settlement | Track claim status; receive settlement; apply to loan outstanding |
| Policy Cancellation/Refund | On loan closure: cancel policy; process premium refund |

**Business Rules:**
- Sum assured = loan outstanding at disbursement (or higher, per policy type)
- Reducing sum assured: for credit life, sum assured can decrease with loan outstanding
- Insurance is not mandatory for loan (per RBI guidelines); borrower can opt out but must acknowledge risk
- If borrower opts out of insurance: separate acknowledgment recorded; risk accepted by institution
- Insurance master: empaneled insurers, products, premium rates, commission structures
- Claim settlement applied to loan: first to overdue interest, then to overdue principal, then to current outstanding (per waterfall)
- If claim settlement exceeds outstanding: excess refunded to nominee/legal heir
- IRDAI guidelines on group insurance for lending: premium transparency, no bundling without consent

**Acceptance Criteria:**
1. Insurance policies linked to loan accounts with complete policy details
2. Premium collection and remittance reconciled monthly
3. Renewal alerts generated 30 days before expiry
4. Claim workflow tracks all stages with SLA
5. Premium refund processed within 15 days of loan closure

---

## 14. REGULATORY REPORTING

### 14.1 Overview

**Requirement ID: LMS-REG-001**
**Title:** Automated Regulatory Return Generation
**Priority:** Must Have

**Description:** The system must support generation of all mandatory regulatory returns. Data extraction must be automated from loan sub-ledger data with minimal manual intervention.

### 14.2 RBI Returns

**Key Regulatory Returns:**

| Return | Full Name | Frequency | Key Data Points from LMS | Applicable To |
|---|---|---|---|---|
| BSR-1 | Basic Statistical Return - Part 1 | Annual | Loan-wise data: outstanding, interest rate, sector, purpose, security | Banks |
| BSR-2 | Basic Statistical Return - Part 2 | Quarterly | Aggregate advances data by category | Banks |
| DSB | Data on Sectoral Deployment of Bank Credit | Monthly | Sector-wise advances: agriculture, industry, services, personal | Banks |
| CRILC | Central Repository of Information on Large Credits | Weekly (SMA); Quarterly (main) | Borrower-wise data for exposures >= Rs 5 Cr; SMA classification; asset quality | Banks, NBFC-UL |
| XBRL | eXtensible Business Reporting Language | Varies | Financial statements, prudential data in XBRL format | Banks, NBFCs |
| PSL Returns | Priority Sector Lending Returns | Quarterly | PSL classification, sub-categories, targets, shortfalls | Banks |
| SMA Reporting | Special Mention Account Reporting | Daily (internal); Weekly (CRILC) | SMA-0/1/2 classification, DPD, outstanding | Banks |
| ALM Returns | Asset-Liability Management Returns | Monthly/Quarterly | Maturity profile of advances; residual maturity buckets | Banks, NBFCs |
| LBS/DBIE | Locational Banking Statistics / Database on Indian Economy | Quarterly | Geographic distribution of credit | Banks |
| CPC Reports | Credit to Priority Sector Returns | Quarterly | Priority sector advances detailed breakup | Banks |
| NPA Returns | NPA Movement and Provisioning | Quarterly | Gross NPA, Net NPA, provisions, write-offs, upgrades, recoveries | Banks, NBFCs |
| R-Returns | Various NHB Returns | Quarterly/Annual | Housing loan portfolio data, NPA, LTV compliance | HFCs |

### 14.3 CRILC Reporting (Detailed)

**Requirement ID: LMS-REG-002**
**Title:** CRILC Data Submission Engine
**Priority:** Must Have

**CRILC Reporting Requirements:**

| Report Type | Frequency | Threshold | Data Points |
|---|---|---|---|
| Main Return | Quarterly | Aggregate exposure >= Rs 5 Cr | Borrower details, all facilities, outstanding, classification, SMA status, restructuring status, wilful defaulter status |
| SMA-1/SMA-2 Return | Weekly | Same threshold + SMA status | Accounts classified as SMA-1 or SMA-2 |
| Fraud Reporting | As occurs | All fraud cases | Fraud type, amount, modus operandi, FIR details |

**Business Rules:**
- CRILC data extracted automatically from loan sub-ledger
- Group exposure: system must aggregate exposure at borrower group level (using common PAN, CIN, or defined group linkages)
- Data quality: CRILC submission validated against CRILC schema before submission
- Reconciliation: CRILC reported data must match GL outstanding
- Historical CRILC data maintained for comparison and audit

### 14.4 Priority Sector Lending (PSL) Tracking

**Requirement ID: LMS-REG-003**
**Title:** PSL Classification and Monitoring
**Priority:** Must Have

**PSL Targets (Banks):**

| Category | Target (% of ANBC) | Sub-Targets |
|---|---|---|
| Total PSL | 40% | -- |
| Agriculture | 18% | Of which 10% Small/Marginal Farmers |
| Micro Enterprises | 7.5% | -- |
| Weaker Sections | 12% | SC/ST, Women, Minorities, etc. |
| Education | No specific sub-target | Loans up to Rs 20 Lakh |
| Housing | No specific sub-target | Loans up to Rs 35 Lakh (metro) / Rs 25 Lakh (non-metro) |

**SFB PSL Target:** 75% of ANBC

**System Requirements:**
- PSL classification tag at loan level: category, sub-category, eligible/ineligible
- PSL eligibility engine: validates loan parameters against PSL criteria (amount thresholds, borrower type, purpose)
- PSL shortfall computation: quarterly assessment of target vs achievement
- PSLC (Priority Sector Lending Certificate) trading: track PSL certificates purchased/sold via RBI e-Kuber platform
- RIDF/NHB/SIDBI/Nabard contribution tracking (for PSL shortfall investment)
- Quarterly PSL MIS for management and Board reporting

---

## 15. CREDIT BUREAU INTEGRATION

### 15.1 Bureau Inquiry (Pull)

**Requirement ID: LMS-BUR-001**
**Title:** Multi-Bureau Credit Inquiry System
**Priority:** Must Have

**Supported Bureaus:**

| Bureau | API Integration | Data Retrieved |
|---|---|---|
| TransUnion CIBIL | CIBIL Connect / API 2.0 | CIBIL Score (300-900), Credit Report, DPD history, existing loans, inquiry history |
| Experian | Experian Connect API | Experian Score, Credit Report, existing credit lines |
| CRIF High Mark | CRIF API | Score, Report (strong for microfinance/MFI data) |
| Equifax | Equifax API | Score, Report |

**Business Rules:**
- Bureau inquiry requires borrower consent (physical or digital; timestamp recorded)
- Inquiry logged: bureau name, date, purpose, score received, report reference
- Bureau report cached: re-inquiry within configurable period (e.g., 30 days) uses cached report unless fresh pull requested
- Multi-bureau score: system can be configured to use best score, worst score, or average across bureaus
- Bureau score thresholds: configurable per product for auto-approve, manual review, and auto-decline bands
- Score trend: track historical scores for existing borrowers (at each inquiry/annual review)
- Bureau data feed into credit assessment engine for automated underwriting

### 15.2 Bureau Reporting (Push)

**Requirement ID: LMS-BUR-002**
**Title:** Monthly Credit Bureau Data Submission
**Priority:** Must Have

**Reporting Requirements:**
- Monthly submission to all 4 bureaus (CIBIL, Experian, CRIF, Equifax)
- Report format: CIBIL TUDF (TransUnion Data Format), Experian format, CRIF format, Equifax format
- Report all active, closed, written-off, settled, and restructured accounts

**Data Fields Reported:**

| Field | Description |
|---|---|
| Member ID | Institution's bureau member ID |
| Account Number | Loan account number |
| Borrower Details | Name, DOB, PAN, Aadhaar (masked), address, phone |
| Loan Details | Product type, sanction amount, disbursement date, tenure |
| Current Balance | Outstanding principal |
| EMI Amount | Current EMI |
| DPD | Days Past Due (current month) |
| Account Status | Standard, SMA, NPA Sub-Standard, Doubtful, Loss, Written-Off, Settled, Closed |
| Payment History | 36-month DPD history (month-by-month) |
| Ownership | Individual, Joint - First Holder, Joint - Second Holder, Guarantor |
| Date of Last Payment | Date of most recent payment received |
| Written-Off Amount | If written off: principal + interest written off |
| Settlement Amount | If settled (OTS): settlement amount |

**Business Rules:**
- Reporting accuracy: 99.9% accuracy target; errors cause customer CIBIL score impact
- Data validation: system validates all records against bureau schema before submission
- Rejection handling: bureau returns rejected records with error codes; system must correct and resubmit
- Customer dispute: if borrower disputes bureau data, system must support investigation and correction workflow
- Closed account reporting: accounts reported as "Closed" with zero balance for minimum 36 months post-closure
- Written-off accounts: continue reporting until recovery or actual write-off
- Settled accounts: reported as "Settled" (negative impact on score vs "Closed"); borrower may request conversion to "Closed" after paying full dues

**Acceptance Criteria:**
1. Monthly bureau file generated in correct format for all 4 bureaus
2. Data validation catches > 99% of format/data errors before submission
3. Rejected records investigated and resubmitted within 30 days
4. Customer dispute workflow available with SLA tracking

---

## 16. DIGITAL LENDING COMPLIANCE

### 16.1 Compliance Framework

**Requirement ID: LMS-DLC-001**
**Title:** RBI Digital Lending Guidelines Compliance Module
**Priority:** Must Have

**Reference:** RBI/2022-23/111 DOR.CRE.REC.66/21.07.001/2022-23 dated September 2, 2022; subsequent FAQs and clarifications.

**System Requirements:**

| Requirement | System Implementation |
|---|---|
| KFS Generation | Auto-generated Key Fact Statement with APR, all charges, terms; borrower e-acknowledgment captured |
| Cooling-Off Period | Configurable period (default 3 days) during which borrower can exit; system blocks full enforcement; pro-rata interest only |
| Direct Disbursement | System enforces disbursement only to borrower's verified bank account (not to LSP/third-party) |
| No Automatic Credit | No auto-enhancement of credit limit without explicit borrower consent |
| Complaint Redressal | Grievance officer details on all customer communications; escalation to RBI IGMS if unresolved in 30 days |
| Data Privacy | Explicit consent for data collection; purpose limitation; data stored only in India (RBI data localization) |
| LSP Management | LSP registration master; LSP-wise portfolio tracking; LSP code of conduct compliance |
| FLDG Tracking | Track FLDG arrangements; cap at 5% of total loan portfolio; mark-to-market of FLDG invoked vs available |

### 16.2 FLDG (First Loss Default Guarantee) Management

**Requirement ID: LMS-DLC-002**
**Title:** FLDG Tracking and Compliance
**Priority:** Must Have

**RBI Reference:** DOR.FIN.REC.No.20/03.10.038/2023-24 dated June 8, 2023

**Business Rules:**
- FLDG = guarantee provided by LSP/DLF to RE (Regulated Entity) for bearing first losses up to a defined percentage
- Maximum FLDG: 5% of the total loan portfolio originated by the LSP/DLF for the RE
- FLDG can be in the form of: (a) cash deposit with RE, (b) bank guarantee, (c) fixed deposit lien
- System tracks: FLDG agreement details, current FLDG utilization, remaining FLDG headroom
- FLDG invocation: when loan defaults, lender invokes FLDG per agreement terms
- Monitoring: monthly FLDG utilization report; alert when utilization exceeds 80% of cap
- FLDG replacement: if LSP fails to replenish invoked FLDG within agreed timeline, alert and escalate

**FLDG Accounting:**

| Event | Dr | Cr |
|---|---|---|
| FLDG received (cash deposit) | Bank Account | FLDG Liability (held for LSP) |
| FLDG invoked on default | FLDG Liability | Loan Recovery / Write-Off Recovery |
| FLDG returned to LSP (unused, on portfolio rundown) | FLDG Liability | Bank Account |

---

## 17. CUSTOMER COMMUNICATION

### 17.1 Communication Framework

**Requirement ID: LMS-COM-001**
**Title:** Multi-Channel Customer Communication System
**Priority:** Must Have

**Communication Channels:**

| Channel | Use Cases | Integration |
|---|---|---|
| SMS | Payment reminders, EMI due alerts, rate change notification, OTP | SMS gateway (Bulk SMS provider via API) |
| Email | Loan statements, tax certificates, demand notices, welcome letters | Email gateway (SMTP/SES) |
| Physical Letter | Demand notices (registered AD), SARFAESI notices, NOC | Postal service; bulk printing vendor integration |
| WhatsApp | Payment reminders, statement sharing, chatbot | WhatsApp Business API |
| IVR / Voice | Collection calls, payment reminders | Contact center integration |
| Push Notification | Mobile app alerts for due dates, rate changes | Firebase/APNS |
| Customer Portal | Self-service: statements, certificates, foreclosure quotes | Web application |
| Mobile App | Self-service: payments, document upload, tracking | Mobile application |

### 17.2 Mandatory Communications

**Requirement ID: LMS-COM-002**
**Title:** Regulatory and Contractual Mandatory Notices
**Priority:** Must Have

| Communication | Trigger | Timeline | Regulatory Reference |
|---|---|---|---|
| Welcome Letter / Sanction Letter | Post-disbursement | Within 1 business day | Fair Practices Code |
| Key Fact Statement (KFS) | Pre-sanction (DL) / At sanction | Before borrower acceptance | RBI DL Guidelines |
| EMI Due Reminder | Before due date | 3-5 days before EMI | Best practice |
| Rate Change Intimation | Benchmark rate change | At least 30 days before reset (or immediately for decrease) | RBI Fair Practices Code; EBLR circular |
| Annual Statement | Financial year end | Within 2 months of FY end | Good practice |
| Interest Certificate | Financial year end | By June 15 | Income Tax Act |
| Demand Notice (overdue) | Account overdue | Per collection schedule | Fair Practices Code |
| NPA Classification Notice | On NPA classification | Within 7 days | IRAC norms (good practice) |
| SARFAESI Section 13(2) Notice | Post NPA (secured loan) | After NPA classification | SARFAESI Act |
| Restructuring Terms Notice | On restructuring approval | Within 7 days | Restructuring framework |
| NOC / Closure Letter | On loan closure | Within 15 days | Fair Practices Code |
| Collateral Release Intimation | On document release | At time of release | Good practice |
| Bounced Payment Notification | NACH/cheque bounce | Within 24 hours | Good practice |

**Business Rules:**
- Template management: all communication templates version-controlled with maker-checker for changes
- Language: English + Hindi + one regional language (configurable per borrower preference/state)
- DND/NCPR compliance: marketing communications respect DND; transactional exempt
- Communication timing: no communications between 7 PM and 7 AM (RBI guideline for collection communication; extended to all)
- Communication log: every communication recorded with channel, timestamp, content hash, delivery status
- Undelivered communications: retry logic for SMS/email; physical letter return tracking
- Borrower preference: system records preferred communication channel per borrower

---
