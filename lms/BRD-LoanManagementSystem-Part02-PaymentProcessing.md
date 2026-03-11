# MODULE 2: LOAN PAYMENT PROCESSING

## 5. LOAN PAYMENT PROCESSING

### 5.1 Overview

The Loan Payment Processing module handles all aspects of EMI computation, amortization schedule management, payment receipt, allocation, and exception handling across the complete loan lifecycle. This module must support the diverse payment structures prevalent in Indian lending -- from standard EMI-based retail loans to bullet repayment structures, step-up/step-down EMIs, balloon payments, and moratorium-linked repayment schedules.

### 5.2 EMI Calculation Engine

#### 5.2.1 Reducing Balance EMI Calculation

**Requirement ID: LMS-PAY-001**
**Title:** EMI Calculation Using Reducing Balance Method
**Priority:** Must Have
**Description:** The system shall calculate EMI using the standard reducing balance (annuity) formula. This is the default and most common method for Indian retail loans (home loans, personal loans, vehicle loans, LAP).

**Formula:**

EMI = P x r x (1 + r)^n / ((1 + r)^n - 1)

Where:
- P = Principal loan amount
- r = Monthly interest rate (annual rate / 12)
- n = Total number of monthly installments

**Example Calculation:**

- Loan Amount: Rs 30,00,000
- Interest Rate: 9.00% p.a.
- Tenure: 20 years (240 months)
- Monthly Rate (r): 9.00% / 12 = 0.75% = 0.0075

EMI = 30,00,000 x 0.0075 x (1.0075)^240 / ((1.0075)^240 - 1)
EMI = 30,00,000 x 0.0075 x 6.0092 / (6.0092 - 1)
EMI = 30,00,000 x 0.0075 x 6.0092 / 5.0092
EMI = 1,35,069 / 5.0092
EMI = Rs 26,965 (rounded)

**Business Rules:**
- EMI rounding: Round to nearest rupee (configurable: floor, ceiling, or standard rounding)
- Rounding difference adjusted in the last EMI
- EMI recalculated on: (a) floating rate change, (b) part-prepayment, (c) tenure change, (d) restructuring
- System must support both monthly and non-monthly EMI frequencies (quarterly, half-yearly, annual -- common in agricultural and MSME loans)
- For daily reducing balance: interest computed on the actual daily outstanding balance, but EMI remains fixed monthly

**Acceptance Criteria:**
1. EMI calculated matches standard financial calculator output within Rs 1 tolerance
2. Amortization schedule total principal equals original loan amount
3. Last EMI adjustment accounts for all rounding differences
4. System supports EMI frequencies: monthly, quarterly, half-yearly, yearly

#### 5.2.2 Flat Rate EMI Calculation

**Requirement ID: LMS-PAY-002**
**Title:** Flat Rate EMI Calculation
**Priority:** Must Have
**Description:** The system shall support flat rate EMI calculation, commonly used for consumer durable loans, two-wheeler loans, and some personal loan products.

**Formula:**

Total Interest = P x R x T
EMI = (P + Total Interest) / (T x 12)

Where:
- P = Principal amount
- R = Annual flat interest rate
- T = Tenure in years

**Example:**
- Loan Amount: Rs 1,00,000 (Consumer Durable)
- Flat Rate: 12% p.a.
- Tenure: 2 years (24 months)
- Total Interest = Rs 1,00,000 x 12% x 2 = Rs 24,000
- EMI = (Rs 1,00,000 + Rs 24,000) / 24 = Rs 5,167

**Business Rules:**
- System must also compute the equivalent reducing balance rate for regulatory disclosure (XIRR/APR)
- RBI's Digital Lending Guidelines mandate disclosure of APR (Annual Percentage Rate) in KFS (Key Fact Statement), so the system must compute APR even for flat rate products
- APR computation must include all charges (processing fee, insurance premium, documentation charges) as per RBI's all-inclusive interest rate concept

**Acceptance Criteria:**
1. Flat rate EMI matches manual calculation
2. System computes and displays equivalent reducing balance rate / APR
3. KFS generated for flat rate products includes APR disclosure

