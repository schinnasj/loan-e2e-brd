# MODULE 4: INTEREST COMPUTATION

## 7. INTEREST COMPUTATION

### 7.1 Overview

The Interest Computation module is the mathematical core of the LMS. Accuracy in interest computation directly impacts customer trust, regulatory compliance, financial reporting, and tax obligations. This module must support the diverse interest computation methods used across Indian financial products, handle floating rate complexities, comply with RBI's guidelines on interest rate transparency, and produce accurate TDS calculations.

### 7.2 Interest Calculation Methods

#### 7.2.1 Simple Interest

**Requirement ID: LMS-INT-001**
**Title:** Simple Interest Calculation
**Priority:** Must Have

**Formula:**
Simple Interest = P x R x T / (Day Count Basis)

Where:
- P = Principal outstanding
- R = Annual interest rate (as decimal)
- T = Number of days
- Day Count Basis = 365 or 360 (as configured)

**Use Cases in Indian Lending:**
- Overdraft/Cash Credit interest (calculated on daily outstanding balance)
- Education loan moratorium period interest
- Agricultural short-term crop loans
- Gold loans (some products)
- Interest calculation for NPA memorandum records

**Example:**
- Principal: Rs 5,00,000
- Rate: 10.50% p.a.
- Period: 45 days
- Day Count: Actual/365

Interest = 5,00,000 x 10.50% x 45 / 365 = Rs 6,472.60

#### 7.2.2 Compound Interest -- Monthly Reducing Balance

**Requirement ID: LMS-INT-002**
**Title:** Monthly Reducing Balance Interest (Standard EMI Method)
**Priority:** Must Have

**Description:** The standard method for most retail loan products (home loans, personal loans, vehicle loans, LAP) in India. Interest calculated on the outstanding principal balance at the beginning of each month.

**Calculation Logic:**
- Monthly interest = Outstanding Principal x (Annual Rate / 12)
- Principal component of EMI = EMI - Monthly Interest
- New outstanding = Previous Outstanding - Principal Component

**Example (Month 1 of a Home Loan):**
- Outstanding: Rs 50,00,000
- Rate: 8.50% p.a.
- EMI: Rs 43,391
- Month 1 Interest = 50,00,000 x 8.50% / 12 = Rs 35,417
- Month 1 Principal = 43,391 - 35,417 = Rs 7,974
- Closing Outstanding = 50,00,000 - 7,974 = Rs 49,92,026

#### 7.2.3 Compound Interest -- Daily Reducing Balance

**Requirement ID: LMS-INT-003**
**Title:** Daily Reducing Balance Interest Calculation
**Priority:** Must Have

**Description:** Interest calculated on the actual daily outstanding balance. More accurate and fair to borrowers, as any mid-month payment immediately reduces the interest-bearing balance. Increasingly common for home loans and personal loans in India.

**Calculation Logic:**
- Daily Interest = Outstanding Principal on that day x (Annual Rate / Day Count Basis)
- Monthly interest = Sum of daily interest for all days in the month
- Principal component = EMI - Monthly interest (adjusted for actual days)

**Example:**
- Outstanding as of April 1: Rs 50,00,000
- Rate: 8.50% p.a.
- Day Count: Actual/365
- Part-prepayment of Rs 2,00,000 on April 15

April 1-14 (14 days): Interest = 50,00,000 x 8.50% / 365 x 14 = Rs 16,301.37
April 15-30 (16 days): Interest = 48,00,000 x 8.50% / 365 x 16 = Rs 17,884.93
Total April Interest = Rs 34,186.30

Compare with monthly reducing (without mid-month prepayment adjustment):
Full month on Rs 50,00,000 = Rs 35,416.67

