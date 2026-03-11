# MODULE 7: INTEREST VS PRINCIPAL ALLOCATION
# MODULE 8: LOAN ORIGINATION LIFECYCLE (ADDITIONAL MODULE)
# MODULE 9: COLLATERAL/SECURITY MANAGEMENT (ADDITIONAL MODULE)

---

## 10. INTEREST VS PRINCIPAL ALLOCATION

### 10.1 Overview

This module manages the precise separation of interest and principal components in every loan transaction, which is essential for tax compliance (Section 80C, Section 24, Section 80E), regulatory reporting, customer communication, and financial accounting.

### 10.2 Amortization Schedule with Interest-Principal Split

**Requirement ID: LMS-IPA-001**
**Title:** Detailed Interest-Principal Breakdown in Amortization
**Priority:** Must Have

**Description:** Every installment in the amortization schedule must show the exact interest and principal components, and these components must be dynamically recalculated when loan terms change.

**Reducing Balance Impact Over Tenure:**

For a typical home loan (Rs 50L, 8.5%, 20 years, EMI Rs 43,391):

| Year | Opening Balance | Annual Interest | Annual Principal | Interest:Principal Ratio | Closing Balance |
|---|---|---|---|---|---|
| 1 | 50,00,000 | 4,22,129 | 98,563 | 81:19 | 49,01,437 |
| 5 | 46,02,153 | 3,84,498 | 1,36,194 | 74:26 | 44,65,959 |
| 10 | 38,60,789 | 3,14,987 | 2,05,705 | 60:40 | 36,55,084 |
| 15 | 27,15,635 | 2,09,043 | 3,11,649 | 40:60 | 24,03,986 |
| 20 | 8,69,332 | 40,262 | 4,80,430 | 8:92 | 0 |

**Business Rules:**
- Interest-principal split computed at individual installment level using the reducing balance method
- For daily reducing balance products: monthly interest = sum of daily interest; principal = EMI - monthly interest
- For flat rate products: interest-principal split follows the agreed method (equal interest per installment or Rule of 78)
- Split data stored at transaction level for each payment made (actual allocation, not scheduled)
- Difference between scheduled split and actual split (due to rate changes, prepayments) documented

### 10.3 Tax Certificate Generation

**Requirement ID: LMS-IPA-002**
**Title:** Comprehensive Tax Certificate Generation System
**Priority:** Must Have

**Section 24(b) -- Interest on Housing Loan:**

| Category | Deduction Limit | Conditions |
|---|---|---|
| Self-Occupied Property | Rs 2,00,000 p.a. (Rs 30,000 for pre-April 1999 loans) | Construction/purchase completed within 5 years from FY of borrowing |
| Let-Out Property | No limit (actual interest) | Property is rented out |
| Pre-Construction Interest | Deductible in 5 equal installments from year of possession | Total pre-construction interest computed separately |
| Under New Tax Regime (Section 115BAC) | Rs 2,00,000 for let-out property only; Nil for self-occupied | As per Finance Act 2023 amendments |

**Section 80C -- Principal Repayment:**

| Component | Deduction Limit | Conditions |
|---|---|---|
| Housing Loan Principal Repayment | Up to Rs 1,50,000 (within overall 80C limit) | For residential house property only |
| Stamp Duty and Registration Charges | Within 80C limit | Year of payment only |

Note: Under new tax regime (Section 115BAC), Section 80C deduction is NOT available.

**Section 80E -- Education Loan Interest:**

| Component | Limit | Period |
|---|---|---|
| Interest on Education Loan | No upper limit | 8 consecutive years from start of repayment, or until interest fully paid, whichever is earlier |

**Certificate Generation Rules:**
- Annual certificates generated within 15 days of FY end (by April 15 target; mandatory by June 15)
- FY period: April 1 to March 31
- Certificate shows: (a) total interest paid during FY, (b) total principal repaid during FY, (c) outstanding balance at FY end
- For co-borrowers: system generates proportional certificates (based on configured split ratio)
- Provisional certificate: generated for current FY based on remaining scheduled EMIs (projected values)
- Certificate includes: PAN of borrower, PAN of lender, TAN of lender, loan account number, property address (for home loans)
- Certificate data must reconcile with actual payment transactions, not accrual entries
- For NPA accounts: certificate reflects only actual amounts paid (cash basis), not accrued amounts