#### 5.2.3 Rule of 78 (Sum of Digits) Method

**Requirement ID: LMS-PAY-003**
**Title:** Rule of 78 Interest Allocation
**Priority:** Should Have
**Description:** For certain legacy products and specific NBFC products, the system must support interest allocation using the Rule of 78 (Sum of Digits) method.

**Formula:**

Interest for month k = Total Interest x (n - k + 1) / Sum(1 to n)

Where n = total number of installments, k = installment number

**Example (24-month loan, Total Interest = Rs 24,000):**
- Sum of digits (1 to 24) = 24 x 25 / 2 = 300
- Month 1 Interest = Rs 24,000 x 24/300 = Rs 1,920
- Month 2 Interest = Rs 24,000 x 23/300 = Rs 1,840
- Month 24 Interest = Rs 24,000 x 1/300 = Rs 80

**Business Rules:**
- Rule of 78 front-loads interest, which is important for prepayment calculations
- For prepayment, unearned interest = Total Interest x [Sum of remaining digits / Sum of all digits]
- This method is increasingly less common but required for legacy portfolio migration
- System must flag if Rule of 78 is used for regulatory reporting as it may attract RBI scrutiny for transparency

#### 5.2.4 Step-Up and Step-Down EMI

**Requirement ID: LMS-PAY-004**
**Title:** Step-Up and Step-Down EMI Structures
**Priority:** Should Have

**Description:** The system shall support EMI structures where the installment amount changes at predefined intervals.

**Step-Up EMI (common for young professionals -- home loans):**
- Year 1-3: Base EMI
- Year 4-6: Base EMI + 10%
- Year 7-10: Base EMI + 20%
- Year 11 onwards: Base EMI + 30%

**Step-Down EMI (common for near-retirement borrowers):**
- Year 1-5: Higher EMI
- Year 6-10: Base EMI - 10%
- Year 11-15: Base EMI - 20%

**Business Rules:**
- Step-up/step-down percentage or absolute amount configurable at product level
- Step intervals configurable (yearly, 2-yearly, 3-yearly)
- Total principal repayment must equal the original loan amount
- Amortization schedule must reflect the step structure
- Eligibility assessment must consider the highest EMI for FOIR (Fixed Obligations to Income Ratio) calculation

#### 5.2.5 Balloon Payment / Bullet Repayment

**Requirement ID: LMS-PAY-005**
**Title:** Balloon and Bullet Payment Structures
**Priority:** Should Have

**Description:** Support for loans with a large final payment (balloon) or interest-only payments with principal at maturity (bullet).

**Types:**
1. **Bullet Repayment**: Interest paid periodically; entire principal at maturity (common for overdraft facilities, some MSME loans)
2. **Balloon Payment**: Regular EMIs with a predetermined lump-sum at maturity
3. **Interest-Only Period**: Initial period with only interest payments, followed by amortizing EMIs (common for LAP, construction-linked home loans)

**Business Rules:**
- Balloon amount configurable as percentage of principal or absolute amount
- Interest-only period configurable in months
- System must track bullet payment maturity date and trigger reminder notices
- Provisioning must account for concentration risk in bullet repayment structures

### 5.3 Amortization Schedule Generation

**Requirement ID: LMS-PAY-006**
**Title:** Comprehensive Amortization Schedule
**Priority:** Must Have

**Description:** The system shall generate a detailed amortization schedule at loan origination and regenerate it whenever there is a material change (rate change, prepayment, restructuring, moratorium).

**Amortization Schedule Fields:**

| Field | Description |
|---|---|
| Installment Number | Sequential number (1, 2, 3, ... n) |
| Due Date | EMI due date for the installment |
| Opening Balance | Principal outstanding at start of period |
| EMI Amount | Total installment amount |
| Interest Component | Interest portion of EMI |
| Principal Component | Principal portion of EMI |
| Closing Balance | Principal outstanding after payment |
| Cumulative Interest Paid | Running total of interest paid |
| Cumulative Principal Paid | Running total of principal repaid |

