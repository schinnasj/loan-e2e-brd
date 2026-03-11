# MODULE 3: RECONCILIATION

## 6. RECONCILIATION

### 6.1 Overview

The Reconciliation module ensures accuracy and integrity of all financial data across the LMS, core banking system, payment systems, and General Ledger. Given the volume of loan transactions in an Indian bank or NBFC (often millions of transactions daily), automated reconciliation with intelligent exception handling is critical for operational efficiency, regulatory compliance, and financial reporting accuracy.

### 6.2 Bank Statement Reconciliation with Loan Collections

**Requirement ID: LMS-REC-001**
**Title:** Automated Bank Statement to Loan Collection Reconciliation
**Priority:** Must Have

**Description:** The system shall reconcile incoming funds reflected in the institution's bank statements (from collection bank accounts) with loan payment records in the LMS.

**Reconciliation Flow:**

1. Import bank statement (MT940/CAMT.053 format or CSV/proprietary format) from CBS or bank
2. Import LMS payment records for the same period
3. Execute matching rules
4. Classify entries as: Matched, Partially Matched, Unmatched (Bank Only), Unmatched (LMS Only)
5. Auto-clear matched entries
6. Route exceptions for manual review
7. Generate reconciliation report

**Matching Rules (executed in priority order):**

| Rule Priority | Matching Criteria | Confidence Level |
|---|---|---|
| 1 | Exact match: Amount + UTR/Reference Number + Date | High -- auto-clear |
| 2 | Exact match: Amount + Loan Account Number (from remarks) + Date (plus/minus 1 day) | High -- auto-clear |
| 3 | Amount match + Date match (no reference) | Medium -- manual review |
| 4 | Reference match + Date match (amount mismatch within tolerance, e.g., Rs 10) | Medium -- manual review |
| 5 | Fuzzy match: Amount within tolerance + Date within 3 days | Low -- manual review |
| 6 | No match found | Exception -- suspense |

**Business Rules:**
- Reconciliation frequency: daily (mandatory), with cumulative weekly/monthly views
- Bank statement import: automated via SFTP from CBS or partner banks, minimum twice daily (morning and evening)
- Matching tolerance: configurable per entity (default: Rs 0 for amount, T+/-1 for date)
- Unmatched bank credits: parked in Suspense Account (A11) with automatic aging alerts
- Unmatched LMS entries (payment recorded but not in bank statement): flagged for investigation (potential fraud indicator)
- Suspense aging thresholds: 3 days (alert), 7 days (escalation), 30 days (senior management review), 90 days (write-off or regulatory reporting consideration)
- Multi-collection account support: institution may have multiple collection accounts across banks

**Acceptance Criteria:**
1. Auto-matching rate of at least 95% for standard payment modes (NACH, UPI)
2. Reconciliation completed within 2 hours of bank statement import
3. Exception dashboard accessible to operations team with drill-down capability
4. Suspense aging report generated automatically with escalation triggers
5. Audit trail for all matching decisions (auto and manual)

### 6.3 NACH/ECS Return File Reconciliation

**Requirement ID: LMS-REC-002**
**Title:** NACH Presentation and Return File Reconciliation
**Priority:** Must Have

**Description:** Reconcile NACH presentation files (sent to NPCI/sponsor bank) with response/return files received.

**Reconciliation Process:**

| Step | Source | Target | Reconciliation Check |
|---|---|---|---|
| 1 | NACH Presentation File (LMS generated) | NACH Response File (from NPCI/Bank) | Every presented mandate must have a response (success or return) |
| 2 | Successful debits in NACH response | Bank credit in collection account | Every successful NACH debit must result in a credit to the collection account |
| 3 | NACH return entries | LMS bounce records | Every return entry must be processed in LMS (bounce charges, notifications) |

**Business Rules:**
- NACH response file expected within T+2 of presentation
- If response not received for a presented entry by T+3: escalate to NACH operations team
- Reconciliation must track: total presented, total successful, total returned, total amount collected, total returned amount
- Partial successes (NACH returned for some accounts in a batch) must be reconciled at individual account level
- Monthly NACH reconciliation summary: total mandates active vs total presentations vs total successes -- used for collection efficiency metrics