**Acceptance Criteria:**
1. Annual certificates generated automatically for all eligible loan accounts within 15 days of FY end
2. Certificate data matches actual transaction records
3. Co-borrower proportional certificates generated correctly
4. Provisional certificates available on-demand through customer portal
5. Certificate download available in PDF format with digital signature

### 10.4 Statement of Account (SOA) Generation

**Requirement ID: LMS-IPA-003**
**Title:** Detailed Statement of Account with Interest-Principal Split
**Priority:** Must Have

**SOA Contents:**

| Section | Details |
|---|---|
| Header | Borrower name, loan account number, product type, branch, loan amount, disbursement date, interest rate, tenure |
| Transaction Details | Date, description, debit, credit, principal component, interest component, charges, running balance |
| Summary | Total principal repaid, total interest paid, total charges, current outstanding, EMIs remaining |
| Rate Change History | Date, old rate, new rate, new EMI (if applicable) |
| Overdue Summary | Overdue installments, DPD, penal charges |
| Future Schedule | Next 12 months' projected installments with interest-principal split |

**Business Rules:**
- SOA available for any date range (default: FY or calendar year)
- SOA available on customer portal for self-service download
- SOA can be generated in real-time (for periods up to 12 months) or batched (for longer periods)
- SOA must include all transactions: EMIs, part-prepayments, penal charges, fee charges, insurance debits, subvention credits, interest reversals
- SOA format: PDF (primary), Excel (on request)
- SOA includes disclaimer about interest certificate for tax purposes (SOA is not a substitute for tax certificate)
- Multi-language support: English + Hindi + regional language (configurable)

### 10.5 Regulatory Reporting of Interest Income vs Principal Recovery

**Requirement ID: LMS-IPA-004**
**Title:** Segregated Reporting for Regulatory Returns
**Priority:** Must Have

**Business Rules:**
- BSR (Basic Statistical Return) reporting: interest income reported separately from principal recovery
- P&L reporting: interest income on loans is a primary line item
- NPA recovery reporting: bifurcated into interest recovery and principal recovery
- CRILC reporting: outstanding reported as principal outstanding; interest is separate
- Profitability analysis: interest spread (yield on advances - cost of funds) computed using interest income data
- IND AS 109: Effective Interest Rate (EIR) income may differ from contractual interest; both must be tracked

---

## 11. LOAN ORIGINATION LIFECYCLE (ADDITIONAL CRITICAL MODULE)

### 11.1 Overview

While the primary focus of this BRD is on loan management (post-disbursement), the LMS must integrate with or include a Loan Origination System (LOS) that covers the application-to-disbursement lifecycle. Key origination functions are specified here to ensure seamless data handoff from origination to management.

### 11.2 Origination Process Flow

**Requirement ID: LMS-ORG-001**
**Title:** End-to-End Loan Origination Workflow
**Priority:** Must Have

**Stages:**

| Stage | Key Activities | System Outputs |
|---|---|---|
| 1. Lead Capture | Customer inquiry; digital/branch/DSA lead; pre-qualification | Lead record with basic eligibility |
| 2. Application | Full application form; document upload; fee collection | Application ID; document checklist |
| 3. KYC/CKYC | Aadhaar eKYC/VKYC; PAN verification; CKYC search/download | KYC verification status; CKYC ID |
| 4. Credit Assessment | Bureau check; income analysis; FOIR/LTV computation; bank statement analysis | Credit score; bureau report; assessment summary |
| 5. Property/Collateral Evaluation | Valuation; legal opinion; technical report (for secured loans) | Valuation report; legal clearance; title search report |
| 6. Underwriting | Risk assessment; policy rule engine; deviation management | Underwriting decision; conditions/deviations |
| 7. Sanction | Approval per delegation; sanction letter generation | Sanction letter; terms and conditions |
| 8. Documentation | Loan agreement; mortgage deed; guarantee deed; CERSAI filing; e-stamping | Executed documents; CERSAI registration number |
| 9. Disbursement | Pre-disbursement checks; fund release; NACH mandate setup | Disbursement advice; GL entries; amortization schedule |
| 10. Post-Disbursement | Welcome letter; KFS issuance; repayment instructions; insurance activation | Customer communications; loan active in LMS |