**Example Amortization (First 6 months):**

Loan: Rs 50,00,000 | Rate: 8.50% p.a. | Tenure: 240 months | EMI: Rs 43,391

| Inst# | Due Date | Opening Bal | EMI | Interest | Principal | Closing Bal |
|---|---|---|---|---|---|---|
| 1 | 05-Apr-2026 | 50,00,000 | 43,391 | 35,417 | 7,974 | 49,92,026 |
| 2 | 05-May-2026 | 49,92,026 | 43,391 | 35,360 | 8,031 | 49,83,995 |
| 3 | 05-Jun-2026 | 49,83,995 | 43,391 | 35,303 | 8,088 | 49,75,907 |
| 4 | 05-Jul-2026 | 49,75,907 | 43,391 | 35,246 | 8,145 | 49,67,762 |
| 5 | 05-Aug-2026 | 49,67,762 | 43,391 | 35,188 | 8,203 | 49,59,559 |
| 6 | 05-Sep-2026 | 49,59,559 | 43,391 | 35,130 | 8,261 | 49,51,298 |

**Business Rules:**
- Amortization schedule generated for the full tenure at disbursement
- Schedule regenerated on: rate change (floating rate reset), part-prepayment, tenure change, restructuring, moratorium end
- Schedule must be available for download (PDF, Excel) via customer portal and branch interface
- Due date falls on holiday: configurable behavior -- (a) advance to previous working day, (b) postpone to next working day, (c) no adjustment (let NACH present on next working day)
- System must maintain version history of all amortization schedule generations with reason for regeneration

**Acceptance Criteria:**
1. Sum of all principal components equals the original loan amount (within Rs 1 rounding tolerance)
2. Closing balance of last installment is zero (or last EMI adjusted to achieve zero)
3. Schedule reflects actual calendar dates with holiday adjustments
4. Version history maintained with at least 24 months of schedule versions

### 5.4 Payment Allocation Waterfall

**Requirement ID: LMS-PAY-007**
**Title:** Configurable Payment Allocation Waterfall
**Priority:** Must Have

**Description:** When a payment is received, the system shall allocate it across various outstanding components in a predefined priority sequence (waterfall). The waterfall must be configurable at the product level and can be overridden at entity level.

**Default Payment Allocation Waterfall (configurable):**

| Priority | Component | Description |
|---|---|---|
| 1 | Costs and Charges Recovered | Legal costs, inspection charges, insurance premium recovery |
| 2 | Penal Charges | Penal charges on overdue (post Apr 2024: charges, not interest) |
| 3 | Overdue Fees | Cheque bounce charges, NACH return charges |
| 4 | Overdue Interest (oldest first) | Interest portion of oldest unpaid EMI |
| 5 | Overdue Principal (oldest first) | Principal portion of oldest unpaid EMI |
| 6 | Current Month Interest | Interest due for current billing cycle |
| 7 | Current Month Principal | Principal due for current billing cycle |
| 8 | Future Principal (Prepayment) | Any excess applied to principal reduction |

**Alternative Waterfall (common for NBFCs):**

| Priority | Component |
|---|---|
| 1 | Penal Charges |
| 2 | Fees and Charges |
| 3 | Interest (oldest EMI first) |
| 4 | Principal (oldest EMI first) |
| 5 | Current EMI Interest |
| 6 | Current EMI Principal |
| 7 | Advance Principal |

**Business Rules:**
- Waterfall is configurable per product; system supports at least 10 priority levels
- Within each priority level, allocation follows FIFO (First In, First Out) for overdue installments
- Partial payment: if received amount is less than total outstanding, allocate as per waterfall until exhausted
- System must log the complete allocation breakup for every payment
- For NPA accounts: allocation waterfall may differ (some institutions apply principal first to reduce NPA outstanding)
- Allocation waterfall for restructured accounts may be different from standard accounts
- Regulatory constraint: RBI's circular on penal charges (effective April 1, 2024) requires that penal charges are not compounded (i.e., no interest on penal charges); allocation must ensure this

**Example -- Partial Payment Allocation:**

