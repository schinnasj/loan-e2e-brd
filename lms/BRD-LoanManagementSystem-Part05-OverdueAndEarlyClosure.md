# MODULE 5: LATE PAYMENT / OVERDUE MANAGEMENT
# MODULE 6: EARLY CLOSURE / PREPAYMENT

---

## 8. LATE PAYMENT / OVERDUE MANAGEMENT

### 8.1 Overview

This module governs the complete lifecycle of overdue loan accounts from initial delinquency through NPA classification, provisioning, collection, legal action, resolution, write-off, and recovery. It must strictly comply with RBI's Master Direction on IRAC norms, SARFAESI Act provisions, RBI's Framework for Resolution of Stressed Assets, and the institution's Board-approved collection and recovery policies.

### 8.2 Days Past Due (DPD) Calculation Engine

**Requirement ID: LMS-OVD-001**
**Title:** Automated DPD Calculation and Tracking
**Priority:** Must Have

**Description:** The system shall calculate DPD for every loan account daily as part of EOD processing. DPD is the foundation for SMA classification, NPA recognition, credit bureau reporting, and provisioning.

**DPD Calculation Logic:**

DPD = Current Business Date - Due Date of Oldest Unpaid Installment (or part thereof)

**Detailed Rules:**

| Scenario | DPD Calculation |
|---|---|
| All EMIs current | DPD = 0 |
| EMI due on 5th of month; not paid by 10th | DPD = 5 |
| Multiple EMIs overdue | DPD counted from due date of oldest unpaid installment |
| Partial payment clearing oldest EMI | DPD resets based on new oldest unpaid installment |
| Partial payment not clearing oldest EMI | DPD continues from original oldest due date |
| Advance EMIs paid | DPD = 0 (advance credit available) |
| Moratorium period | DPD frozen (for regulatory moratorium) |
| Restructured account | DPD resets on restructuring; new DPD per revised schedule |

**Business Rules:**
- DPD calculated as part of EOD before asset classification
- DPD is a whole number (number of calendar days, not business days)
- DPD never goes negative (minimum is 0)
- DPD recalculated whenever a payment is received (intra-day if real-time, or at next EOD)
- DPD history maintained for each account (daily snapshot not required, but classification change dates recorded)
- For SMA and CRILC reporting: DPD is the primary metric

**Acceptance Criteria:**
1. DPD matches manual calculation for sample accounts in UAT
2. Partial payment impact on DPD is correctly computed
3. DPD frozen during regulatory moratorium
4. DPD available for real-time query and reporting

### 8.3 Asset Classification -- SMA and NPA Buckets

**Requirement ID: LMS-OVD-002**
**Title:** Automated Asset Classification Engine
**Priority:** Must Have

**Classification Hierarchy:**

| Classification | DPD Range | Description | Regulatory Significance |
|---|---|---|---|
| Standard (SMA-0) | 1-30 DPD | Overdue but not yet SMA-1 | Internal monitoring; CRILC reporting for Rs 5 Cr+ |
| SMA-1 | 31-60 DPD | Special Mention Account -- 1 | CRILC reporting mandatory (weekly for Rs 5 Cr+) |
| SMA-2 | 61-90 DPD | Special Mention Account -- 2 | CRILC reporting; triggers Early Warning Signal |
| NPA -- Sub-Standard | 91-365 DPD (up to 12 months) | Non-Performing Asset | Provisioning: 15% secured / 25% unsecured (banks) |
| NPA -- Doubtful D1 | 366-730 DPD (12-24 months) | Doubtful -- Category 1 | Provisioning: 25% secured + 100% unsecured |
| NPA -- Doubtful D2 | 731-1460 DPD (24-48 months) | Doubtful -- Category 2 | Provisioning: 40% secured + 100% unsecured |
| NPA -- Doubtful D3 | 1461+ DPD (48+ months) | Doubtful -- Category 3 | Provisioning: 100% |
| Loss | Identified as loss (regardless of DPD) | Loss Asset | Provisioning: 100% |

Note: DPD ranges above are for banks. For NBFCs, NPA was 180 DPD until March 2024; now aligned to 90 DPD per RBI harmonization.

**Special Classification Rules:**