### 11.3 Key Origination Requirements

**Requirement ID: LMS-ORG-002**
**Title:** Credit Bureau Integration for Origination
**Priority:** Must Have

**Bureau Checks Required:**
- CIBIL (TransUnion): CIBIL Score, CIBIL Report, Commercial Credit Report (for business loans)
- Experian: Experian Score, Credit Report
- CRIF High Mark: Score, Report (especially for microfinance -- MFI bureau data)
- Equifax: Score, Report

**Business Rules:**
- Minimum one bureau check mandatory before credit decision
- Multi-bureau check recommended (at least 2 bureaus for loans > Rs 10 Lakh)
- Bureau score thresholds configurable per product (e.g., CIBIL > 700 for home loans, > 650 for personal loans)
- Bureau report cached for configurable period (e.g., 30 days); fresh pull if older
- Consent: explicit borrower consent required before bureau pull (digital consent with timestamp)
- Bureau inquiry recorded; excessive inquiries tracked (multiple applications across lenders)

**Requirement ID: LMS-ORG-003**
**Title:** Account Aggregator (AA) Integration
**Priority:** Should Have

**Business Rules:**
- Integration with RBI-licensed Account Aggregators (Finvu, OneMoney, CAMS FinServ, NADL, etc.)
- Consent flow: borrower creates consent artifact on AA; LMS (as FIU -- Financial Information User) fetches data
- Data types: bank statements (deposit accounts), mutual fund holdings, insurance policies, tax data (26AS/AIS)
- Bank statement analysis: automated income computation, average balance, EMI bounce history, cash flow patterns
- AA data supplements traditional document-based verification
- Data retention: AA data retained only for the consented purpose and duration
- Fallback: if AA consent not available, accept traditional bank statements

**Requirement ID: LMS-ORG-004**
**Title:** Digital Lending Compliance in Origination
**Priority:** Must Have

**RBI Digital Lending Guidelines (September 2, 2022) Requirements:**

| Requirement | Implementation |
|---|---|
| Key Fact Statement (KFS) | System-generated KFS before sanction; borrower acknowledgment recorded |
| APR Disclosure | APR computed including all fees and charges; displayed in KFS |
| Cooling-Off Period | Configurable period (typically 3 days for DL); borrower can exit without penalty |
| Direct Disbursement | Loan disbursed directly to borrower's bank account (not to LSP/third party) |
| LSP Disclosure | Name and details of LSP disclosed to borrower at every touchpoint |
| Complaint Redressal | Nodal officer details displayed; grievance portal link provided |
| Data Access | Only need-based data collected; explicit consent for data sharing |
| FLDG Cap | First Loss Default Guarantee from LSP capped at 5% of portfolio |

**KFS Mandatory Fields (per RBI):**

| Field | Description |
|---|---|
| APR (Annual Percentage Rate) | All-inclusive cost of loan expressed as annualized rate |
| Loan Amount | Sanctioned and disbursed amount |
| Total Interest Payable | Over full tenure at current rate |
| Total Fees and Charges | Processing fee, documentation charges, insurance, etc. |
| EMI Amount | Monthly installment |
| Number of Installments | Total installments |
| Prepayment Terms | Penalty (if any), lock-in period |
| Recovery Mechanism | NACH/ECS/UPI AutoPay details |
| Complaint Redressal | RE's nodal officer details; RBI's IGMS portal reference |
| Cooling-Off Period | Duration and exit terms |

**Requirement ID: LMS-ORG-005**
**Title:** Sanction Letter and Loan Agreement Generation
**Priority:** Must Have