Outstanding on account:
- Overdue EMI 1 (60 days overdue): Principal Rs 8,000 + Interest Rs 35,000 = Rs 43,000
- Overdue EMI 2 (30 days overdue): Principal Rs 8,050 + Interest Rs 34,950 = Rs 43,000
- Penal Charges: Rs 2,500
- Current EMI due: Rs 43,000
- Total Outstanding: Rs 1,31,500

Payment Received: Rs 90,000

Allocation using default waterfall:
1. Penal Charges: Rs 2,500 (Balance remaining: Rs 87,500)
2. Overdue Interest -- EMI 1: Rs 35,000 (Balance: Rs 52,500)
3. Overdue Principal -- EMI 1: Rs 8,000 (Balance: Rs 44,500)
4. Overdue Interest -- EMI 2: Rs 34,950 (Balance: Rs 9,550)
5. Overdue Principal -- EMI 2: Rs 8,050 (Balance: Rs 1,500)
6. Current Interest: Rs 1,500 (Balance: Rs 0) -- partial current interest payment

Result: EMI 1 fully settled, EMI 2 fully settled, partial current month interest paid.
DPD impact: DPD reduces as oldest EMI is cleared.

**Acceptance Criteria:**
1. Payment allocation follows configured waterfall exactly
2. Allocation breakup logged for audit trail
3. Partial payments allocated correctly with remaining amounts tracked
4. DPD recalculated post allocation
5. Waterfall can be modified by authorized users with maker-checker; changes apply prospectively

### 5.5 Payment Modes

#### 5.5.1 NACH / ECS Mandates

**Requirement ID: LMS-PAY-008**
**Title:** NACH Mandate Registration and EMI Collection
**Priority:** Must Have

**Description:** The system shall support NPCI's NACH (National Automated Clearing House) debit mandate for automated EMI collection. NACH has replaced ECS for most banks; legacy ECS support may be needed for older mandates.

**NACH Mandate Lifecycle:**

1. **Mandate Creation**: Generate mandate form (physical or e-mandate via net banking / debit card / Aadhaar-based)
2. **Mandate Registration**: Submit to NACH system via sponsor bank; receive UMRN (Unique Mandate Reference Number)
3. **Mandate Confirmation**: System records UMRN and activation status
4. **Periodic Presentation**: Present EMI debit instruction to NACH on due date (or configurable days before due date)
5. **Response Processing**: Process NACH response file -- success or return (with return reason codes)
6. **Mandate Amendment**: Handle changes in amount, frequency, bank account
7. **Mandate Cancellation**: On loan closure, prepayment, or borrower request

**NACH Return Reason Codes (Key Codes):**

| Return Code | Description | System Action |
|---|---|---|
| 01 | Insufficient Funds | Mark EMI as bounced; trigger bounce charges; retry logic |
| 02 | Account Closed | Alert operations; initiate alternate payment mode |
| 03 | Account Does Not Exist | Alert operations; verify bank details |
| 04 | NRI Account | May need different mandate type |
| 05 | Party Deceased | Trigger insurance claim workflow (if credit life); alert collections |
| 06 | Account Frozen by Order of Authority | Alert compliance; legal review |
| 10 | Mandate Not Registered | Re-register mandate |
| 68 | Mandate Cancelled by Customer | Follow up with customer; alternate payment |

**Business Rules:**
- NACH presentation date: configurable per product (typically EMI due date or 1-2 days after to allow salary credit)
- Maximum mandate amount: should be set higher than EMI to accommodate rate changes (typically EMI x 1.25 or 1.5)
- Auto-retry on insufficient funds: configurable number of retries (1-3) with configurable interval (e.g., retry after 3 days, 7 days)
- Bounce charges: configurable per product (subject to RBI circular on penal charges; must be "reasonable and proportionate")
- For RBI's penal charges norms (effective April 1, 2024): bounce charges are classified as "charges" not "interest"; cannot be compounded; must be disclosed upfront in loan agreement and KFS
- e-NACH mandate: support Aadhaar-based e-mandate (up to Rs 1 lakh per transaction without debit card/net banking; higher with additional authentication)

