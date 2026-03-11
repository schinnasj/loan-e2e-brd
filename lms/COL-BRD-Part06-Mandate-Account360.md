# PART 06 -- MANDATE & ACCOUNT 360 BOUNDED CONTEXT (BC-06)

---

## 12. MANDATE & ACCOUNT CONTEXT

### 12.1 Context Overview

| Attribute | Details |
|---|---|
| Context Name | Mandate & Account Context |
| Context ID | BC-06 |
| Domain Classification | Supporting Domain |
| Owner Team | Loan Operations |
| Upstream Dependencies | External LMS/CBS (loan data, EMI schedule, customer data); Reconciliation (BC-01) — PaymentCredited, BounceConfirmed |
| Downstream Consumers | Workflow (BC-03), Communication (BC-04), Sentiment (BC-05), Agency (BC-07), Legal (BC-08), Payment (BC-09) |
| Primary Objective | Provide the single source of truth for mandate status, customer 360° view, EMI lifecycle, and interaction history for the collections domain |

### 12.2 Ubiquitous Language -- Mandate & Account-Specific Terms

| Term | Definition |
|---|---|
| **Mandate** | An authorization by the borrower for automatic debit of EMI from their bank account. Types: NACH, ECS, UPI AutoPay, Standing Instruction (SI). |
| **UMRN** | Unique Mandate Reference Number assigned by NPCI for NACH mandates. 23-character alphanumeric. |
| **Sponsor Bank** | The bank on whose behalf NACH mandates are presented (lender's bank). |
| **Destination Bank** | The borrower's bank where the account is debited. |
| **Mandate Status** | Lifecycle states: INITIATED, REGISTERED, ACTIVE, SUSPENDED, CANCELLED, EXPIRED, REJECTED. |
| **Customer 360** | A consolidated view of all information about a borrower relevant to collections: demographics, loans, EMIs, payments, interactions, mandates, sentiment, legal status. |
| **EMI Lifecycle** | States of an individual EMI installment: SCHEDULED, DUE, OVERDUE, PARTIALLY_PAID, PAID, WAIVED, RESTRUCTURED. |
| **Interaction** | Any contact between the institution and the customer: outbound call, inbound call, SMS, WhatsApp message, email, field visit, legal notice, AI conversation. |
| **SOA (Statement of Account)** | Detailed account statement showing all transactions, interest accrual, charges, payments, and outstanding balance. |

### 12.3 Aggregates

#### 12.3.1 Aggregate: Mandate

| Aspect | Details |
|---|---|
| Aggregate Root | `Mandate` |
| Identity | `MandateId` (UUID), also carries `UMRN` for NACH mandates |
| Description | Represents a payment mandate (auto-debit authorization) linked to a loan account. |

**Entities:**
- `Mandate` — Root. Contains mandate type, UMRN, sponsor bank, destination bank details, amount, frequency, validity, status.
- `MandatePresentationHistory` — Record of each time the mandate was presented (success/bounce).
- `MandateAmendmentHistory` — Changes to mandate (amount increase/decrease, validity extension).

**Value Objects:**
- `MandateType` — Enum: NACH, ECS, UPI_AUTOPAY, STANDING_INSTRUCTION
- `MandateStatus` — Enum: INITIATED, REGISTRATION_PENDING, REGISTERED, ACTIVE, SUSPENDED, CANCELLED, EXPIRED, REJECTED
- `BankAccountDetails` — (accountNumber [masked], IFSC, bankName, branchName, accountHolderName)
- `MandateFrequency` — Enum: MONTHLY, QUARTERLY, HALF_YEARLY, YEARLY, AS_PRESENTED
- `MandateAmount` — (fixedAmount OR maxAmount, isFixedAmount: boolean)
- `MandateValidity` — (startDate, endDate)
- `PresentationRecord` — (presentationDate, amount, status [SUCCESS/RETURNED], returnCode, returnReason, settlementDate)

**Invariants:**
- A loan account can have at most one active mandate of each type at a time
- Mandate amount must be ≥ EMI amount (for fixed amount mandates)
- Mandate validity end date must be ≥ loan maturity date
- Cancelled/expired mandates cannot be presented
- Presentation can only occur on business days

#### 12.3.2 Aggregate: LoanAccountSnapshot

| Aspect | Details |
|---|---|
| Aggregate Root | `LoanAccountSnapshot` |
| Identity | `LoanAccountId` |
| Description | Collections-domain snapshot of loan account data. Sourced from LMS/CBS via ACL. Enriched with collections-specific data. |

**Entities:**
- `LoanAccountSnapshot` — Root. Contains loan details, outstanding amounts, DPD, bucket, assigned strategy.
- `EMIScheduleEntry` — Individual EMI with status (scheduled/due/overdue/paid/partial).
- `PaymentRecord` — Payments received against this account with allocation details.
- `ChargeRecord` — Charges applied (bounce charges, penal interest, legal costs).

**Value Objects:**
- `LoanDetails` — (productType, sanctionedAmount, disbursedAmount, tenureMonths, interestRate, rateType [FIXED/FLOATING], disbursementDate, maturityDate, emiAmount, emiDay)
- `OutstandingBreakup` — (principalOutstanding, interestOutstanding, penalInterest, charges, totalOutstanding)
- `EMIStatus` — Enum: SCHEDULED, DUE, OVERDUE, PARTIALLY_PAID, PAID, WAIVED, RESTRUCTURED
- `AccountStatus` — Enum: ACTIVE, NPA, RESTRUCTURED, SETTLED, CLOSED, WRITTEN_OFF

#### 12.3.3 Aggregate: CustomerProfile

| Aspect | Details |
|---|---|
| Aggregate Root | `CustomerProfile` |
| Identity | `CustomerId` |
| Description | Customer information relevant to collections. Enriched with interaction history and preferences. |

**Entities:**
- `CustomerProfile` — Root. Demographics, contact details, KYC status, employment details.
- `ContactAttemptLog` — Every attempt to contact this customer (success/failure, channel, disposition).
- `InteractionRecord` — Summary of each interaction (linked to detailed records in Communication Context).
- `AddressRecord` — Multiple addresses (residence, office, permanent) with verification status.

**Value Objects:**
- `ContactInfo` — (primaryMobile, secondaryMobile, email, whatsappNumber, preferredLanguage, preferredChannel, dndStatus)
- `EmploymentInfo` — (employmentType [SALARIED/SELF_EMPLOYED/BUSINESS/PROFESSIONAL/AGRICULTURE], employerName, designation, monthlySalary, salaryAccountBank)
- `KYCStatus` — (panVerified, aadhaarVerified, addressVerified, photoVerified, lastKYCDate)
- `RelationshipSummary` — (totalLoans, totalExposure, totalOverdue, longestRelationshipYears, previousNPACount)

### 12.4 Domain Events

| Event ID | Event Name | Published When | Payload | Consumed By |
|---|---|---|---|---|
| DE-MA-001 | `MandateRegistered` | New mandate successfully registered with NPCI/bank | mandateId, umrn, loanAccountId, mandateType, amount | Reconciliation (BC-01), Workflow (BC-03) |
| DE-MA-002 | `MandateCancelled` | Mandate cancelled (by customer, bank, or system) | mandateId, umrn, loanAccountId, cancellationReason | Workflow (BC-03), EWS in Sentiment (BC-05) |
| DE-MA-003 | `MandateAmended` | Mandate amount or validity changed | mandateId, umrn, previousAmount, newAmount, amendmentReason | Reconciliation (BC-01) |
| DE-MA-004 | `MandateBounced` | Mandate presentation failed | mandateId, loanAccountId, returnCode, bounceCount | Delinquency (BC-02), Workflow (BC-03) |
| DE-MA-005 | `EMIStatusChanged` | Individual EMI status changed (e.g., overdue → paid) | loanAccountId, emiNumber, previousStatus, newStatus, paidAmount | Delinquency (BC-02) |
| DE-MA-006 | `CustomerContactUpdated` | Customer's contact information updated | customerId, updatedFields, source | Communication (BC-04) |
| DE-MA-007 | `CustomerAddressVerified` | Address verified via field visit or digital verification | customerId, addressType, verificationMethod, result | Agency (BC-07) |
| DE-MA-008 | `AccountRestructured` | Loan account restructured with new terms | loanAccountId, previousTerms, newTerms, restructuringType | Delinquency (BC-02), Workflow (BC-03) |

### 12.5 Functional Requirements

#### 12.5.1 Mandate Management

| Req ID | Requirement | Priority | Acceptance Criteria |
|---|---|---|---|
| COL-MA-001 | System shall display all mandates (active, inactive, cancelled, expired) for a loan account with complete details (UMRN, type, amount, frequency, bank details, validity, status). | P0 | All mandates listed; status accurate as of real-time |
| COL-MA-002 | System shall show mandate presentation history: every presentation with date, amount, outcome (success/failure), return code, and settlement date. | P0 | Complete history with search/filter by date range |
| COL-MA-003 | System shall show mandate success rate (successful presentations / total presentations) per mandate and per loan account. | P0 | Success rate calculated; displayed as percentage |
| COL-MA-004 | System shall track mandate health score based on: success rate, bounce pattern, last successful date, consecutive bounces. | P1 | Health score: GREEN (>90% success), AMBER (70-90%), RED (<70%) |
| COL-MA-005 | System shall support mandate refresh workflow: initiate new mandate registration when existing mandate is unhealthy or cancelled. | P0 | Refresh workflow with customer notification, digital mandate registration, status tracking |
| COL-MA-006 | System shall support UPI AutoPay mandate management including: registration via intent/collect flow, mandate status tracking, revocation handling. | P0 | UPI AutoPay mandates tracked with same detail as NACH |
| COL-MA-007 | System shall alert when mandate amount is less than EMI amount (e.g., after rate reset causing EMI increase). | P0 | Alert raised within 1 day of EMI change; mandate amendment initiated |
| COL-MA-008 | System shall support mandate amount amendment (increase) workflow with customer consent (digital consent via OTP). | P1 | Amendment submitted to NPCI/bank; tracked to completion |

#### 12.5.2 Customer 360° View

| Req ID | Requirement | Priority | Acceptance Criteria |
|---|---|---|---|
| COL-MA-009 | System shall provide a unified Customer 360° view accessible to all collection roles with role-based data access. | P0 | Single screen with all relevant customer data; loads in <3 seconds |
| COL-MA-010 | Customer 360 shall display: demographics (name, age, gender, PAN, Aadhaar status), all contact numbers (with last successful contact channel/date), all addresses (with verification status). | P0 | Demographic data accurate; sourced from LMS/KYC system |
| COL-MA-011 | Customer 360 shall display: all loan accounts with current status, outstanding, DPD, bucket, EMI amount, next due date, overdue amount breakup. | P0 | All accounts visible; data refreshed within 5 minutes of change |
| COL-MA-012 | Customer 360 shall display: EMI schedule for selected account with paid/unpaid/partial status, payment date, payment amount, payment source for each EMI. | P0 | Complete EMI schedule with color-coded status |
| COL-MA-013 | Customer 360 shall display: complete payment history with date, amount, source/rail, allocation breakup (principal/interest/charges). | P0 | Payment history searchable by date range |
| COL-MA-014 | Customer 360 shall display: all active/historical mandates with health status. | P0 | Mandate data as specified in COL-MA-001 through COL-MA-004 |
| COL-MA-015 | Customer 360 shall display: complete interaction history across all channels (calls with recording playback, SMS, WhatsApp with message thread, email, field visit reports, legal notices) in unified timeline. | P0 | Unified timeline; call recording playback inline; message threads viewable |
| COL-MA-016 | Customer 360 shall display: sentiment score with trend chart, risk category, propensity scores. | P0 | Intelligence data from BC-05 integrated |
| COL-MA-017 | Customer 360 shall display: PTP history with adherence rate (kept/broken/partially kept). | P0 | PTP data from Delinquency Context |
| COL-MA-018 | Customer 360 shall display: collection agency assignment (current and historical) with agency name, assignment date, performance notes. | P0 | Agency data from BC-07 |
| COL-MA-019 | Customer 360 shall display: dispute/complaint history with status and resolution. | P0 | Dispute data linked; resolution visible |
| COL-MA-020 | Customer 360 shall display: legal case status (if any) — notice dates, case type, court, next hearing, current status. | P0 | Legal data from BC-08 |
| COL-MA-021 | Customer 360 shall display: co-borrower/guarantor information with their contact details and collection status. | P0 | Co-applicant data linked |
| COL-MA-022 | Customer 360 shall display: collateral/security details (for secured loans) — type, value, last valuation date, insurance status. | P1 | Collateral data from LMS |
| COL-MA-023 | Customer 360 shall support quick actions: initiate call, send WhatsApp, send SMS, generate SOA, generate demand notice, capture PTP, raise dispute, create field visit task. | P0 | One-click actions from 360 view |

#### 12.5.3 EMI & Statement Management

| Req ID | Requirement | Priority | Acceptance Criteria |
|---|---|---|---|
| COL-MA-024 | System shall maintain real-time EMI schedule with status updates reflecting payments, bounces, waivers, and restructuring. | P0 | EMI status updated within 5 minutes of payment/bounce event |
| COL-MA-025 | System shall generate Statement of Account (SOA) on-demand and scheduled (monthly) with detailed transaction-level breakup. | P0 | SOA in PDF format; includes every debit, credit, charge, interest posting |
| COL-MA-026 | SOA shall include: opening balance, each EMI due (with principal/interest split), each payment received (with allocation), each charge applied, interest accrual entries, closing balance. | P0 | SOA balances to the paisa |
| COL-MA-027 | System shall generate overdue breakup statement showing: overdue principal, overdue interest, penal interest, bounce charges, legal costs — per EMI and in total. | P0 | Breakup accurate; usable for demand notice |
| COL-MA-028 | System shall generate foreclosure/settlement quote showing: principal outstanding, accrued interest to date, penal interest, charges, prepayment penalty (if applicable), total closure amount — valid for configurable period (default 7 days). | P0 | Quote generated in <5 seconds; validity tracked |

#### 12.5.4 Collection Agency View

| Req ID | Requirement | Priority | Acceptance Criteria |
|---|---|---|---|
| COL-MA-029 | System shall provide a dedicated collection agency portal/view showing only accounts allocated to that agency. | P0 | Agency sees only their allocated accounts; no cross-agency visibility |
| COL-MA-030 | Agency view shall show for each account: customer name, contact details (masked per policy), loan details, outstanding, DPD, overdue breakup, payment history (last 6 months), interaction history (last 3 months), PTP history. | P0 | Data visible per access policy; updated daily (or real-time for premium agencies) |
| COL-MA-031 | Agency view shall support: disposition capture, PTP recording, field visit report submission, payment recording, escalation to institution. | P0 | All actions available; logged with audit trail |
| COL-MA-032 | Agency view shall show performance dashboard: allocated accounts, contacted, PTP obtained, PTP converted, collected amount, collection rate, SLA adherence. | P0 | Dashboard with daily/weekly/monthly views |

### 12.6 Integration Events Consumed

| Event | From Context | Action Taken |
|---|---|---|
| `PaymentCredited` | Reconciliation (BC-01) | Update EMI status, payment history, outstanding breakup, last payment date |
| `BounceConfirmed` | Reconciliation (BC-01) | Update mandate presentation history, mandate health score, bounce count |
| `BucketChanged` | Delinquency (BC-02) | Update account bucket display, trigger refresh of overdue breakup |
| `NPAClassified` | Delinquency (BC-02) | Update account status to NPA |
| `SentimentScoreUpdated` | Sentiment (BC-05) | Update customer intelligence section in 360 view |
| `ConversationCompleted` | Communication (BC-04) | Add to interaction history timeline |
| `FieldVisitCompleted` | Agency (BC-07) | Add field visit report to interaction history; update address verification if applicable |
| `LegalNoticeServed` | Legal (BC-08) | Add to legal status section; update interaction timeline |

---

## END OF PART 06

Next: **COL-BRD-Part07-Agency-Field-Legal.md** -- Agency & Field + Legal & Recovery Bounded Contexts.