| Product Type | NPA Classification Rule | Reference |
|---|---|---|
| Term Loan | 90+ DPD on any installment | Standard IRAC |
| Overdraft/Cash Credit | Out of order for 90+ days (see definition below) | IRAC Master Direction Section 4.2.4 |
| Agricultural -- Short Duration | Overdue for 2 crop seasons from due date | IRAC Section 4.2.4.1 |
| Agricultural -- Long Duration | Overdue for 1 crop season from due date | IRAC Section 4.2.4.1 |
| Bill Purchased/Discounted | Overdue for 90+ days | Standard IRAC |

**Overdraft/Cash Credit "Out of Order" Definition (RBI IRAC):**
- An OD/CC account is "out of order" if:
  a. Outstanding balance exceeds sanctioned limit/drawing power continuously for 90 days, OR
  b. There are no credits in the account for 90 consecutive days, OR
  c. Credits are not enough to cover the interest debited during the 90-day period

**System Requirements:**
- Classification engine runs daily as part of EOD
- Automatic classification based on DPD rules; no manual intervention for standard cases
- Manual override allowed only for: (a) regulatory dispensation (with circular reference), (b) classification upgrade on satisfying upgrade criteria
- Override requires senior management approval with documented rationale
- Classification change triggers: GL reclassification entries (see LMS-GL-009), provisioning computation, interest income reversal (for NPA), credit bureau update
- System must handle special categories: Wilful Defaulter, Red Flagged Account (RFA), Fraud
- Board-approved NPA management policy configurable in system

**Acceptance Criteria:**
1. Asset classification matches IRAC norms exactly for all loan types
2. Classification changes trigger all downstream actions (GL, provisions, bureau, interest reversal)
3. Override requires documented rationale and senior management approval
4. Classification history maintained with date, old class, new class, trigger (DPD/manual/upgrade)
5. OD/CC "out of order" detection implemented per IRAC definition

### 8.4 NPA Upgrade Rules

**Requirement ID: LMS-OVD-003**
**Title:** NPA to Standard Account Upgrade Processing
**Priority:** Must Have

**Upgrade Criteria (per RBI IRAC norms):**

For a term loan NPA to be upgraded to Standard:
1. All arrears of interest AND principal must be fully paid (not just current EMI)
2. Account must remain regular for a specified period post-payment (varies by product/scheme)
3. For restructured accounts: additional upgrade conditions may apply per restructuring terms

**System Actions on Upgrade:**
- Reclassify from NPA (Sub-Standard/Doubtful/Loss) to Standard
- Reverse excess provisioning (write-back provisions to P&L)
- Reclassify GL entries (move principal from NPA GL heads to Standard GL heads)
- Resume interest accrual in P&L (not retrospectively -- only prospectively)
- Update credit bureau: report as "current" instead of "NPA/overdue"
- Generate upgrade report for management

**Business Rules:**
- Automatic upgrade check runs as part of EOD: if NPA account clears all arrears, evaluate for upgrade
- For accounts that were restructured and classified NPA: upgrade requires satisfactory performance for a defined period (varies per restructuring framework; typically 12 months of regular payments post-restructuring)
- Upgrade does not retroactively recognize interest that was reversed on NPA classification
- Provision write-back (excess provision reversed) limited to the difference between NPA provision rate and standard provision rate
- CRILC and credit bureau updated within the reporting cycle following upgrade

### 8.5 Demand Notice and Reminder Generation

**Requirement ID: LMS-OVD-004**
**Title:** Automated Demand and Reminder Notice Generation
**Priority:** Must Have

**Communication Schedule:**

| DPD | Communication Type | Channel | Content |
|---|---|---|---|
| 1-3 | Payment Reminder | SMS | "Your EMI of Rs [X] was due on [date]. Kindly pay immediately." |
| 4-7 | First Reminder | SMS + Email | Reminder with payment link and helpline number |
| 8-15 | Second Reminder | SMS + Email + IVR Call | Stronger reminder; mention of penal charges |
| 16-30 | Pre-Due Demand Notice | SMS + Email + Letter (physical) + Call | Formal demand notice; mention of consequences of continued default |
| 31-60 (SMA-1) | Formal Demand Notice | Registered Letter + Email + Call | "Recall of facility" notice; NPA warning |
| 61-89 (SMA-2) | Final Demand Before NPA | Registered AD Letter | Final opportunity to regularize before NPA classification |
| 90 (NPA) | NPA Classification Notice | Registered AD Letter + Email | Formal notification of NPA classification; consequences including CIBIL reporting, SARFAESI action |
| 90+ | Statutory Notice (SARFAESI Section 13(2)) | Registered AD Letter | 60-day notice under SARFAESI for enforcement of security interest (if applicable) |
| 150+ | Possession Notice (SARFAESI Section 13(4)) | Physical service + newspaper publication | Notice of intention to take possession of secured asset |