**Acceptance Criteria:**
1. System generates NACH mandate in prescribed NPCI format
2. UMRN is stored and linked to loan account
3. Monthly presentation file generated automatically on configured schedule
4. Return file processing is automated with appropriate system actions per return code
5. Retry logic executes automatically based on configured rules
6. Mandate cancellation triggered automatically on loan closure

#### 5.5.2 UPI AutoPay

**Requirement ID: LMS-PAY-009**
**Title:** UPI AutoPay / UPI Mandate for EMI Collection
**Priority:** Must Have

**Description:** The system shall support UPI AutoPay (recurring payment mandates) as per NPCI UPI 2.0 specifications for automated EMI collection.

**Business Rules:**
- UPI mandate creation via customer's UPI app (Google Pay, PhonePe, Paytm, BHIM, bank apps)
- Mandate parameters: amount (fixed or variable up to max), frequency, start date, end date
- For variable amount mandates: actual EMI amount presented each cycle
- Pre-debit notification: mandatory 24-hour advance notification to customer before debit (as per NPCI guidelines)
- Single-block-multi-debit: supported for certain loan products
- UPI mandate limit: currently Rs 1,00,000 per transaction for loan repayment category (verify latest NPCI circular)
- Mandate status tracking: Created, Approved, Active, Paused, Revoked, Expired
- Integration with UPI mandate management system via PSP (Payment Service Provider) APIs

#### 5.5.3 NEFT/RTGS/IMPS Payments

**Requirement ID: LMS-PAY-010**
**Title:** NEFT/RTGS/IMPS Payment Receipt and Allocation
**Priority:** Must Have

**Business Rules:**
- System generates unique virtual account number or loan account reference for NEFT/RTGS/IMPS payments
- Payment received via NEFT/RTGS identified by UTR (Unique Transaction Reference) number
- Automatic matching: system matches incoming NEFT/RTGS credit to loan account using (a) virtual account number, (b) loan account number in remarks, or (c) manual matching
- Unmatched payments parked in Suspense Account (A11) with aging and escalation
- RTGS: real-time credit; NEFT: batch settlement (half-hourly); IMPS: real-time
- System must record UTR for reconciliation
- Receipt generation upon successful allocation

#### 5.5.4 Cash and Cheque Payments

**Requirement ID: LMS-PAY-011**
**Title:** Cash and Cheque Payment Processing
**Priority:** Must Have

**Cash Payment:**
- Cash received at branch; teller enters payment against loan account
- Immediate allocation per waterfall rules
- Cash receipt generated with payment breakup
- For amounts Rs 50,000 and above: PAN mandatory (Income Tax Act, Section 269ST -- Rs 2 lakh limit for cash transactions)
- CTR (Cash Transaction Report) triggered for cash receipts Rs 10 lakh and above (RBI KYC/AML guidelines)

**Cheque Payment:**
- Cheque accepted; system creates clearing entry
- Cheque sent for clearing (CTS -- Cheque Truncation System)
- On clearing: allocated to loan per waterfall
- On return: cheque bounce entry; bounce charges levied; customer notified
- Cheque return reason codes tracked and reported

#### 5.5.5 Post-Dated Cheque (PDC) Management

**Requirement ID: LMS-PAY-012**
**Title:** PDC Lifecycle Management
**Priority:** Should Have

**Description:** For NBFC and older bank loan products, PDC collection is common. The system shall manage the full PDC lifecycle.

**PDC Lifecycle:**
1. **Collection**: Record PDC details (cheque number, bank, date, amount) at loan booking
2. **Safekeeping**: Track physical custody (vault, branch, centralized hub)
3. **Presentation**: Auto-generate list of PDCs due for presentation (configurable days before due date, typically 2-3 banking days)
4. **Clearing**: Track clearing status
5. **Return Handling**: Process bounced PDCs; initiate bounce charges and collection follow-up
6. **Return to Customer**: On loan closure/prepayment, return unused PDCs with acknowledgment