**Business Rules:**
- Daily reducing balance must track the exact outstanding balance for each calendar day
- Any payment (EMI, part-prepayment, insurance recovery) that reduces principal on day D: interest computed on reduced balance from day D+1
- For backdated payments (e.g., cheque value-dated earlier): system must support interest recalculation from the value date
- System must handle leap year (366 days) correctly when using Actual/365 or Actual/Actual convention

#### 7.2.4 Yearly Reducing Balance

**Requirement ID: LMS-INT-004**
**Title:** Yearly Reducing Balance Interest Calculation
**Priority:** Should Have

**Description:** Interest calculated on the outstanding balance at the beginning of each year. Principal reduction recognized only at year-end. Less common but used in some MSME and agricultural term loans.

**Business Rules:**
- Annual interest = Outstanding at start of year x Annual Rate
- Monthly EMI splits: more interest-heavy throughout the year
- Principal reduction applied at year-end
- Less favorable to borrower compared to monthly/daily reducing

### 7.3 Day Count Conventions

**Requirement ID: LMS-INT-005**
**Title:** Configurable Day Count Conventions
**Priority:** Must Have

**Description:** The system must support multiple day count conventions and allow configuration at the product level.

| Convention | Days in Period | Days in Year | Common Usage in India |
|---|---|---|---|
| Actual/365 | Actual calendar days | 365 (always, even in leap year) | Most common for Indian retail loans |
| Actual/Actual | Actual calendar days | 365 or 366 (leap year) | Some NBFC products; IND AS compliant EIR calculation |
| Actual/360 | Actual calendar days | 360 | ECB/foreign currency loans; some money market instruments |
| 30/360 | Each month = 30 days | 360 | Some legacy products; European-style calculations |
| Actual/365 (Fixed) | Actual calendar days | 365 (fixed, ignores leap year) | Variant used by some Indian banks |

**Business Rules:**
- Day count convention configurable at product level; cannot be changed for existing loans without restructuring
- For IND AS 109 EIR calculation: Actual/Actual convention recommended for accuracy
- System must handle month-end dates correctly (e.g., February 28/29, months with 30/31 days)
- Leap year handling: explicitly documented and tested for each convention

**Acceptance Criteria:**
1. Interest calculated using each convention matches reference financial calculator
2. Leap year calculations validated for all conventions
3. Day count convention displayed in loan agreement and amortization schedule

### 7.4 Interest Accrual

**Requirement ID: LMS-INT-006**
**Title:** Daily Interest Accrual Engine
**Priority:** Must Have

**Description:** The system shall accrue interest daily for all active loan accounts as part of EOD processing. Accrual is the recognition of earned interest in the books, regardless of whether it has been collected.

**Accrual Process Flow:**

1. EOD batch picks all active loan accounts (status: Disbursed, Performing, SMA, NPA)
2. For each account:
   a. Determine today's outstanding principal balance
   b. Determine applicable interest rate (check if rate reset applies today)
   c. Apply day count convention
   d. Compute daily interest amount
   e. For performing accounts: post accrual entry (Dr Interest Receivable, Cr Interest Income)
   f. For NPA accounts: post to memorandum account (Dr NPA Interest Memorandum, Cr NPA Interest Memorandum -- off-balance sheet only)
3. Generate accrual summary report

**Business Rules:**
- Accrual runs for every calendar day, including holidays and weekends
- Holiday processing: if EOD runs on Monday for Saturday and Sunday, three days of accrual are processed
- If EOD was missed (system outage), catch-up accrual for missed days must be processed on the next EOD run
- Interest accrual for NPA accounts: per RBI IRAC norms, interest income on NPA accounts is not recognized in P&L; it is tracked in a memorandum account and recognized only upon actual recovery (cash basis)
- Reversal of accrued interest on NPA classification: see LMS-GL-007
- Accrual amount stored at individual loan level for: (a) interest certificate generation, (b) provisioning calculation, (c) tax reporting
- Month-end accrual: separate flag to mark the last accrual of the month for month-end financial closing