**Business Rules:**
- Communication templates configurable per product and entity
- Language: templates in English + regional language (Hindi, local language as per borrower's preference/state)
- RBI Fair Practices Code compliance: no harassment; communication only between 7 AM and 7 PM
- DND (Do Not Disturb) compliance: respect TRAI DND for promotional communications; transactional messages (demand notices) exempt
- Communication log: every communication recorded with date, channel, content, delivery status
- Co-borrower and guarantor communication: separate configurable rules for when co-borrower/guarantor is communicated
- Digital Lending accounts: additional disclosures per RBI DL guidelines (name of RE, complaint redressal)
- Deceased borrower handling: stop all communication to deceased; redirect to legal heirs/nominees; initiate insurance claim

### 8.6 SARFAESI Act Compliance

**Requirement ID: LMS-OVD-005**
**Title:** SARFAESI Act Workflow Management
**Priority:** Must Have

**Applicability:**
- Banks: all secured loans where NPA outstanding > Rs 1 Lakh
- NBFCs: asset size > Rs 100 Crore and secured NPA outstanding > Rs 20 Lakh
- HFCs: as per NHB guidelines
- NOT applicable to: agricultural land, property < Rs 1 Lakh (banks) or Rs 20 Lakh (NBFCs), pledge of movables

**SARFAESI Workflow Stages:**

| Stage | Action | Timeline | System Requirement |
|---|---|---|---|
| 1 | Demand Notice under Section 13(2) | After NPA classification | Generate notice; record service; start 60-day timer |
| 2 | Borrower Response Period | 60 days from notice | Track response; allow representation; record outcome |
| 3 | Section 13(4) -- Measures | After 60 days if no resolution | Take possession / appoint manager / debt assignment |
| 4 | Possession Notice | 15 days before possession | Generate notice; publication in newspapers |
| 5 | Take Possession | After 15-day notice | Record possession; asset custody management |
| 6 | Sale/Auction Notice | 30 days before sale | Generate auction notice; publication requirements |
| 7 | Asset Sale/Auction | As scheduled | Track sale proceeds; bidder details; allocation |
| 8 | Recovery and Closure | Post-sale | Allocate proceeds; close or continue for deficit |

**System Requirements:**
- Automated timer management for each SARFAESI stage
- Notice generation in prescribed format (model forms per rules)
- Integration with legal case management system
- CERSAI charge enforcement registration
- Auction management: reserve price setting, bidder registration, bid tracking
- Sale proceeds allocation: costs, interest, principal (per SARFAESI Act allocation order)
- DRT (Debt Recovery Tribunal) application tracking if SARFAESI is challenged
- Status dashboard for all SARFAESI cases by stage, branch, product

### 8.7 NPA Provisioning Engine

**Requirement ID: LMS-OVD-006**
**Title:** Automated NPA Provisioning Computation
**Priority:** Must Have

**Description:** System computes provisions automatically based on asset classification, secured/unsecured status, collateral value, and applicable norms (IRAC for regulatory books; ECL for IND AS books).

**Provisioning Computation Logic (IRAC norms for banks):**

For Secured NPA:
- Realizable value of security (collateral) determined by latest valuation
- Unsecured portion = Outstanding - Realizable value of security
- Provision = (Secured portion x Secured provision rate) + (Unsecured portion x 100%)

**Example:**
- Outstanding: Rs 1,00,00,000
- Collateral value (latest valuation): Rs 70,00,000
- Depreciation factor on collateral: 15% (as per bank policy)
- Realizable value: Rs 70,00,000 x 85% = Rs 59,50,000
- Secured portion: Rs 59,50,000
- Unsecured portion: Rs 1,00,00,000 - Rs 59,50,000 = Rs 40,50,000
- Classification: Doubtful D1
- Provision = (Rs 59,50,000 x 25%) + (Rs 40,50,000 x 100%) = Rs 14,87,500 + Rs 40,50,000 = Rs 55,37,500

**Business Rules:**
- Provision recalculated daily (at EOD) for any account with classification change
- Monthly full portfolio provision recomputation
- Collateral revaluation: provision adjusted when collateral is revalued
- Provision capped at 100% of outstanding (cannot exceed outstanding)
- System maintains provision movement schedule: opening + additions - write-backs - write-offs = closing
- Additional provisions: Board/management may require additional provisions beyond regulatory minimum (configurable as "management overlay")
- Counter-cyclical provisioning buffer: if required by RBI, system must support additional buffer computation
- Provision adequacy report: regulatory minimum vs actual provision held

**NBFC Provisioning Norms (differences from banks):**

| Category | NBFC-ML/UL Rate | NBFC-BL Rate |
|---|---|---|
| Standard | 0.40% (general); higher for specific categories | 0.40% |
| Sub-Standard | 10% | 10% |
| Doubtful D1 | 20% | 20% |
| Doubtful D2 | 30% | 30% |
| Doubtful D3 | 50% | 50% |
| Loss | 100% | 100% |

Note: NBFC provisioning norms differ from bank norms. System must be configurable per entity type.

### 8.8 Technical Write-Off vs Actual Write-Off

**Requirement ID: LMS-OVD-007**
**Title:** Write-Off Processing and Post-Write-Off Tracking
**Priority:** Must Have

**Technical Write-Off:**
- Loan removed from active books (balance sheet)
- 100% provision utilized against the outstanding
- Recovery efforts continue
- Account tracked in off-balance sheet memorandum
- Reported to credit bureau as "Written Off"
- Authority: Board-approved write-off policy; typically delegated to MD/CEO up to a threshold

**Actual Write-Off:**
- All recovery efforts exhausted; loss crystallized
- Tax benefit claimed under Section 36(1)(vii) of IT Act (requires actual write-off in books)
- Regulatory reporting: part of "write-off" figure in NPA disclosures

**Write-Off Authority Matrix (example):**

| Outstanding Amount | Approval Authority |
|---|---|
| Up to Rs 10 Lakh | General Manager / Regional Head |
| Rs 10 Lakh to Rs 1 Crore | Executive Director / Committee |
| Rs 1 Crore to Rs 10 Crore | MD & CEO |
| Above Rs 10 Crore | Board of Directors |

**Business Rules:**
- Write-off only after 100% provision is held
- Write-off proposal workflow: branch recommendation -> regional review -> head office approval -> authority approval
- Post-write-off: account continues in system for recovery tracking; separate collections module
- Recovery from written-off accounts: credited to P&L as "Recovery from Written-Off Accounts" (I09)
- Annual review of written-off portfolio for recovery potential
- Write-off portfolio MIS: year-wise write-offs, cumulative recovery, recovery rate

### 8.9 One-Time Settlement (OTS) Processing

**Requirement ID: LMS-OVD-008**
**Title:** One-Time Settlement / Compromise Settlement Processing
**Priority:** Must Have

**Description:** For NPA accounts where full recovery is unlikely, the institution may offer/accept a one-time settlement (OTS) or compromise settlement.

**OTS Workflow:**

1. **OTS Proposal**: Branch/collections team prepares OTS proposal with rationale, amount offered, and recovery analysis
2. **OTS Evaluation**: Compare OTS amount with: (a) total outstanding, (b) realizable collateral value, (c) litigation cost and timeline, (d) borrower's paying capacity
3. **OTS Approval**: Authority based on sacrifice amount (outstanding - OTS amount)
4. **OTS Sanction**: Issue OTS sanction letter with terms (payment timeline, typically 30-90 days)
5. **OTS Payment Receipt**: Track OTS payments (may be in installments within sanctioned timeline)
6. **OTS Closure**: Once full OTS amount received, close account; release collateral; update bureau; issue NOC
7. **OTS Lapse**: If OTS payment not received within timeline, OTS lapses; resume normal collection

**OTS Authority Matrix (example based on sacrifice amount):**

| Sacrifice Amount | Approval Authority |
|---|---|
| Up to Rs 5 Lakh | Zonal Manager |
| Rs 5 Lakh to Rs 25 Lakh | General Manager |
| Rs 25 Lakh to Rs 1 Crore | ED / Committee |
| Rs 1 Crore to Rs 5 Crore | MD & CEO |
| Above Rs 5 Crore | Board |

**Business Rules:**
- OTS sacrifice = Total Outstanding (Principal + Interest + Charges) - OTS Settlement Amount
- Sacrifice reported as loss in P&L (if not already provisioned); or provision write-back if sacrifice < provision held
- OTS accounts reported to CIBIL as "Settled" (distinct from "Closed" -- affects borrower's credit score)
- Minimum OTS amount: typically >= realizable value of security
- OTS for wilful defaulters: stricter policy; Board approval mandatory regardless of amount
- Cooling period: no fresh credit facility for minimum 12 months post-OTS (configurable per policy)
- Tax implications: OTS waiver amount may be taxable income for borrower (Section 41(1) if previously claimed as deduction); institution to issue certificate

### 8.10 Wilful Defaulter Identification and Reporting

**Requirement ID: LMS-OVD-009**
**Title:** Wilful Defaulter Classification and Reporting
**Priority:** Must Have

**RBI Reference:** Master Circular on Wilful Defaulters (DOR.CRE.REC.No.43/21.04.048/2024-25)

**Wilful Default Definition (RBI):**
1. Default in repayment despite having capacity to pay
2. Diversion of funds from stated purpose
3. Siphoning of funds
4. Disposal/sale of secured assets without bank's knowledge

**System Requirements:**
- Wilful defaulter identification workflow: proposal by branch -> examination committee -> identification committee (chaired by ED/MD)
- Show-cause notice to borrower with opportunity to respond
- Committee decision recorded with detailed reasoning
- Classification as "Wilful Defaulter" or "Non-Cooperative Borrower"
- Reporting to RBI (CRILC) and credit bureaus
- Bar on additional credit facilities (system must prevent new loan sanctioning for identified wilful defaulters)
- Connected parties/group entities tracking (wilful defaulter status impacts all group entities' credit access)
- Suit-Filed Accounts reporting (SFA) integrated with wilful defaulter tracking

**Acceptance Criteria:**
1. Wilful defaulter identification workflow configurable with all stages
2. System prevents new loan sanction to identified wilful defaulters and connected entities
3. CRILC and bureau reporting includes wilful defaulter classification
4. Audit trail of complete identification process maintained

### 8.11 Collection Strategy Engine

**Requirement ID: LMS-OVD-010**
**Title:** Rule-Based Collection Strategy and Allocation
**Priority:** Must Have

**Description:** The system shall implement a configurable collection strategy engine that determines the appropriate collection action based on account attributes.

**Strategy Segmentation Parameters:**

| Parameter | Examples |
|---|---|
| DPD Bucket | 1-30, 31-60, 61-90, 91-180, 181-365, 365+ |
| Product Type | Home Loan, Personal Loan, Vehicle Loan, etc. |
| Outstanding Amount | Buckets: < 1L, 1-5L, 5-25L, 25L-1Cr, > 1Cr |
| Geographic Zone | Metro, Urban, Semi-Urban, Rural |
| Customer Segment | Salaried, Self-Employed, Business, Agriculture |
| Previous Payment Behavior | First-time defaulter, Repeat defaulter, Roller |
| Collateral Type | Secured (property/gold/vehicle) vs Unsecured |
| Legal Status | Pre-legal, Section 13(2) issued, DRT filed |

**Collection Actions:**

| Action | Description |
|---|---|
| Soft Calling | Outbound call from call center; polite reminder |
| Field Visit | Physical visit by field agent to borrower's residence/office |
| Legal Notice | Formal legal notice through advocate |
| SARFAESI | SARFAESI proceedings (for eligible secured accounts) |
| DRT Filing | Debt Recovery Tribunal application |
| Skip Tracing | Locate borrower if contact details are outdated |
| Third-Party Collection Agency | Assign to outsourced collection agency (per RBI guidelines on outsourcing) |
| Write-Off Recommendation | For accounts with no recovery prospect |

**Business Rules:**
- Collection agency assignment: system tracks agency-wise allocation, recovery performance, and commission
- RBI guidelines on outsourcing of collection: agent must identify themselves; no harassment; no threatening language; calling hours 7 AM-7 PM only; no public humiliation
- Collection call recording: if integrated, all call recordings linked to loan account
- Field visit geo-tagging: if mobile app for field agents, GPS tracking of visits
- Promise-to-Pay (PTP) tracking: agent records PTP with amount and date; system tracks fulfillment
- Collection escalation matrix: if PTP broken twice, escalate to next action level
- Dashboard: collection team workload, recovery rates by bucket/agent/agency, PTP fulfillment rate, roll rates

---

## 9. EARLY CLOSURE / PREPAYMENT

### 9.1 Overview

The Early Closure / Prepayment module handles foreclosure (full early closure), part-prepayment, and all associated processes including penalty computation, interest refund, security release, NOC generation, and regulatory compliance. RBI guidelines on prepayment penalties and transparency are critical here.

### 9.2 Foreclosure (Full Prepayment) Processing

**Requirement ID: LMS-ECL-001**
**Title:** Loan Foreclosure Workflow
**Priority:** Must Have

**Description:** Complete loan closure before scheduled maturity upon receipt of full outstanding amount.

**Foreclosure Calculation:**

Foreclosure Amount = Outstanding Principal + Accrued Interest (up to foreclosure date) + Penal Charges (if any) + Foreclosure Penalty (if applicable) + Other Dues (insurance, fees) - Advance EMI Credit (if any)

**Example:**
- Outstanding Principal: Rs 35,42,000
- Accrued Interest (15 days of current month): Rs 12,376
- Penal Charges Outstanding: Rs 0
- Foreclosure Penalty: Rs 0 (floating rate home loan -- no penalty per RBI)
- Insurance Premium Adjustment: Rs 0
- Advance Credit Balance: Rs 5,000

Foreclosure Amount = Rs 35,42,000 + Rs 12,376 + Rs 0 + Rs 0 - Rs 5,000 = Rs 35,49,376

**Foreclosure Statement Contents:**

| Item | Amount |
|---|---|
| Principal Outstanding | Rs 35,42,000 |
| Interest Accrued but Not Due | Rs 12,376 |
| Overdue Interest (if any) | Rs 0 |
| Penal Charges (if any) | Rs 0 |
| Foreclosure Penalty (if applicable) | Rs 0 |
| Other Charges (documentation, CERSAI closure, etc.) | Rs 500 |
| Less: Credit Balance / Advance EMI | (Rs 5,000) |
| **Total Foreclosure Amount** | **Rs 35,49,876** |
| Statement Valid Until | [Date + 7 days] |

**Business Rules:**
- Foreclosure statement valid for a limited period (configurable: 7-15 days) due to daily interest accrual
- Statement can be generated by customer (self-service portal) or branch
- Foreclosure fee/penalty: see LMS-ECL-002
- Upon receipt of foreclosure amount: immediate closure processing
- GL entries: reverse outstanding from Principal Outstanding GL; credit interest income for accrued amount; close all sub-ledger entries
- Post-closure: initiate security release, NOC generation, CERSAI satisfaction, bureau update, insurance refund

### 9.3 Foreclosure Penalty

**Requirement ID: LMS-ECL-002**
**Title:** Foreclosure / Prepayment Penalty Calculation
**Priority:** Must Have

**RBI Guidelines on Prepayment Penalty:**

| Loan Type | Borrower Type | RBI Rule | Penalty Allowed? |
|---|---|---|---|
| Home Loan (floating rate) | Individual | No prepayment penalty allowed (RBI circular DNBS.CC.PD.No.266/03.10.01/2012-13 for NBFCs; similar for banks) | No |
| Home Loan (fixed rate) | Individual | Penalty may be charged; must be disclosed upfront | Yes (with disclosure) |
| Personal Loan (floating rate) | Individual | No penalty (as per Fair Practices Code and RBI guidance) | Typically No |
| Vehicle Loan (floating rate) | Individual | No penalty | No |
| LAP (floating rate) | Individual | No penalty | No |
| Business/MSME Loan | Business | Penalty may be charged as per agreement | Yes (with disclosure) |
| Fixed Rate Products | Any | Penalty allowed; typically 2-5% of outstanding | Yes |

**Penalty Calculation Methods (configurable per product):**

| Method | Formula | Example (Outstanding Rs 30L) |
|---|---|---|
| Percentage of Outstanding | Outstanding x Penalty% | Rs 30,00,000 x 2% = Rs 60,000 |
| Percentage of Prepaid Amount | Prepaid Amount x Penalty% | Used for part-prepayment |
| Flat Amount | Fixed amount per product | Rs 5,000 |
| EMI-Based | N months' EMI | 3 x Rs 26,965 = Rs 80,895 |
| Reducing Penalty | Higher penalty in early years, lower later | Year 1-3: 4%; Year 4-7: 2%; Year 8+: Nil |
| Lock-In Based | Penalty only during lock-in period | Lock-in 12 months; penalty 3% if closed in 12 months |

**Business Rules:**
- Penalty applied only if not prohibited by RBI norms for the product/rate type
- Penalty waiver: allowed with appropriate authority (maker-checker)
- GST on foreclosure penalty: 18% GST applicable (classified as a service charge)
- Penalty disclosed in: sanction letter, loan agreement, KFS
- Lock-in period: for some products, foreclosure not allowed during lock-in (e.g., first 6 months for personal loans); system must enforce

### 9.4 Part-Prepayment Processing

**Requirement ID: LMS-ECL-003**
**Title:** Part-Prepayment Handling with Tenure/EMI Adjustment
**Priority:** Must Have

**Description:** When a borrower makes a lump-sum payment exceeding the current EMI amount with the intent to reduce the loan burden.

**Part-Prepayment Options:**

| Option | Impact | Borrower Benefit |
|---|---|---|
| Reduce Tenure (EMI unchanged) | Fewer remaining installments | Lower total interest cost |
| Reduce EMI (Tenure unchanged) | Lower monthly installment | Improved monthly cash flow |
| Borrower's Choice at Transaction Time | System prompts borrower to choose | Flexibility |

**Example -- Part-Prepayment of Rs 5,00,000:**

Original Loan: Rs 50,00,000 | Rate: 8.50% | Tenure: 240 months | EMI: Rs 43,391
After 24 months: Outstanding = Rs 48,14,000 | Remaining Tenure: 216 months

After Part-Prepayment of Rs 5,00,000: New Outstanding = Rs 43,14,000

**Option A -- Reduce Tenure (EMI = Rs 43,391):**
New Tenure = 188 months (reduced by 28 months)
Interest Saved = approximately Rs 12,14,948

**Option B -- Reduce EMI (Tenure = 216 months):**
New EMI = Rs 38,857 (reduced by Rs 4,534/month)

**Business Rules:**
- Minimum part-prepayment amount: configurable per product (e.g., minimum Rs 10,000 or 1 EMI equivalent)
- Maximum part-prepayments per year: configurable (some products limit to 2-4 per year for administrative reasons)
- Part-prepayment penalty: same rules as foreclosure penalty (see LMS-ECL-002)
- Part-prepayment applied to principal only (after clearing any overdue amounts per waterfall)
- Amortization schedule regenerated after part-prepayment
- Customer receives confirmation with: amount adjusted, new EMI or new tenure, new amortization schedule
- Interest saving statement generated on request (shows interest saved due to prepayment)
- For fixed rate loans: may trigger penalty computation
- For NPA accounts: part-prepayment helps reduce DPD if it clears overdue installments

### 9.5 Security / Collateral Release Workflow

**Requirement ID: LMS-ECL-004**
**Title:** Post-Closure Security Release Process
**Priority:** Must Have

**Collateral Release Workflow:**

| Step | Action | Responsibility | SLA |
|---|---|---|---|
| 1 | Loan closure confirmed; all dues cleared | System / Operations | Day 0 |
| 2 | Initiate collateral release request | Operations / Customer | Day 0-1 |
| 3 | Verify no other loans against same collateral | System check | Day 1 |
| 4 | CERSAI charge satisfaction filing | Operations | Within 30 days (regulatory) |
| 5 | Retrieve original property documents from vault/custodian | Document management | Day 3-5 |
| 6 | Prepare release letter / NOC | Operations | Day 2-3 |
| 7 | Obtain authorized signatory approval | Maker-Checker | Day 3-5 |
| 8 | Hand over documents to customer with acknowledgment | Branch / Courier | Day 5-15 |
| 9 | Remove lien from vehicle RC book (vehicle loans) | RTO process | Day 15-30 |
| 10 | Return original shares/FD receipts (Loan against securities/FD) | Operations | Day 3-5 |

**Business Rules:**
- No documents released until all dues (including penal charges, insurance, CERSAI fees) are cleared
- Multiple loans against same collateral: collateral released only when ALL loans are closed
- Document return acknowledgment: signed by borrower or authorized representative
- Lost/damaged document: documented with senior management approval; indemnity from customer
- CERSAI satisfaction filing: mandatory within 30 days of loan closure (penalty for non-compliance)
- For gold loans: gold ornaments returned on the same day as closure (typically)
- For vehicle loans: lien removal from RC book; Form 35 (lien removal) generated by system
- Document return register: physical register and system log maintained

### 9.6 NOC and Lien Removal

**Requirement ID: LMS-ECL-005**
**Title:** No Objection Certificate (NOC) and Lien Removal Processing
**Priority:** Must Have

**NOC Contents:**

| Field | Description |
|---|---|
| NOC Reference Number | Unique system-generated number |
| Date of Issue | NOC issuance date |
| Borrower Details | Name, address, PAN, loan account number |
| Loan Details | Product, disbursement date, original amount, closure date, final amount paid |
| Statement | "The above loan account has been fully closed and we have no objection/claim against the property/collateral described herein." |
| Collateral Details | Property address / vehicle details / FD number / gold description |
| CERSAI Reference | CERSAI charge registration number and satisfaction filing reference |
| Authorized Signatory | Name, designation, signature, seal |

**Business Rules:**
- NOC generated automatically upon loan closure confirmation
- Available for download on customer portal
- Physical copy dispatched within 15 days of closure (as per Fair Practices Code; some institutions commit 7 days)
- NOC cannot be generated if there are any pending dues or charges
- Duplicate NOC: tracked and limited; requires maker-checker

### 9.7 Account Closure Checklist

**Requirement ID: LMS-ECL-006**
**Title:** Loan Account Closure Verification Checklist
**Priority:** Must Have

**System-Enforced Closure Checklist:**

| Checklist Item | Validation Type | Mandatory |
|---|---|---|
| All principal outstanding cleared | System calculated; zero balance | Yes |
| All interest (accrued and overdue) cleared | System calculated | Yes |
| All penal charges cleared | System calculated | Yes |
| All fees and charges cleared | System calculated | Yes |
| Insurance premium reconciled | System check | Yes |
| NACH mandate cancelled | System action | Yes |
| PDCs returned (if applicable) | Manual confirmation | Yes |
| CERSAI satisfaction filed | System/manual | Yes |
| Credit bureau updated with closure status | System action | Yes |
| Security documents identified for release | Manual confirmation | Yes |
| NOC generated | System action | Yes |
| GL entries posted (closure entries) | System action | Yes |
| Excess amount refunded (if any) | System check | If applicable |
| Tax certificate generated (provisional/final) | System action | Yes |
| Co-lender notification sent (for CLM accounts) | System action | If applicable |

**Business Rules:**
- All mandatory checklist items must be completed/confirmed before account status changes to "Closed"
- Checklist completion tracked with user ID and timestamp
- Partially completed checklists remain in "Closure In Progress" status
- Aging report for accounts in "Closure In Progress" beyond SLA (e.g., 7 days)

### 9.8 Tax Certificate Issuance

**Requirement ID: LMS-ECL-007**
**Title:** Interest and Principal Tax Certificate Generation
**Priority:** Must Have

**Home Loan Tax Certificates:**

| Certificate | For Section | Content | Timing |
|---|---|---|---|
| Interest Certificate (Section 24(b)) | Interest deduction on housing loan | Total interest paid during the financial year | Annual (by June 15) |
| Principal Repayment Certificate (Section 80C) | Principal deduction | Total principal repaid during the financial year | Annual (by June 15) |
| Provisional Certificate | Both | Projected interest and principal for current FY based on amortization | On request / beginning of FY |
| Pre-Possession Interest Certificate | Section 24(b) | Total pre-EMI interest paid before possession (eligible for deduction in 5 equal installments post-possession) | On possession / request |
| Closure Certificate | Both | Final year's interest and principal up to closure date | On closure |

**Business Rules:**
- Annual certificates generated automatically at FY end (March 31) for all active home loan accounts
- Certificates available on customer portal for download
- Certificate data must match actual transactions (interest and principal paid, not accrued)
- For co-borrowers: certificate split based on (a) equal split or (b) as per ownership ratio, as requested by borrowers (configurable)
- PAN of borrower mandatory on certificate
- Education loan interest certificate: for Section 80E deduction (interest only, no limit, for 8 years from start of repayment)
- Certificate data reconciled with GL postings before issuance

### 9.9 Insurance Premium Refund Processing

**Requirement ID: LMS-ECL-008**
**Title:** Insurance Premium Refund on Early Closure
**Priority:** Must Have

**Business Rules:**
- Single premium credit life insurance: refund of unexpired premium on early closure (pro-rata or per insurance company's refund table)
- Annual premium insurance: no refund for current year; future premiums not charged
- Property insurance: refund depends on insurance policy terms
- Refund amount calculated by insurance integration module; confirmed by insurer
- Refund credited to customer's bank account (not adjusted against loan as loan is already closed)
- Refund timeline: within 15 days of closure (as per IRDAI guidelines on refunds)
- If insurance claim was made during loan tenure: refund may not apply; system must check claim history

---