**Business Rules:**
- PDC presentation can be configured to run automatically or manually
- System tracks PDC inventory with custodian details
- Regulatory: Section 138 of Negotiable Instruments Act -- bounced cheques can lead to criminal proceedings; system must track PDC bounces for legal action
- PDC presentation suspended when loan is fully repaid or when NACH/UPI mandate is active
- Unused PDC return is part of loan closure checklist

### 5.6 Partial Payment Handling

**Requirement ID: LMS-PAY-013**
**Title:** Partial Payment Processing and DPD Impact
**Priority:** Must Have

**Description:** When a borrower pays less than the full EMI amount, the system must handle partial payments correctly.

**Business Rules:**
- Partial payment allocated per waterfall; remaining balance continues as overdue
- DPD impact: partial payment does not reset DPD unless the oldest overdue EMI is fully cleared
  - Example: If 2 EMIs overdue (60 DPD) and borrower pays 1.5 EMIs, the oldest EMI is cleared (DPD resets for that installment), but the second EMI remains partially unpaid
- For SMA/NPA classification: DPD counted based on oldest unpaid installment amount
- System must track partial payments distinctly from full EMI payments for collection analytics
- Partial payment threshold: some institutions define a minimum payment amount (e.g., 50% of EMI) below which the payment is treated as "token payment" -- configurable

**Acceptance Criteria:**
1. Partial payments allocated correctly per waterfall
2. DPD recalculated accurately post partial payment
3. Audit trail shows allocation breakup
4. Collection team alerted for accounts with repeated partial payments

### 5.7 Advance EMI and Lump-Sum Payments

**Requirement ID: LMS-PAY-014**
**Title:** Advance EMI and Lump-Sum Payment Processing
**Priority:** Must Have

**Description:** The system must handle payments exceeding the current due amount.

**Scenarios:**

1. **Advance EMI Payment**: Borrower pays multiple future EMIs in advance
   - Interest for future months not yet accrued; system can either: (a) hold as advance/credit balance and apply on future due dates, or (b) apply immediately as part-prepayment
   - Configurable per product: default is to hold as advance and adjust against future EMIs

2. **Lump-Sum Part-Prepayment**: Borrower makes additional payment to reduce principal
   - See Module 6 (Early Closure/Prepayment) for detailed handling
   - System must ask/apply: reduce tenure OR reduce EMI (borrower choice, configurable default)

**Business Rules:**
- Advance EMI held in a separate "Advance EMI/Credit Balance" ledger head
- Interest income not recognized on advance EMI until the actual EMI due date
- If borrower has advance EMI credit and a new EMI falls due, system auto-adjusts from credit balance
- If borrower defaults after paying advance EMIs, advance EMI credits are applied before marking as overdue
- Advance EMI payments do not earn interest for the borrower (unless specifically agreed)

### 5.8 Payment Holiday / Moratorium Processing

**Requirement ID: LMS-PAY-015**
**Title:** Payment Holiday and Moratorium Management
**Priority:** Must Have

**Description:** The system must handle various moratorium and payment holiday scenarios prevalent in Indian lending.

**Moratorium Types:**

| Type | Description | Interest Treatment | Common Use Case |
|---|---|---|---|
| Pre-EMI Period | Period between disbursement and EMI commencement for under-construction properties | Interest charged monthly (pre-EMI interest) or accumulated | Home loans -- under construction |
| Initial Moratorium | Grace period at loan start where no EMI is due | Interest accrues; may be added to principal (capitalized) or collected separately | Education loans (moratorium during study + 6-12 months) |
| Interim Moratorium | Mid-tenure payment holiday granted by lender | Interest accrues; typically capitalized into principal | COVID moratorium; natural disaster relief |
| Regulatory Moratorium | Mandated by RBI/Government | Governed by specific circular (e.g., RBI COVID moratorium March-August 2020) | COVID-19; natural calamities |