**Sanction Letter Contents:**
- Borrower details; co-borrower/guarantor details
- Loan amount sanctioned; tenure; interest rate (fixed/floating, benchmark + spread)
- EMI amount; mode of repayment
- Security/collateral details
- Processing fee and other charges
- Insurance requirements
- Special conditions/covenants (DSCR maintenance, end-use compliance, etc.)
- Validity of sanction (typically 3-6 months)
- Disbursement conditions (documents required, pre-disbursement checklist)

**Business Rules:**
- Sanction letter template configurable per product
- Digital signature on sanction letter (authorized signatory)
- Sanction amount may differ from applied amount (based on credit assessment)
- Sanction conditions tracked as checklist; disbursement blocked until all conditions met
- MITC (Most Important Terms and Conditions) as per RBI Fair Practices Code annexed to sanction letter

---

## 12. COLLATERAL/SECURITY MANAGEMENT (ADDITIONAL CRITICAL MODULE)

### 12.1 Overview

Comprehensive collateral management is critical for secured lending, which constitutes the majority of Indian bank lending. This module covers collateral registration, valuation, monitoring, CERSAI filing, lien management, and release.

### 12.2 Collateral Types and Management

**Requirement ID: LMS-COL-001**
**Title:** Multi-Type Collateral Management System
**Priority:** Must Have

**Collateral Types Supported:**

| Type | Sub-Types | Key Attributes | Valuation Method |
|---|---|---|---|
| Immovable Property | Residential, Commercial, Industrial, Agricultural Land, Plot | Address, survey number, area, title chain, encumbrance status, occupation status | Registered valuer; distress value at 75-85% of market |
| Vehicle | Car, Two-Wheeler, Commercial Vehicle, Construction Equipment | Registration number, chassis number, engine number, make/model/year, RC book | IDV (Insured Declared Value); depreciated value |
| Gold | Jewellery, Coins, Bars | Weight (gross and net), purity (karat), item description | LTV on gold value; RBI max LTV 75% for gold loans |
| Fixed Deposit | FD with same bank, FD with other bank | FD number, bank name, amount, maturity date, lien status | Face value; lien marked |
| Securities | Shares (listed/unlisted), Mutual Funds, Bonds, Debentures | ISIN, quantity, demat account, current market value | Mark-to-market; haircut per RBI norms |
| Plant & Machinery | Fixed assets, movable machinery | Description, serial number, location, condition | Valuation by approved valuer; depreciated replacement cost |
| Stock/Inventory | Raw material, WIP, Finished goods | Type, quantity, location, shelf life | Drawing power calculation; margin on stock value |
| Book Debts | Trade receivables | Debtor list, aging, disputed amounts | Drawing power; margin on book debts < 90 days |
| Personal/Corporate Guarantee | Individual guarantor, Corporate guarantee | Guarantor details, net worth, financial statements | Not directly valued; assessed for recovery potential |
| Assignment of Receivables | Insurance policy, contractual receivables | Policy/contract details, amount, duration | Present value of future receivables |

### 12.3 Collateral Valuation

**Requirement ID: LMS-COL-002**
**Title:** Collateral Valuation and Revaluation Management
**Priority:** Must Have

**Valuation Rules:**

| Collateral Type | Initial Valuation | Revaluation Frequency | Revaluation Trigger |
|---|---|---|---|
| Residential Property | At origination by approved valuer | Every 3 years (standard); annually for NPA | NPA classification; restructuring; significant market change |
| Commercial Property | At origination by approved valuer | Annually | Same as above |
| Gold | At origination (assayer) | Daily mark-to-market (gold price) | Automatic |
| Listed Securities | At origination (market price) | Daily mark-to-market | Automatic |
| Fixed Deposit | At origination (face value) | At maturity/renewal | Maturity date |
| Stock/Inventory | Monthly/Quarterly stock audit | Monthly/Quarterly | Drawing power review |
| Vehicle | At origination (IDV) | Annually | Depreciation schedule |

**LTV (Loan-to-Value) Norms:**