**Acceptance Criteria:**
1. Daily accrual processed for 100% of active loan accounts
2. Accrual amount matches manual calculation (random sampling during UAT)
3. NPA accounts accrued in memorandum only, not in P&L
4. Multi-day catch-up accrual processed correctly after system outage
5. Month-end accrual clearly identified for financial closing

### 7.5 Floating Rate Interest Management

#### 7.5.1 Benchmark Rate Types

**Requirement ID: LMS-INT-007**
**Title:** Floating Rate Benchmark Management
**Priority:** Must Have

**Description:** Indian floating rate loans are linked to various benchmark rates. The system must maintain a benchmark rate master and manage rate resets.

**Benchmark Rate Types in India:**

| Benchmark | Full Name | Set By | Typical Products | Reset Frequency |
|---|---|---|---|---|
| Repo Rate / EBLR | External Benchmark Lending Rate | RBI (Repo Rate); Bank sets spread | Home Loans, Personal Loans, MSME Loans (banks -- mandatory since Oct 2019) | Quarterly (minimum as per RBI) |
| MCLR | Marginal Cost of Funds Based Lending Rate | Individual Bank | Term Loans (legacy; banks) | Monthly/Quarterly/Half-yearly/Yearly |
| RLLR | Repo Linked Lending Rate | Individual Bank (Repo + spread) | Home Loans, Auto Loans | When Repo Rate changes |
| T-Bill Linked | Treasury Bill Linked Rate | Market (3-month/6-month T-bill) | SFBs often use T-bill linked rates | Quarterly |
| BPLR | Benchmark Prime Lending Rate | Individual Bank (legacy) | Legacy loans (pre-2016) | As decided by bank |
| PLR | Prime Lending Rate | NBFCs | Most NBFC loan products | As decided by NBFC Board |
| Individual NBFC Benchmark | NBFC-specific benchmark | NBFC | NBFC products | As decided by NBFC |

**RBI Mandate on EBLR (Reference: RBI circular on External Benchmarking of Loans, September 4, 2019):**
- All new floating rate loans in following categories must be linked to an external benchmark: (a) personal loans, (b) home loans, (c) loans to MSEs, (d) vehicle loans (effective October 1, 2019)
- External benchmarks allowed: RBI Repo Rate, GoI 3-month T-bill yield, GoI 6-month T-bill yield, any other benchmark published by Financial Benchmarks India Pvt Ltd (FBIL)
- Spread over external benchmark: fixed at origination for the loan tenure (barring credit risk reassessment)
- Reset frequency: minimum quarterly for EBLR-linked loans
- Banks can change spread for new borrowers but not for existing borrowers (except for credit event-based reassessment)

#### 7.5.2 Rate Reset Mechanism

**Requirement ID: LMS-INT-008**
**Title:** Floating Rate Reset Engine
**Priority:** Must Have

**Description:** The system must process benchmark rate changes and cascade them to individual loan accounts based on the reset date and frequency.

**Rate Reset Process:**

1. **Benchmark Rate Update**: Authorized user updates benchmark rate in the rate master (e.g., RBI changes Repo Rate from 6.50% to 6.25%)
   - Maker-checker approval required
   - Effective date recorded (e.g., effective April 5, 2026)

2. **Identify Impacted Accounts**: System identifies all loan accounts linked to the changed benchmark with reset date on or after the effective date
   - Filter by benchmark type, reset frequency, next reset date

3. **Compute New Rate**: New Loan Rate = New Benchmark Rate + Fixed Spread + Risk Premium (if any)
   - Example: New Repo = 6.25%, Spread = 2.40%, Risk Premium = 0.00%
   - New Loan Rate = 6.25% + 2.40% = 8.65% (was 8.90% when Repo was 6.50%)

4. **Apply New Rate**: On the reset date for each account:
   - Update interest rate in loan master
   - Recalculate EMI (if EMI-change option) or remaining tenure (if tenure-change option)
   - Regenerate amortization schedule
   - Generate customer notification (mandatory per RBI guidelines)