**Acceptance Criteria:**
1. 100% of presented NACH entries reconciled with response within T+3
2. Zero unreconciled NACH entries after T+5 (escalated and resolved)
3. NACH collection efficiency dashboard available in real-time
4. Reconciliation breaks auto-trigger investigation workflow

### 6.4 Payment Gateway Reconciliation

**Requirement ID: LMS-REC-003**
**Title:** Payment Gateway Transaction Reconciliation
**Priority:** Must Have

**Description:** For payments received via the institution's online portal or mobile app through payment gateways (Razorpay, PayU, CCAvenue, BillDesk, etc.), reconciliation must be performed between the LMS payment records, payment gateway settlement reports, and bank account credits.

**Three-Way Reconciliation:**

| Leg | Source | Data Points |
|---|---|---|
| 1 | LMS Payment Record | Payment ID, Loan Account, Amount, Date, Status |
| 2 | Payment Gateway Settlement Report | Transaction ID, Merchant Reference, Amount, Fee/Commission, Net Settlement, Date |
| 3 | Bank Account Statement | Credit Amount, UTR, Value Date |

**Matching Rules:**
- LMS Payment ID = Gateway Merchant Reference
- Gateway Transaction ID linked to Bank UTR via settlement report
- Amount validation: LMS Amount = Gateway Gross Amount; Bank Credit = Gateway Net Settlement (after gateway commission/MDR deduction)
- Gateway commission/MDR: tracked in a separate GL head (payment processing expense)

**Business Rules:**
- Payment gateway settlement cycle: T+1 or T+2 (configurable per gateway)
- MDR/Commission: tracked and reconciled against agreed rates with gateway provider
- Failed transactions at gateway: must be reconciled to ensure no debit without credit (customer protection)
- Refund reconciliation: if gateway initiated refund, must be matched with LMS refund record
- Gateway-wise reconciliation reports for finance team

**Acceptance Criteria:**
1. Three-way reconciliation automated for all supported payment gateways
2. MDR/commission variance beyond agreed rate flagged
3. Failed/pending transactions aged beyond 24 hours auto-escalated
4. Gateway settlement delay beyond SLA triggers alert

### 6.5 GL vs Sub-Ledger Reconciliation

**Requirement ID: LMS-REC-004**
**Title:** General Ledger to Sub-Ledger Reconciliation
**Priority:** Must Have

**Description:** The system must perform automated reconciliation between LMS sub-ledger balances (sum of all individual loan accounts per GL head) and the corresponding GL account balances in the CBS/ERP system.

**Reconciliation Matrix:**

| GL Head | Sub-Ledger Source | Reconciliation Frequency | Tolerance |
|---|---|---|---|
| Principal Outstanding (A01-A05) | Sum of principal balances of all loan accounts by classification | Daily | Rs 0 (zero tolerance) |
| Interest Receivable (A06) | Sum of accrued interest on performing loan accounts | Daily | Rs 0 |
| Provision Accounts (P01-P10) | Sum of account-level provisions | Daily | Rs 0 |
| Interest Income (I01) | Sum of interest income entries in sub-ledger | Daily | Rs 0 |
| Fee Income (I02-I08) | Sum of fee entries in sub-ledger | Daily | Rs 0 |
| Suspense (A11) | Sum of unallocated entries in suspense | Daily | Rs 0 |
| EMI Collection Clearing (L01) | Temporary holding -- should be zero at end of day | Daily | Rs 0 |

**Business Rules:**
- Zero-tolerance policy for GL-SL reconciliation; any break is a critical exception
- Reconciliation runs as part of EOD processing, after GL posting and before business date roll
- If reconciliation break detected: EOD can be configured to (a) halt and alert, or (b) continue with alert (configurable per GL head; critical heads like principal outstanding should halt)
- Root cause categories for breaks: timing difference, missed posting, duplicate posting, incorrect GL mapping, manual journal entry without SL counterpart
- Correction entries require maker-checker approval
- Monthly certification: operations head must certify zero-break status at month-end

**Acceptance Criteria:**
1. GL-SL reconciliation runs automatically as part of EOD
2. Zero breaks on all critical GL heads (principal, interest income, provisions)
3. Break report generated with root cause classification
4. Historical reconciliation data available for audit (minimum 7 years as per RBI record retention norms)