**COVID Moratorium -- Specific Handling (Reference: RBI circulars dated March 27, 2020 and May 23, 2020):**
- 6-month moratorium on term loan installments (March 1 - August 31, 2020)
- Interest continued to accrue during moratorium
- Supreme Court ruling on "Interest on Interest" (IoI): Ex-gratia relief for loans up to Rs 2 Cr -- compound interest charged during moratorium refunded/adjusted (reference: Government of India scheme for IoI relief)
- System must have handled IoI calculation, refund, and government claim submission

**Business Rules for Moratorium Processing:**
- During moratorium: EMI due date generation suspended; interest accrual continues
- Post-moratorium options (configurable): (a) extend tenure by moratorium period, (b) increase EMI to accommodate accrued interest, (c) capitalize accrued interest and recalculate EMI, (d) recover accrued interest as lump-sum
- Amortization schedule regenerated post-moratorium
- DPD frozen during regulatory moratorium (as per RBI guidance)
- Asset classification: standstill on DPD during regulatory moratorium
- System must support retroactive moratorium application (apply moratorium to accounts after the fact, as was required during COVID)

**Education Loan Moratorium (specific rules):**
- Moratorium = Course period + 12 months (or 6 months after getting job, whichever is earlier)
- During moratorium: simple interest accrues
- Interest accumulated during moratorium may be capitalized or first EMIs may be interest-only
- Government interest subvention for economically weaker section students (under Central Sector Interest Subsidy Scheme)

**Acceptance Criteria:**
1. Moratorium can be applied to individual accounts or bulk (filtered by product, branch, segment)
2. Interest accrual continues correctly during moratorium
3. Post-moratorium EMI recalculation with all options (extend tenure, increase EMI, capitalize interest)
4. Amortization schedule regenerated automatically
5. DPD standstill applied during regulatory moratorium
6. Audit trail for moratorium application and removal

### 5.9 Restructured Loan Payment Handling

**Requirement ID: LMS-PAY-016**
**Title:** Restructured Loan Repayment Management
**Priority:** Must Have

**Description:** The system must handle payments for loans restructured under various RBI frameworks.

**Restructuring Scenarios in Indian Banking:**

1. **RBI Framework for Resolution of Stressed Assets (June 7, 2019, updated)**: Applies to accounts with aggregate exposure Rs 1,500 Cr and above from banking system; Resolution Plan (RP) within 180 days of default
2. **MSME Restructuring**: One-time restructuring for MSME borrowers (various circulars; latest as per prevailing RBI norms)
3. **COVID-19 Restructuring (Resolution Framework 1.0 and 2.0)**: One-time restructuring for COVID-impacted borrowers
4. **Bilateral Restructuring**: Lender-level restructuring for smaller accounts

**Restructuring System Requirements:**

| Requirement | Description |
|---|---|
| Revised Repayment Schedule | New amortization schedule with modified terms (tenure, rate, EMI, moratorium) |
| DCCO Extension | For project loans: extension of Date of Commencement of Commercial Operations |
| Conversion of Debt | Convert part of debt to equity or FITL (Funded Interest Term Loan) |
| FITL Creation | Separate loan account for funded interest; own repayment schedule |
| Provisioning | Higher provisioning for restructured accounts (5% for standard restructured -- banks) |
| Downgrade Tracking | If restructured account defaults again, downgrade to NPA immediately (no 90-day cure) |
| Inter-Creditor Agreement (ICA) | For consortium/multiple banking: ICA implementation tracking |

**Business Rules:**
- Restructured account flagged in system with restructuring type, date, and terms
- Original terms preserved for comparison and reporting
- Separate CRILC reporting for restructured accounts
- If restructured standard account becomes overdue > 30 days post-restructuring: immediate NPA classification (for accounts restructured under some frameworks -- verify per applicable circular)
- Monitoring period post-restructuring: typically 12 months; must be tracked in system
- System must support creation of subordinate/linked accounts (e.g., FITL, equity conversion tracking)

### 5.10 Excess Payment and Refund Processing

**Requirement ID: LMS-PAY-017**
**Title:** Excess Payment Handling and Refund Workflow
**Priority:** Must Have