| Product | RBI/NHB Mandated Max LTV | Reference |
|---|---|---|
| Home Loan (up to Rs 30 Lakh) | 90% | NHB Circular |
| Home Loan (Rs 30-75 Lakh) | 80% | NHB Circular |
| Home Loan (above Rs 75 Lakh) | 75% | NHB Circular |
| Gold Loan | 75% | RBI Master Direction on gold loans |
| Loan Against FD | 90-95% (bank's own FD); 75-85% (other bank FD) | Internal policy |
| Loan Against Shares (listed) | 50% of market value | RBI/SEBI norms |
| LAP (Residential) | 65-75% | Internal policy (RBI advisory) |
| LAP (Commercial) | 60-65% | Internal policy |

**Business Rules:**
- LTV checked at origination and at every revaluation
- If LTV exceeds threshold due to property depreciation: trigger additional collateral request or margin call
- For gold loans: continuous LTV monitoring; if gold price drops and LTV breaches 75%, system triggers margin call or part-closure notice
- Empaneled valuers: valuation accepted only from institution's empaneled valuers; empanelment managed in master
- Valuation report format: standardized template with photographs, GPS coordinates, comparable analysis
- Valuation fee: borne by borrower; tracked in loan account
- Two-valuer opinion: required for property loans above configurable threshold (e.g., Rs 50 Lakh+)

### 12.4 CERSAI Integration

**Requirement ID: LMS-COL-003**
**Title:** CERSAI Registration and Management
**Priority:** Must Have

**CERSAI (Central Registry of Securitisation Asset Reconstruction and Security Interest of India):**

**Registration Requirements:**
- Mandatory for all equitable mortgages, hypothecation, and other security interests
- Filing within 30 days of creation of security interest
- Modification filing within 30 days of any change
- Satisfaction filing within 30 days of loan closure/security release

**System Requirements:**

| Function | Description |
|---|---|
| CERSAI Search | Before creating security interest, search CERSAI for existing charges on the asset |
| CERSAI Registration | File Form I (creation of security interest) via CERSAI portal/API |
| CERSAI Modification | File modification if security interest terms change |
| CERSAI Satisfaction | File Form IV on loan closure / security release |
| CERSAI Reporting | Track CERSAI filing status; aging of pending filings |

**Business Rules:**
- CERSAI search mandatory before sanction of any secured loan (to verify no prior charges)
- CERSAI registration initiated automatically on disbursement
- CERSAI fees: recovered from borrower (currently Rs 500-1000 depending on loan amount)
- Non-filing penalty: applicable per CERSAI Act; system must alert for pending filings
- CERSAI Asset ID: stored in collateral master and linked to loan account
- Integration: API-based integration with CERSAI portal preferred; manual filing as fallback
- Monthly CERSAI filing compliance report for management and audit

### 12.5 Lien Marking and Management

**Requirement ID: LMS-COL-004**
**Title:** Lien Marking for Financial Collateral
**Priority:** Must Have

**Lien Types:**

| Collateral | Lien Mechanism | System Requirement |
|---|---|---|
| Fixed Deposit (own bank) | System lien in CBS | API to CBS for lien mark/unmark; auto-renewal tracking |
| Fixed Deposit (other bank) | Letter of lien to other bank | Track lien confirmation; renewal; lien release letter |
| Shares/Demat | Pledge via CDSL/NSDL | Integration with depository; pledge creation/invocation/release |
| Mutual Funds | Lien via AMC/RTA | Integration with CAMs/KFintech for lien marking |
| Insurance Policy | Assignment to lender | Track assignment letter; policy renewal; nominee amendment |
| Vehicle | Hypothecation in RC book | Form 34 for hypothecation; Form 35 for removal; RTO integration |

**Business Rules:**
- Lien status tracked in system: Pending, Marked, Confirmed, Released
- Lien renewal: for FDs nearing maturity, auto-alert for lien renewal
- Lien adequacy: if FD value (principal + interest) falls below threshold vs loan outstanding, trigger top-up
- Share pledge: margin monitoring; if share price falls below trigger level, margin call issued
- Insurance assignment: if policy lapses, alert for renewal; affects credit risk assessment

---