5. **Post Rate-Change Entries**: If rate change results in EIR adjustment under IND AS 109, post fair value adjustment entries

**Rate Reset Frequency Options:**

| Frequency | Description | Common For |
|---|---|---|
| Immediate | Rate changes as soon as benchmark changes | Some NBFC products |
| Monthly | Rate reset on first of every month based on benchmark as of previous month-end | MCLR (monthly reset) |
| Quarterly | Rate reset every quarter (Jan, Apr, Jul, Oct) | EBLR (mandatory minimum) |
| Half-Yearly | Rate reset every 6 months from disbursement | MCLR (semi-annual reset) |
| Yearly | Rate reset once a year from disbursement date | MCLR (annual reset) |
| At Bank's Discretion | Rate changes when bank decides (legacy) | BPLR-linked (legacy) |

**Business Rules:**
- Rate reset date for each loan tracked individually (based on disbursement date and reset frequency)
- If reset date falls on holiday: process on next business day (configurable)
- Customer notification: mandatory at least 30 days before rate reset takes effect (or immediately if rate decreases, as per Fair Practices Code)
- For EBLR-linked loans: spread cannot change during loan tenure except in case of "credit event" or "change in credit risk profile" (as per RBI circular)
- Rate change history maintained for each loan account: old rate, new rate, effective date, benchmark, spread, reason
- Bulk rate reset: system must handle rate change for potentially millions of accounts in a single EOD cycle without performance degradation
- Rate change can result in: (a) EMI change (same tenure), (b) tenure change (same EMI), or (c) customer choice -- default option configurable per product

**Example -- Rate Reset Impact:**

Original Loan Terms:
- Loan: Rs 40,00,000 | Rate: 8.90% | Tenure: 240 months | EMI: Rs 37,578
- Outstanding after 24 months: Rs 38,54,000 | Remaining tenure: 216 months

Rate Change: Repo rate reduced by 25 bps; new rate = 8.65%

Option A -- EMI Change (tenure same at 216 months):
- New EMI = Rs 36,846 (reduction of Rs 732/month)

Option B -- Tenure Change (EMI same at Rs 37,578):
- New tenure = 208 months (reduction of 8 months)

**Acceptance Criteria:**
1. Rate master update with maker-checker; effective date recorded
2. All impacted accounts identified within 1 hour of rate update
3. Rate reset applied on correct reset date per account
4. EMI/tenure recalculated accurately
5. Amortization schedule regenerated
6. Customer notification generated (SMS + email + letter)
7. Bulk rate reset for 1 million accounts completed within 4-hour EOD window
8. Rate change history queryable for audit

### 7.6 Interest on Interest (IoI) Calculation

**Requirement ID: LMS-INT-009**
**Title:** Interest on Interest Computation
**Priority:** Must Have

**Description:** Handle scenarios where interest itself becomes interest-bearing, such as during moratorium periods, restructured accounts, or overdue accounts where interest is capitalized.

**Scenarios:**
1. **Moratorium Period IoI**: During moratorium (e.g., COVID moratorium), interest accrues on the outstanding. If this interest is not paid and is capitalized (added to principal), future interest is charged on the enhanced principal, effectively creating IoI.
2. **Overdue Interest Compounding**: For NPA accounts, some institutions capitalize overdue interest annually (subject to RBI norms). Post-RBI penal charges circular (April 2024), compounding of penal charges is prohibited, but interest compounding on unpaid interest may still apply per agreement.
3. **Restructured Account Interest Capitalization**: Interest accrued during moratorium/restructuring period added to principal per restructuring terms.

**COVID IoI Relief (Reference: Government of India scheme, November 2020; Supreme Court order):**
- For loans up to Rs 2 Crore (across 8 categories including MSME, education, housing, consumer, auto, personal, credit card)
- Difference between compound interest and simple interest during moratorium period (March 1 - August 31, 2020) to be refunded/credited
- Banks to claim reimbursement from Government of India