**Business Rules:**
- Excess payment (amount received > total outstanding on account) parked in Excess Collection GL (L02)
- Refund initiated: (a) automatically for amounts exceeding configurable threshold (e.g., Rs 100), or (b) on customer request
- Refund modes: NEFT to registered bank account; no cash refund for amounts > Rs 5,000
- Refund workflow: initiate request -> verify amount -> maker-checker approval -> payment instruction to CBS/payment system -> confirmation
- TDS applicability: if excess includes interest component that has had TDS deducted, TDS refund/adjustment must be handled
- Refund timeline: within 7 working days of identification (as per Fair Practices Code)
- System must track refund status and generate alerts for aged pending refunds

### 5.11 Failed Payment Retry and Bounce Handling

**Requirement ID: LMS-PAY-018**
**Title:** Payment Bounce Management and Retry Logic
**Priority:** Must Have

**NACH/ECS Bounce Processing:**

| Step | Action | Timeline |
|---|---|---|
| 1 | Receive NACH return file | T+1 or T+2 from presentation date |
| 2 | Parse return file; identify bounced accounts with return reason codes | Automated, within 1 hour of file receipt |
| 3 | Update loan account status; mark EMI as unpaid/bounced | Immediate |
| 4 | Levy bounce charges (as per product configuration and RBI penal charges norms) | Same day |
| 5 | Send bounce notification to customer (SMS + email) | Within 2 hours |
| 6 | Schedule retry (if configured) | As per retry schedule (e.g., T+3, T+7) |
| 7 | If final retry also fails, escalate to collection team | After last retry failure |

**Retry Logic Configuration:**

| Parameter | Description | Example Value |
|---|---|---|
| Max Retries | Maximum number of auto-retries | 2 |
| Retry Interval | Days between retries | 3 days, 7 days |
| Retry Amount | Same amount or revised (e.g., after partial credit to customer account) | Same amount |
| Retry Conditions | Only retry if return code is "Insufficient Funds"; do not retry for "Account Closed" | Configurable per return code |
| Retry Cut-Off | No retry after Nth day of the month (to avoid overlapping with next month's presentation) | 25th of month |

**Bounce Charges:**
- Bounce charge amount: configurable per product (e.g., Rs 500 per bounce for home loans, Rs 300 for personal loans)
- GST on bounce charges: 18% GST applicable
- Bounce charges must be "reasonable and proportionate" per RBI circular on penal charges (effective April 1, 2024)
- Bounce charges disclosed in sanction letter and KFS
- Total bounce charges in a year may be capped (configurable)
- Bounce charges are NOT compounded (no interest charged on bounce charges)

**Acceptance Criteria:**
1. NACH return file processed within 1 hour of receipt
2. Bounce charges levied automatically with GST
3. Customer notification sent within 2 hours
4. Retry scheduled automatically based on configured rules
5. Return code-specific actions executed correctly
6. Bounce MIS available for collection strategy

### 5.12 Payment Receipt Generation

**Requirement ID: LMS-PAY-019**
**Title:** Digital and Physical Payment Receipt Generation
**Priority:** Must Have

**Receipt Contents:**

| Field | Description |
|---|---|
| Receipt Number | Unique system-generated number |
| Loan Account Number | Customer's loan account number |
| Customer Name | Borrower name |
| Payment Date | Date of payment receipt |
| Payment Mode | NACH/UPI/NEFT/Cash/Cheque |
| Payment Reference | UTR/UMRN/Cheque Number/Cash Receipt Number |
| Total Amount Received | Total payment amount |
| Allocation Breakup | Principal, Interest, Penal Charges, Fees -- individual amounts |
| Principal Outstanding After Payment | Updated outstanding balance |
| Next EMI Due Date | Date of next installment |
| Next EMI Amount | Amount of next installment |

**Business Rules:**
- Receipt auto-generated for every successful payment
- Available via: customer portal (download), mobile app, email, SMS link
- Branch can print physical receipt
- Receipt carries digital signature / authentication for electronic receipts
- Receipt number format: [Branch]-[Year]-[Sequence] (configurable)
- Duplicate receipt issuance tracked and limited (anti-fraud)

---