### 6.6 Suspense Account Clearing and Aging

**Requirement ID: LMS-REC-005**
**Title:** Suspense Account Management and Aging
**Priority:** Must Have

**Description:** Manage all unidentified and unallocated credits/debits parked in suspense accounts with systematic clearing and aging.

**Suspense Account Types:**

| Account | Purpose | Target Clearing SLA |
|---|---|---|
| Collection Suspense | Unidentified loan collections | 3 business days |
| Disbursement Suspense | Disbursement entries pending final booking | 1 business day |
| NACH Suspense | NACH credits received but not matched to loan account | 2 business days |
| Inter-Branch Suspense | Inter-branch transfer entries pending clearing | 5 business days |
| Fee Suspense | Fees collected but not allocated to income heads | 2 business days |
| Insurance Suspense | Insurance premium collected but not remitted | 5 business days |

**Aging Escalation Matrix:**

| Age Bucket | Action | Responsibility |
|---|---|---|
| 0-3 days | Normal clearing; investigation by operations | Operations Officer |
| 4-7 days | Escalation Level 1; daily follow-up | Operations Manager |
| 8-15 days | Escalation Level 2; root cause report required | Branch Manager / Regional Head |
| 16-30 days | Escalation Level 3; senior management review | Head of Operations |
| 31-90 days | Critical escalation; regulatory risk assessment | CFO / Compliance |
| 90+ days | Write-off consideration or regulatory reporting | Board / Audit Committee |

**Business Rules:**
- Daily suspense aging report generated automatically
- Entries in suspense beyond 30 days must have documented reason and action plan
- RBI inspection teams specifically examine aged suspense accounts; system must provide instant reports
- Suspense account balance included in reconciliation dashboards
- Auto-clearing rules: if NACH credit can be matched to a loan account by amount and date (within tolerance), auto-clear with maker-checker
- Bulk clearing functionality for month-end clearing operations

### 6.7 Inter-Branch Reconciliation

**Requirement ID: LMS-REC-006**
**Title:** Inter-Branch Loan Transaction Reconciliation
**Priority:** Must Have

**Description:** For institutions with branch networks, loan payments made at a branch different from the booking branch require inter-branch accounting and reconciliation.

**Business Rules:**
- Customer pays EMI at Branch B; loan is booked at Branch A
- Branch B credits Inter-Branch Account (IBA) and debits cash/collection account
- Branch A debits IBA and credits the loan account
- Reconciliation ensures matching of IBA debits and credits across branches
- CBS typically handles IBA; LMS must reconcile LMS-side entries with CBS IBA
- Target: zero IBA outstanding at end of each business day
- IBA entries not cleared within 3 days escalated for investigation

### 6.8 Nostro/Vostro Reconciliation for Cross-Border Loans

**Requirement ID: LMS-REC-007**
**Title:** Nostro/Vostro Account Reconciliation for FCY Loans
**Priority:** Could Have

**Description:** For foreign currency loans (ECB servicing, FCNR loans), reconciliation of Nostro (our account with foreign bank) and Vostro (their account with us) accounts.

**Business Rules:**
- Disbursement in foreign currency: Nostro account debited; reconciled with correspondent bank statement
- Repayment: if received in INR, converted at applicable rate; Nostro credited for foreign currency obligation
- Exchange rate differences tracked and reconciled
- FEMA compliance: all FCY loan transactions must be FEMA-compliant; system tracks AD Category I bank authorization
- Monthly Nostro reconciliation with correspondent bank statements

### 6.9 Insurance Premium Reconciliation

**Requirement ID: LMS-REC-008**
**Title:** Insurance Premium Collection and Remittance Reconciliation
**Priority:** Must Have

**Description:** Reconcile insurance premiums collected from borrowers (either upfront or as part of EMI) with remittances made to insurance companies.

**Reconciliation Flow:**