**Business Rules:**
- IoI calculation must be separate from standard interest computation
- System must track simple interest and compound interest separately for IoI scenarios
- IoI credit/debit must be a distinct transaction type for audit trail
- Government claim amount (for COVID IoI) must be tracked and reconciled
- For new moratorium scenarios: system must be able to compute IoI impact and generate customer disclosure

### 7.7 Penal Interest / Penal Charges

**Requirement ID: LMS-INT-010**
**Title:** Penal Charges on Overdue Amounts
**Priority:** Must Have

**Critical Regulatory Change -- RBI Circular on Penal Charges (Reference: DoR.MCS.REC.28/01.01.001/2023-24 dated August 18, 2023; effective April 1, 2024):**

Key mandates:
1. Penalty for non-compliance/default shall be treated as "penal charges" and NOT as "penal interest"
2. Penal charges shall not be levied on the interest component; only on the outstanding principal/installment amount
3. Penal charges must NOT be compounded (no interest on penal charges, no penal charges on penal charges)
4. Penal charges must be "reasonable" and not be used as a revenue enhancement tool
5. Penal charges must be transparently disclosed in loan agreement and KFS
6. Same penal charge norms for same product category; no discrimination
7. Penal charges must be clearly distinguished from interest rate

**System Implementation:**

| Parameter | Pre-April 2024 (Legacy) | Post-April 2024 (Current) |
|---|---|---|
| Terminology | Penal Interest | Penal Charges |
| Basis | % added to interest rate on overdue amounts | Fixed amount or % on overdue installment/principal; not added to interest rate |
| Compounding | Often compounded with regular interest | Strictly no compounding |
| GL Treatment | Part of Interest Income | Separate Penal Charges Income (I04) |
| Tax Treatment | Part of interest for TDS | Separate charge; GST applicable (not TDS) |

**Example -- Penal Charges Calculation:**

Loan Product: Personal Loan
Penal Charge: 2% p.m. on overdue EMI amount (illustrative; must be "reasonable")
Overdue EMI: Rs 25,000 (30 days overdue)

Penal Charge = Rs 25,000 x 2% = Rs 500

If EMI continues overdue for 60 days:
Penal Charge for second month = Rs 25,000 x 2% = Rs 500 (NOT Rs 500 on previous penal charge; no compounding)
Total Penal Charges = Rs 1,000

**Business Rules:**
- Penal charge formula configurable per product: (a) flat amount per overdue EMI, (b) percentage of overdue amount, (c) percentage with cap
- Penal charges calculated as part of EOD, only for overdue accounts
- Penal charges ledger separate from interest ledger
- No penal charges during regulatory moratorium period
- Penal charges disclosed in: sanction letter, KFS, MITC (Most Important Terms and Conditions)
- GST on penal charges: 18% GST applicable as it is classified as a service charge
- Waiver of penal charges: allowed with appropriate authority (maker-checker; authority matrix based on amount)

**Acceptance Criteria:**
1. Penal charges calculated as charges, not interest (post April 2024)
2. No compounding of penal charges
3. Penal charges GL entries are separate from interest entries
4. GST computed on penal charges (not TDS)
5. Penal charge amount disclosed in customer-facing documents (KFS, statement)

### 7.8 Interest Rebate and Concession Processing

**Requirement ID: LMS-INT-011**
**Title:** Interest Rate Concession and Rebate Management
**Priority:** Should Have

**Description:** Handle interest rate concessions granted to specific borrower segments or for specific conditions.

**Common Concession Scenarios in India:**