| Leg | Source | Check |
|---|---|---|
| 1 | Insurance premium collected from borrower (part of disbursement deduction or EMI) | Recorded in LMS insurance module and Insurance Premium Payable GL (L03) |
| 2 | Premium remitted to insurance company | Payment to insurer reduces Insurance Premium Payable GL |
| 3 | Insurance company confirmation / policy issuance | Policy number linked to loan account; premium status updated |

**Business Rules:**
- Premium collection and remittance must be reconciled monthly
- Unremitted premiums aged beyond 30 days: escalation to operations head
- Insurance company-wise reconciliation for premium paid vs policies issued
- Commission/rebate from insurance company tracked and reconciled
- IRDAI guidelines on group insurance: compliance tracking for premium rates and coverage adequacy
- GST on insurance premium: input tax credit tracking (if applicable)

### 6.10 Escrow Account Reconciliation

**Requirement ID: LMS-REC-009**
**Title:** Escrow Account Reconciliation
**Priority:** Should Have

**Description:** For co-lending arrangements, digital lending partnerships (LSP model), and certain structured finance transactions, escrow accounts are used. The system must reconcile escrow account movements.

**Co-Lending Escrow Reconciliation (per RBI CLM Guidelines):**
- NBFC originates; 80% of loan transferred to Bank partner
- Escrow account receives borrower payments
- Payments split: 80% to Bank, 20% to NBFC (per agreed ratio)
- Reconciliation: total collection = bank share + NBFC share + any charges

**Digital Lending Escrow (per RBI DL Guidelines Sep 2022):**
- LSP collections routed through escrow
- Disbursements routed through escrow to borrower's bank account
- No pass-through of funds; direct credit mandated
- Escrow used only for regulatory compliance documentation

**Business Rules:**
- Daily escrow reconciliation mandatory
- Escrow balance at end of day should match expected holding (collections received but not yet distributed)
- Any excess or deficit in escrow flagged immediately
- Escrow account statements obtained from escrow bank and reconciled

### 6.11 Reconciliation Dashboards and MIS

**Requirement ID: LMS-REC-010**
**Title:** Reconciliation Dashboard and Reporting
**Priority:** Must Have

**Dashboard Metrics:**

| Metric | Description | Frequency |
|---|---|---|
| Auto-Match Rate | % of entries auto-matched without manual intervention | Daily |
| Exception Count | Number of unmatched entries by category | Daily |
| Suspense Balance | Total and aged breakdown of suspense balances | Daily |
| GL-SL Break Count | Number of GL heads with reconciliation breaks | Daily |
| NACH Match Rate | % of NACH presentations reconciled with responses | Daily |
| Escrow Balance | Current escrow balance vs expected | Daily |
| Aged Exceptions | Exceptions by aging bucket with trend | Daily |
| Resolution Time | Average time to resolve reconciliation exceptions | Weekly |
| Collection Efficiency | Total collected vs total due, by payment mode | Daily |

**Report Types:**

| Report | Audience | Frequency |
|---|---|---|
| Daily Reconciliation Summary | Operations Manager | Daily |
| Suspense Aging Report | Operations Head, CFO | Daily |
| GL-SL Reconciliation Certificate | Head of Operations, Audit | Monthly |
| NACH Collection Efficiency | Collections Head | Daily |
| Payment Gateway Settlement Report | Treasury/Finance | Daily |
| Inter-Branch Outstanding Report | Branch Operations | Daily |
| Escrow Account Statement Reconciliation | Partnerships, Compliance | Daily |
| Exception Trend Analysis | CTO, COO | Monthly |

**Business Rules:**
- Dashboard accessible via role-based access; branch users see branch-level data; head office users see consolidated view
- Drill-down capability: from dashboard metric to individual exception entry
- Exception workflow: assign, investigate, resolve, close -- with SLA tracking
- Email/SMS alerts for critical breaks (GL-SL mismatch, high suspense aging, escrow variance)
- Historical reconciliation data retained for minimum 8 years (as per RBI record retention guidelines)
- Reconciliation data available for concurrent and statutory audit teams

**Acceptance Criteria:**
1. Dashboard loads within 5 seconds for current day view
2. All metrics update within 30 minutes of source data availability
3. Drill-down from summary to individual transaction available
4. Export capability for all reports (Excel, PDF, CSV)
5. Role-based access enforced for all reconciliation data

---