| Concession Type | Description | Example |
|---|---|---|
| Women Borrower Concession | Rate reduction for women borrowers (sole or co-applicant) | 5-10 bps reduction on home loans |
| Green Building Concession | Rate concession for green-certified properties | 5-10 bps reduction |
| Salary Account Holder | Concession for existing salary account customers | 10-25 bps reduction |
| Credit Score Based | Concession for high CIBIL score (e.g., 750+) | 10-50 bps reduction |
| Bulk/Employer Tie-Up | Concession for employees of approved corporates | 10-50 bps reduction |
| Loyalty Concession | Concession for long-standing customers | Variable |
| Government Scheme | Interest subvention under govt schemes (PMAY, KCC) | Separate handling -- see 7.9 |
| Autopay Concession | Concession for setting up NACH/UPI AutoPay | 5-10 bps |

**Business Rules:**
- Concessions configurable as: (a) reduction in spread over benchmark, or (b) absolute rate override
- Multiple concessions can be stacked (cumulative) up to a configurable minimum rate
- Concession has a validity period: some are for entire tenure, some for initial period only
- Concession withdrawal conditions: if NACH cancelled, if salary account closed, etc.
- Concession tracking for P&L impact analysis (interest income foregone)
- Maker-checker for concession approval; authority matrix based on concession value
- Concession reported as "interest income foregone" in MIS

### 7.9 Interest Subvention Schemes

**Requirement ID: LMS-INT-012**
**Title:** Government Interest Subvention Scheme Processing
**Priority:** Must Have

**Description:** The system must handle various Government of India and State Government interest subvention schemes where the government bears part of the interest cost.

**Key Subvention Schemes:**

**a) PMAY-CLSS (Pradhan Mantri Awas Yojana -- Credit Linked Subsidy Scheme):**
- Subsidy on home loan interest for EWS/LIG/MIG categories
- Subsidy amount calculated as NPV of interest reduction over loan tenure (max 20 years)
- Subsidy credited to borrower's loan account, reducing principal outstanding
- Subsidy amount calculated by NHB/HUDCO (Central Nodal Agencies)

| Category | Household Income | Max Loan for Subsidy | Interest Subsidy | Max Carpet Area |
|---|---|---|---|---|
| EWS | Up to Rs 3 Lakh | Rs 6 Lakh | 6.50% | 30 sq.m. |
| LIG | Rs 3-6 Lakh | Rs 6 Lakh | 6.50% | 60 sq.m. |
| MIG-I | Rs 6-12 Lakh | Rs 9 Lakh | 4.00% | 160 sq.m. |
| MIG-II | Rs 12-18 Lakh | Rs 12 Lakh | 3.00% | 200 sq.m. |

Note: PMAY-CLSS for MIG closed Dec 2024; verify current status of scheme extension.

**System Requirements for PMAY:**
- PMAY eligibility assessment module
- Subsidy NPV calculation engine
- Claim submission to NHB/HUDCO via prescribed format
- Subsidy receipt tracking and crediting to borrower's loan account
- Modified amortization schedule post-subsidy credit
- Reporting: quarterly PMAY claims summary, disbursement-wise tracking
- Clawback tracking: if borrower found ineligible, subsidy recovery mechanism

**b) KCC Interest Subvention (Agricultural Loans):**
- Government provides 2% interest subvention for short-term crop loans up to Rs 3 Lakh
- Additional 3% prompt repayment incentive (effective rate for farmer: 4% p.a.)
- Bank charges farmer 7% p.a.; government reimburses 2% + 3% (if prompt)
- Subvention claim submitted to NABARD quarterly

**System Requirements for KCC Subvention:**
- Track eligible crop loan accounts (limit up to Rs 3 Lakh)
- Compute subvention amount (2% on average outstanding for the period)
- Track prompt repayment incentive eligibility (repaid within crop season)
- Generate NABARD claim file in prescribed format
- Reconcile subvention received from NABARD with claims submitted

**c) Education Loan Interest Subsidy (Central Sector Interest Subsidy Scheme):**
- Full interest subsidy during moratorium period for economically weaker section students
- Annual family income up to Rs 4.5 Lakh
- Applicable for courses in India at recognized institutions
- Subsidy claimed from Government via Canara Bank (nodal bank)

**Business Rules:**
- Subvention schemes configurable as templates with eligibility criteria, rates, claim formats
- Multiple subvention schemes can apply to the same loan (rare but possible)
- Subvention income tracked separately from regular interest income
- Claim filing frequency: quarterly/half-yearly as per scheme requirements
- Reconciliation of claims submitted vs amounts received from government agencies
- Pending claims aging report for follow-up

### 7.10 TDS on Interest (Section 194A)

**Requirement ID: LMS-INT-013**
**Title:** TDS Deduction on Interest Payments Under Section 194A
**Priority:** Must Have

**Description:** When the institution pays interest to depositors on loan-linked deposits or when TDS is applicable on interest payments to specific parties, the system must compute and deduct TDS.

**Note on TDS Applicability in Lending Context:**
- TDS under Section 194A is primarily on interest paid by banks/institutions on deposits
- In loan context, TDS relevance arises for: (a) interest on security deposits held, (b) interest on excess amounts held, (c) interest paid on delayed refunds
- For NBFCs paying interest on NCDs/deposits: TDS mandatory if interest exceeds Rs 5,000 p.a.
- For banks: TDS on interest on deposits exceeding Rs 40,000 p.a. (Rs 50,000 for senior citizens)

**System Requirements:**
- Compute annual interest paid/payable per payee
- Deduct TDS at applicable rate (10% for residents with PAN; 20% without PAN; Nil for Form 15G/15H holders)
- Generate TDS certificate (Form 16A) quarterly
- File quarterly TDS returns (Form 26Q)
- Track Form 15G/15H submissions and validate eligibility
- Handle TDS on penal charges if classified as income

**Business Rules:**
- TDS computation at payee PAN level (aggregate across all accounts of the same PAN)
- TDS deduction timing: at the time of credit or payment, whichever is earlier
- Interest certificate (for borrower's tax filing): separate from TDS -- covered in Module 7 (Interest vs Principal Allocation)
- TDS compliance: Form 26Q filing by 31st of month following the quarter
- TDS rate changes: configurable in tax rate master with effective dates

### 7.11 NPA Interest Recognition Reversal (IRAC Norms)

**Requirement ID: LMS-INT-014**
**Title:** Interest Income Reversal and Non-Recognition for NPA Accounts
**Priority:** Must Have

**RBI IRAC Norms Reference:** Master Direction - Classification of Accounts and Provisioning (DOR.STR.REC.4/21.04.048/2021-22)

**Rules:**
1. When a loan account is classified as NPA (90+ DPD for most categories):
   - All interest income recognized in the current financial year but not yet realized (collected) must be reversed from P&L
   - Interest income recognized in previous financial years but not realized: provision must be made (or reversed if accounting policy allows)
2. Post-NPA classification:
   - Interest accrual continues but only in memorandum (off-balance sheet) records
   - Interest income recognized in P&L only on cash basis (when actually received)
3. If NPA account is upgraded (becomes performing again):
   - All arrears of interest and principal fully paid
   - Interest accrual resumes in P&L
   - Previously reversed interest is NOT automatically re-recognized; only future accrual goes to P&L

**Specific Rules for Agricultural Loans:**
- For short-duration crops: NPA if overdue for 2 crop seasons beyond the due date
- For long-duration crops: NPA if overdue for 1 crop season beyond the due date
- Interest recognition norms same as above once classified as NPA

**System Implementation:**

| Event | System Action |
|---|---|
| Account classified as NPA | Reverse all unrealized interest from Interest Income GL; move to Memorandum |
| Daily accrual on NPA account | Post to Memorandum (off-BS) only; NO P&L impact |
| Cash collection on NPA account | Recognize interest income to the extent of interest collected (per allocation waterfall) |
| NPA upgraded to Standard | Resume normal accrual to P&L; do not retrospectively recognize memorandum interest |

**Acceptance Criteria:**
1. Interest reversal automated on NPA classification within the same EOD cycle
2. No interest income leakage into P&L for NPA accounts
3. Memorandum interest tracked accurately for recovery and provisioning purposes
4. Cash-basis recognition correctly applied for NPA collections
5. Audit trail for every interest reversal with NPA classification reference

### 7.12 Pre-EMI Interest for Under-Construction Property Loans

**Requirement ID: LMS-INT-015**
**Title:** Pre-EMI Interest (PEMI) Calculation and Collection
**Priority:** Must Have

**Description:** For home loans disbursed against under-construction properties, only interest is charged on the disbursed amount until full disbursement or EMI commencement (whichever is configured).

**Calculation:**
- PEMI = Disbursed Amount (cumulative tranches) x Annual Rate / 12 (for monthly PEMI)
- Or daily calculation: Disbursed Amount x Annual Rate / 365 x Number of Days

**Example:**
- Sanctioned: Rs 60,00,000
- Tranche 1 (April 1): Rs 15,00,000
- Tranche 2 (July 1): Rs 20,00,000
- Rate: 8.75% p.a.

PEMI for April = 15,00,000 x 8.75% / 12 = Rs 10,937.50
PEMI for May = 15,00,000 x 8.75% / 12 = Rs 10,937.50
PEMI for July = 35,00,000 x 8.75% / 12 = Rs 25,520.83

**Business Rules:**
- PEMI collected monthly via NACH or cheque
- PEMI amount changes with each tranche disbursement
- Some institutions offer "No PEMI" option: interest accumulated and capitalized to principal at EMI commencement (increases total cost)
- Full EMI commencement: triggered by either (a) final tranche disbursement + configurable grace period, or (b) specific date in loan agreement, or (c) customer request
- Upon EMI commencement: amortization schedule generated on the total disbursed + capitalized interest (if "No PEMI" option)
- PEMI not eligible for Section 24 deduction until possession is received (as per Income Tax Act); system must note this in interest certificate

### 7.13 Moratorium Period Interest Handling

**Requirement ID: LMS-INT-016**
**Title:** Interest Treatment During Moratorium Periods
**Priority:** Must Have

**Description:** Detailed interest handling rules for various moratorium scenarios.

**Moratorium Interest Options (configurable per restructuring scheme):**

| Option | Description | Impact on Loan |
|---|---|---|
| Interest Serviced | Borrower continues to pay interest during moratorium; only principal repayment deferred | No increase in total loan cost; tenure extended |
| Interest Capitalized | Interest accrued during moratorium added to principal at moratorium end | Increased principal; higher EMI or longer tenure |
| Interest Funded (FITL) | Interest converted to a separate Funded Interest Term Loan | Original loan principal unchanged; separate FITL with own repayment |
| Interest Waived | Interest during moratorium period waived (rare; only for specific govt/regulatory schemes) | Loss to lender; charged to P&L as interest waiver expense |

**Business Rules:**
- Option selection: (a) at product level for regulatory moratoriums, or (b) per borrower for bilateral restructuring
- Capitalized interest calculation: accurate compound interest computation for the moratorium period
- FITL creation: system must support creation of a linked/subsidiary loan account with its own schedule
- Post-moratorium recalculation: EMI/tenure adjusted based on the enhanced principal (if capitalized) or original principal (if serviced/FITL)
- IND AS 109 impact: modification accounting may be required if moratorium represents a substantial modification; present value impact computed using original EIR

**Acceptance Criteria:**
1. All four moratorium interest options supported
2. Capitalized interest computed accurately
3. FITL created as linked account with independent amortization
4. Post-moratorium EMI/tenure recalculation is accurate
5. Customer notification of new terms post-moratorium

---
