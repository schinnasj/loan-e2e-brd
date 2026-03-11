# MODULE 15: AUDIT TRAIL AND MAKER-CHECKER
# MODULE 16: MULTI-ENTITY ARCHITECTURE
# MODULE 17: API-FIRST ARCHITECTURE
# MODULE 18: DATA PRIVACY AND SECURITY
# MODULE 19: DISASTER RECOVERY AND BUSINESS CONTINUITY
# NON-FUNCTIONAL REQUIREMENTS
# DATA MODEL HIGHLIGHTS
# INTEGRATION ARCHITECTURE
# REGULATORY COMPLIANCE MATRIX

---

## 18. AUDIT TRAIL AND MAKER-CHECKER WORKFLOWS

### 18.1 Audit Trail Requirements

**Requirement ID: LMS-AUD-001**
**Title:** Comprehensive Audit Trail for All System Actions
**Priority:** Must Have

**Description:** Every action in the LMS must generate an immutable audit trail entry. This is critical for regulatory compliance (RBI cybersecurity framework, internal audit, statutory audit, and RBI inspection).

**Audit Trail Data Captured:**

| Field | Description |
|---|---|
| Audit ID | Unique, sequential, system-generated identifier |
| Timestamp | Date and time of action (server clock, NTP synchronized) |
| User ID | System user who performed the action |
| User Role | Role of the user at the time of action |
| IP Address | Source IP address |
| Session ID | User session reference |
| Action Type | Create, Read, Update, Delete, Approve, Reject, Override, Login, Logout |
| Entity Type | Loan Account, Customer, Collateral, GL Entry, Rate Master, etc. |
| Entity ID | Specific record identifier |
| Old Value | Previous value (for updates) |
| New Value | New value (for updates and creates) |
| Remarks | User-entered or system-generated reason |
| Status | Success, Failed, Pending Approval |
| Module | Module name (GL, Payment, Collections, etc.) |

**Business Rules:**
- Audit trail records are immutable -- no deletion or modification allowed
- Audit trail stored separately from transactional data (dedicated audit schema/database)
- Retention: minimum 8 years (as per RBI record retention guidelines; 10 years recommended)
- Audit trail accessible to: internal auditors, compliance team, RBI inspectors (read-only, role-based)
- Tamper detection: hash chain or blockchain-style integrity verification for critical audit entries
- Performance: audit trail logging must not degrade transaction performance by more than 5%
- Bulk operations: each record in a bulk operation generates its own audit entry
- Sensitive data masking: Aadhaar number, account numbers masked in audit logs (show last 4 digits)
- Read access auditing: for sensitive data (customer PII, financial data), even read/view operations are logged

### 18.2 Maker-Checker Workflows

**Requirement ID: LMS-AUD-002**
**Title:** Configurable Maker-Checker Approval Workflows
**Priority:** Must Have

**Description:** All significant financial and administrative actions require at least maker-checker (two-person) approval. The system must support configurable multi-level approval workflows.

**Actions Requiring Maker-Checker:**

| Category | Actions | Minimum Levels |
|---|---|---|
| Financial | Loan disbursement, write-off, OTS approval, refund processing | Maker + Checker (+ Authority for above threshold) |
| Rate/Configuration | Interest rate change, penalty waiver, concession grant, GL mapping change | Maker + Checker |
| Customer Data | Customer name change, address change, phone number change, PAN update | Maker + Checker |
| Collateral | Collateral valuation update, collateral release, lien removal | Maker + Checker + Approver |
| Regulatory | NPA classification override, provision override, SMA reclassification | Maker + Checker + Senior Authority |
| System | User creation/modification, role assignment, product configuration | Maker + Checker (System Admin level) |
| Reporting | Regulatory return submission, bureau data submission | Maker + Checker |

**Approval Workflow Features:**
- Configurable number of approval levels (2 to 5 levels)
- Delegation: if primary approver unavailable, auto-route to delegate (configured per role)
- Escalation: if approval pending beyond SLA, escalate to next level
- Bulk approval: for routine operations, bulk checker workflow (e.g., daily NACH presentations)
- Rejection: checker can reject with mandatory remarks; returns to maker for correction or cancellation
- Recall: maker can recall a pending approval before checker acts
- Segregation of duties: maker and checker must be different users; same user cannot perform both roles
- Four-eyes principle: for high-value transactions (configurable threshold), additional approver required

**Acceptance Criteria:**
1. All configurable actions enforce maker-checker before execution
2. Same user cannot be both maker and checker for the same transaction
3. Approval SLA tracking with escalation
4. Complete audit trail of approval workflow (who approved, when, remarks)
5. Delegation and escalation rules are configurable

---

## 19. MULTI-ENTITY, MULTI-BRANCH, MULTI-PRODUCT ARCHITECTURE

### 19.1 Multi-Entity Support

**Requirement ID: LMS-ARC-001**
**Title:** Multi-Entity Deployment Architecture
**Priority:** Must Have

**Description:** The system must support deployment across multiple legal entities within a group (e.g., a bank and its subsidiary NBFC/HFC, or a group of NBFCs) from a single platform instance.

**Entity-Level Configuration:**

| Configuration | Entity-Specific | Description |
|---|---|---|
| GL Chart of Accounts | Yes | Each entity has its own CoA |
| Product Catalog | Yes | Products can be entity-specific or shared |
| Interest Rate Master | Yes | Entity-specific benchmark rates and spreads |
| Provisioning Norms | Yes | Banks vs NBFC provisioning rates differ |
| Regulatory Returns | Yes | Different regulatory returns per entity type |
| User Roles | Yes | Users tagged to one or more entities |
| Branch Master | Yes | Branches belong to specific entity |
| Holiday Calendar | Yes/Shared | Can be entity-specific or shared |
| Communication Templates | Yes | Entity branding and disclaimers differ |

**Business Rules:**
- Data isolation: strict entity-level data segregation; users cannot access other entity's data without explicit cross-entity authorization
- Consolidated reporting: group-level consolidated reporting available for management (with appropriate authorization)
- Cross-entity transactions: loan transfer between entities (e.g., NBFC to bank in co-lending buyout) supported with full accounting trail
- Regulatory compliance: each entity's compliance tracked independently

### 19.2 Multi-Branch Architecture

**Requirement ID: LMS-ARC-002**
**Title:** Branch-Level Operations and Reporting
**Priority:** Must Have

**Business Rules:**
- Branch master: branch code, name, address, region, zone, IFSC code, BSR code
- Branch-level operations: loan booking, customer onboarding, payment receipt, document management
- Centralized processing: credit underwriting, GL posting, NACH processing, regulatory reporting can be centralized
- Branch-level reports: loan portfolio, NPA, collections, disbursements
- Inter-branch transfer: loan account transfer between branches (with GL adjustments)
- Branch opening/closing: system handles branch closure by transferring all accounts and GL balances

### 19.3 Multi-Product Architecture

**Requirement ID: LMS-ARC-003**
**Title:** Product Configuration Engine
**Priority:** Must Have

**Product Configuration Parameters:**

| Parameter Group | Parameters |
|---|---|
| Basic | Product code, name, description, entity, category (retail/MSME/corporate/agri) |
| Loan Limits | Min/max loan amount, min/max tenure, min/max age at maturity |
| Interest | Calculation method, day count convention, rate type (fixed/floating), benchmark, spread range, penalty/penal charges |
| Repayment | EMI frequency, payment modes allowed, payment allocation waterfall, moratorium rules |
| Fees | Processing fee, documentation charges, prepayment penalty, bounce charges, stamp duty |
| Eligibility | Min credit score, max FOIR, max LTV, eligible borrower types, income criteria |
| Collateral | Required/optional, eligible collateral types, LTV norms, valuation rules |
| Insurance | Required/optional, eligible insurance types, premium payment rules |
| GL Mapping | Complete GL account mapping for all event types |
| Provisioning | Provisioning rates (if different from entity default), ECL parameters |
| Regulatory | PSL eligibility, CRILC reporting category, NHB classification |
| Documents | Required documents checklist, eKYC/CKYC applicability |
| Communication | Templates for sanction letter, welcome letter, demand notices, etc. |

**Business Rules:**
- Product creation requires complete parameter configuration
- Product modification: changes apply to new loans only (existing loans retain original terms unless explicitly migrated)
- Product versioning: system maintains version history of product configurations
- Product lifecycle: Draft -> Active -> Suspended -> Retired
- Product cloning: ability to clone existing product as starting point for new product
- Product hierarchy: product category -> product type -> product variant (e.g., Retail -> Home Loan -> Affordable Housing)

---

## 20. API-FIRST ARCHITECTURE

### 20.1 API Design Principles

**Requirement ID: LMS-API-001**
**Title:** RESTful API Architecture for All Business Functions
**Priority:** Must Have

**Design Standards:**
- RESTful API design following OpenAPI 3.0 specification
- JSON request/response format
- API versioning: URL-based (e.g., /api/v1/, /api/v2/)
- HTTP methods: GET (read), POST (create), PUT (update), PATCH (partial update), DELETE (soft delete)
- Standard HTTP status codes: 200 (OK), 201 (Created), 400 (Bad Request), 401 (Unauthorized), 403 (Forbidden), 404 (Not Found), 409 (Conflict), 422 (Unprocessable Entity), 429 (Too Many Requests), 500 (Server Error)
- Pagination: cursor-based or offset-based (configurable)
- Request/Response logging: all API calls logged for audit and debugging

### 20.2 Key API Categories

**Requirement ID: LMS-API-002**
**Title:** API Catalog for LMS Functions
**Priority:** Must Have

| API Category | Key Endpoints | Primary Consumers |
|---|---|---|
| Loan Application | POST /loans/apply, GET /loans/{id}/status | LOS, Digital Lending App, LSP |
| Loan Account | GET /loans/{id}, GET /loans/{id}/schedule, GET /loans/{id}/statement | Customer Portal, Mobile App, CBS |
| Payment | POST /loans/{id}/payment, GET /loans/{id}/payments | Payment Gateway, NACH System, CBS |
| Foreclosure | GET /loans/{id}/foreclosure-quote, POST /loans/{id}/foreclose | Customer Portal, Branch App |
| Collections | GET /collections/overdue, POST /collections/{id}/promise-to-pay | Collection CRM, Agency App |
| Credit Bureau | POST /bureau/inquiry, POST /bureau/report-submit | LOS, Bureau Connector |
| GL Integration | POST /gl/posting, GET /gl/reconciliation | CBS, ERP, GL System |
| Regulatory | GET /reports/{type}, POST /reports/{type}/generate | Compliance Portal, RBI Submission System |
| Master Data | GET /products, GET /branches, GET /rates | All integrated systems |
| Customer | GET /customers/{id}, PUT /customers/{id} | CRM, Customer Portal |
| Collateral | POST /collateral, GET /collateral/{id}, PUT /collateral/{id}/valuation | Valuation System, CERSAI |
| Insurance | POST /insurance/proposal, GET /insurance/{id}/status | Insurer API, Policy Admin |
| Communication | POST /notifications/send, GET /notifications/{id}/status | SMS Gateway, Email System |

### 20.3 API Security

**Requirement ID: LMS-API-003**
**Title:** API Security Framework
**Priority:** Must Have

**Security Measures:**

| Measure | Implementation |
|---|---|
| Authentication | OAuth 2.0 with JWT tokens; API key for server-to-server |
| Authorization | Role-based access control; scope-based token permissions |
| Encryption | TLS 1.2+ for all API communication; no plain HTTP |
| Rate Limiting | Configurable per client/endpoint (e.g., 100 req/min for inquiry, 1000 req/min for batch) |
| Input Validation | Request payload validation against schema; SQL injection prevention; XSS prevention |
| API Gateway | Centralized API gateway for routing, throttling, logging, transformation |
| IP Whitelisting | Optional IP-based access restriction for partner APIs |
| Request Signing | HMAC-SHA256 request signing for sensitive operations (disbursement, payment) |
| Idempotency | Idempotency key support for payment and disbursement APIs (prevent duplicate processing) |
| Audit | All API calls logged with request/response (PII masked), latency, status |

### 20.4 Webhook Support

**Requirement ID: LMS-API-004**
**Title:** Event-Driven Webhook Notifications
**Priority:** Should Have

**Webhook Events:**

| Event | Payload | Use Case |
|---|---|---|
| loan.disbursed | Loan ID, amount, date, account details | Notify LSP/partner of disbursement |
| loan.payment.received | Loan ID, payment amount, allocation, new balance | Real-time payment confirmation |
| loan.emi.bounced | Loan ID, bounce date, reason code, retry date | Trigger collection action in partner system |
| loan.npa.classified | Loan ID, classification, DPD, provision | Risk management alert |
| loan.closed | Loan ID, closure date, closure type | Trigger NOC, bureau update |
| loan.rate.changed | Loan ID, old rate, new rate, new EMI | Customer notification trigger |
| collateral.valuation.updated | Collateral ID, old value, new value, LTV | Risk monitoring |

**Business Rules:**
- Webhook endpoints registered per consumer with authentication (secret-based HMAC)
- Retry logic: 3 retries with exponential backoff (1 min, 5 min, 30 min)
- Failed webhook events queued for manual retry
- Webhook delivery log maintained for audit

---

## 21. DATA PRIVACY AND SECURITY

### 21.1 Data Privacy Framework

**Requirement ID: LMS-SEC-001**
**Title:** Data Privacy Compliance (DPDP Act, RBI Cybersecurity Framework)
**Priority:** Must Have

**Applicable Regulations:**
1. **Digital Personal Data Protection (DPDP) Act, 2023**: Consent-based data processing; data minimization; purpose limitation; right to erasure (with exceptions for regulatory requirements)
2. **RBI Master Direction on Information Technology Framework** (updated): Cyber security controls; data governance; incident management
3. **RBI Data Localization Circular (April 2018)**: All payment data must be stored in India; no mirroring outside India
4. **IT Act, 2000 and IT Rules**: Data protection; reasonable security practices

**System Requirements:**

| Requirement | Implementation |
|---|---|
| Data Classification | All data fields classified: Public, Internal, Confidential, Restricted |
| Encryption at Rest | AES-256 encryption for all databases; TDE (Transparent Data Encryption) |
| Encryption in Transit | TLS 1.2+ for all internal and external communication |
| PII Masking | Aadhaar (show last 4 digits), PAN (masked middle digits), account numbers (masked) in UI and logs |
| Access Control | Role-Based Access Control (RBAC) with principle of least privilege |
| Data Retention | Configurable per data category; minimum 8 years for financial records (RBI); right to erasure per DPDP Act (with regulatory exceptions) |
| Consent Management | Explicit consent capture for data collection, processing, and sharing; consent log maintained |
| Data Localization | All production data stored in Indian data centers; no cross-border data transfer without RBI approval |
| Key Management | HSM (Hardware Security Module) based key management; key rotation policies |
| Data Anonymization | Anonymized/pseudonymized data for analytics and testing environments |
| Breach Notification | Incident response plan; breach notification to CERT-In within 6 hours (per CERT-In rules, April 2022) |
| Vulnerability Assessment | Quarterly VAPT; annual penetration testing (per RBI guidelines) |

### 21.2 Application Security

**Requirement ID: LMS-SEC-002**
**Title:** Application-Level Security Controls
**Priority:** Must Have

| Control | Description |
|---|---|
| Authentication | Multi-factor authentication (MFA) for all users; password policy (min 8 chars, complexity, expiry 90 days, history 12) |
| Session Management | Configurable session timeout (default 15 min inactivity); concurrent session limit per user |
| OWASP Top 10 | Protection against: injection, broken authentication, sensitive data exposure, XSS, CSRF, SSRF, etc. |
| WAF | Web Application Firewall for all internet-facing components |
| SIEM Integration | Security events forwarded to SIEM (Security Information and Event Management) for monitoring |
| DLP | Data Loss Prevention controls for sensitive data (customer PII, financial data) |
| Privileged Access | PAM (Privileged Access Management) for admin/DBA access; session recording |
| Code Security | SAST/DAST in CI/CD pipeline; no deployment without security scan clearance |

---

## 22. DISASTER RECOVERY AND BUSINESS CONTINUITY

### 22.1 DR Requirements

**Requirement ID: LMS-DR-001**
**Title:** Disaster Recovery and Business Continuity Plan
**Priority:** Must Have

**RPO/RTO Targets:**

| System Tier | RPO (Recovery Point Objective) | RTO (Recovery Time Objective) | Description |
|---|---|---|---|
| Tier 1 -- Core LMS (transactions, GL) | 0 (zero data loss) | 2 hours | Synchronous replication to DR site |
| Tier 2 -- Reporting, MIS | 4 hours | 8 hours | Asynchronous replication |
| Tier 3 -- Archives, Audit Logs | 24 hours | 24 hours | Daily backup replication |

**DR Architecture:**
- Primary Data Center: Tier-3+ data center in India (as per RBI data localization)
- DR Site: Geographically separated (minimum 500 km) Tier-3+ data center
- Near-DR Site: Within same city for high-availability (metro DR)
- Active-Active: for critical components (API gateway, payment processing)
- Active-Passive: for database (primary-standby with synchronous replication)

**Business Rules:**
- DR drill: quarterly DR drill mandatory (per RBI IT framework); results documented and reported to Board
- Failover: automated failover for infra failures; manual controlled failover for site-level disaster
- Data consistency: post-failover, zero data loss for Tier 1 systems; reconciliation check mandatory before DR site goes live
- Fallback: plan for returning to primary site post-disaster; data sync from DR to primary
- BCP (Business Continuity Plan): documented process for operations to continue from DR site; staff allocation, communication plan, vendor coordination

---

## 23. NON-FUNCTIONAL REQUIREMENTS

### 23.1 Performance

**Requirement ID: LMS-NFR-001**
**Title:** Performance Benchmarks
**Priority:** Must Have

| Operation | Target Response Time | Throughput |
|---|---|---|
| Loan Account Inquiry | < 1 second | 500 concurrent users |
| EMI Payment Processing | < 2 seconds | 1,000 transactions/minute |
| Foreclosure Statement Generation | < 3 seconds | 100 concurrent requests |
| Amortization Schedule Generation | < 2 seconds | 200 concurrent requests |
| Interest Accrual (EOD batch) | < 2 hours for 10 million accounts | Full portfolio nightly |
| DPD/NPA Classification (EOD) | < 1 hour for 10 million accounts | Full portfolio nightly |
| Bureau Inquiry | < 5 seconds (includes bureau response) | 100 concurrent requests |
| Report Generation (standard) | < 30 seconds | 50 concurrent reports |
| Report Generation (large MIS) | < 5 minutes | 10 concurrent reports |
| API Response (general) | < 500 ms (95th percentile) | 2,000 requests/second |
| API Response (payment) | < 1 second (99th percentile) | 500 requests/second |

### 23.2 Scalability

**Requirement ID: LMS-NFR-002**
**Title:** Scalability Requirements
**Priority:** Must Have

| Parameter | Current | 3-Year Target | 5-Year Target |
|---|---|---|---|
| Active Loan Accounts | 5 million | 15 million | 30 million |
| Daily Transactions | 500,000 | 2 million | 5 million |
| Concurrent Users (Internal) | 2,000 | 5,000 | 10,000 |
| Concurrent API Consumers | 500 | 2,000 | 5,000 |
| Data Storage | 5 TB | 20 TB | 50 TB |
| Products Supported | 15 | 30 | 50 |
| Entities Supported | 1 | 3 | 5 |

**Business Rules:**
- Horizontal scalability: system must scale by adding nodes (not vertical scaling only)
- Database scalability: support for read replicas, sharding if required
- EOD batch scalability: parallel processing across product/branch partitions
- No degradation in response times up to 3-year target volumes

### 23.3 Availability

**Requirement ID: LMS-NFR-003**
**Title:** System Availability Targets
**Priority:** Must Have

| Component | Availability Target | Planned Downtime Window |
|---|---|---|
| Core LMS (Online Transactions) | 99.95% (excludes planned maintenance) | Sunday 2 AM - 6 AM (4 hours/week) |
| APIs (External/Partner) | 99.9% | Same as above |
| EOD Batch Processing | 99.9% completion within window | Nightly 11 PM - 5 AM |
| Customer Portal / Mobile | 99.9% | Same as core LMS |
| Reporting / MIS | 99.5% | Saturday night maintenance allowed |

### 23.4 Data Retention and Archival

**Requirement ID: LMS-NFR-004**
**Title:** Data Retention and Archival Policy
**Priority:** Must Have

| Data Category | Online Retention | Archival Period | Total Retention |
|---|---|---|---|
| Active Loan Data | Full tenure + 3 years | Additional 5 years archived | 8 years post-closure minimum |
| Closed Loan Data | 3 years post-closure | 5 years | 8 years post-closure |
| Transaction Data | Current FY + 2 FYs | Additional 5 FYs | 8 FYs |
| Audit Trail | 3 years online | 7 years archived | 10 years |
| Customer KYC Data | Duration of relationship + 5 years | Additional 3 years | 8 years (per KYC/AML guidelines) |
| Bureau Reports | 3 years | 5 years | 8 years |
| Regulatory Returns | 5 years online | 5 years archived | 10 years |
| Communication Logs | 2 years online | 6 years | 8 years |
| Collateral Documents | Tenure + 3 years | 5 years | 8 years post-closure |

**Business Rules:**
- Archived data must be retrievable within 24 hours
- Archival must not impact online system performance
- Archival runs during off-peak hours (weekend batch)
- Data destruction post retention: documented approval; certificate of destruction

---

## 24. DATA MODEL HIGHLIGHTS

### 24.1 Key Entities and Relationships

**Requirement ID: LMS-DM-001**
**Title:** Core Data Model Entities
**Priority:** Must Have

**Entity Relationship Overview:**

```
ENTITY (Legal Entity)
  |-- BRANCH
  |-- PRODUCT_MASTER
  |     |-- PRODUCT_GL_MAPPING
  |     |-- PRODUCT_FEE_STRUCTURE
  |     |-- PRODUCT_INTEREST_CONFIG
  |     |-- PRODUCT_ELIGIBILITY_RULES
  |
  |-- CUSTOMER
  |     |-- CUSTOMER_KYC
  |     |-- CUSTOMER_ADDRESS
  |     |-- CUSTOMER_BANK_ACCOUNT
  |     |-- CUSTOMER_EMPLOYMENT
  |     |-- CUSTOMER_INCOME
  |
  |-- LOAN_APPLICATION (LOS)
  |     |-- APPLICATION_DOCUMENT
  |     |-- CREDIT_ASSESSMENT
  |     |-- BUREAU_INQUIRY
  |
  |-- LOAN_ACCOUNT
  |     |-- LOAN_CUSTOMER_MAP (borrower, co-borrower, guarantor)
  |     |-- LOAN_DISBURSEMENT
  |     |-- LOAN_AMORTIZATION_SCHEDULE
  |     |-- LOAN_TRANSACTION (all financial transactions)
  |     |-- LOAN_INTEREST_ACCRUAL
  |     |-- LOAN_RATE_HISTORY
  |     |-- LOAN_MANDATE (NACH/UPI/ECS)
  |     |-- LOAN_PDC
  |     |-- LOAN_INSURANCE
  |     |-- LOAN_OVERDUE (DPD, SMA, NPA tracking)
  |     |-- LOAN_PROVISION
  |     |-- LOAN_RESTRUCTURING
  |     |-- LOAN_COLLECTION_ACTIVITY
  |     |-- LOAN_LEGAL_CASE
  |     |-- LOAN_COMMUNICATION_LOG
  |     |-- LOAN_DOCUMENT
  |     |-- LOAN_CLOSURE
  |
  |-- COLLATERAL
  |     |-- COLLATERAL_VALUATION
  |     |-- COLLATERAL_CERSAI
  |     |-- COLLATERAL_LIEN
  |     |-- COLLATERAL_LOAN_MAP (many-to-many: one collateral can secure multiple loans)
  |
  |-- GL_ACCOUNT_MASTER
  |     |-- GL_POSTING (all GL entries)
  |     |-- GL_RECONCILIATION
  |
  |-- RATE_MASTER
  |     |-- BENCHMARK_RATE_HISTORY
  |
  |-- SUSPENSE_ENTRY
  |-- RECONCILIATION_ENTRY
  |
  |-- USER_MASTER
  |     |-- USER_ROLE
  |     |-- ROLE_PERMISSION
  |
  |-- AUDIT_TRAIL
```

### 24.2 Key Entity Attributes

**LOAN_ACCOUNT (Central Entity):**

| Attribute | Type | Description |
|---|---|---|
| loan_account_id | VARCHAR(20) | Unique loan account number |
| entity_id | FK | Legal entity |
| branch_id | FK | Booking branch |
| product_id | FK | Loan product |
| primary_customer_id | FK | Primary borrower |
| sanction_amount | DECIMAL(15,2) | Sanctioned loan amount |
| disbursed_amount | DECIMAL(15,2) | Total disbursed amount |
| outstanding_principal | DECIMAL(15,2) | Current principal outstanding |
| outstanding_interest | DECIMAL(15,2) | Current interest outstanding |
| outstanding_penal | DECIMAL(15,2) | Current penal charges outstanding |
| interest_rate | DECIMAL(5,4) | Current applicable interest rate |
| benchmark_rate | DECIMAL(5,4) | Current benchmark component |
| spread | DECIMAL(5,4) | Spread over benchmark |
| emi_amount | DECIMAL(12,2) | Current EMI amount |
| emi_frequency | ENUM | MONTHLY, QUARTERLY, HALF_YEARLY, ANNUAL |
| first_emi_date | DATE | Date of first EMI |
| last_emi_date | DATE | Date of last EMI (maturity) |
| next_emi_date | DATE | Next EMI due date |
| total_installments | INT | Total number of installments |
| paid_installments | INT | Number of installments paid |
| remaining_installments | INT | Remaining installments |
| rate_type | ENUM | FIXED, FLOATING |
| rate_reset_frequency | ENUM | IMMEDIATE, MONTHLY, QUARTERLY, HALF_YEARLY, ANNUAL |
| next_rate_reset_date | DATE | Date of next rate reset |
| disbursement_date | DATE | Date of first/full disbursement |
| maturity_date | DATE | Scheduled loan maturity date |
| dpd | INT | Current Days Past Due |
| asset_classification | ENUM | STANDARD, SMA_0, SMA_1, SMA_2, NPA_SUB, NPA_D1, NPA_D2, NPA_D3, LOSS |
| npa_date | DATE | Date of NPA classification (if applicable) |
| provision_amount | DECIMAL(15,2) | Current provision held |
| ecl_stage | ENUM | STAGE_1, STAGE_2, STAGE_3 |
| ecl_amount | DECIMAL(15,2) | Current ECL provision |
| restructuring_flag | BOOLEAN | Whether account is restructured |
| wilful_defaulter_flag | BOOLEAN | Whether classified as wilful defaulter |
| psl_category | VARCHAR(50) | PSL classification |
| account_status | ENUM | PENDING_DISBURSEMENT, ACTIVE, MORATORIUM, RESTRUCTURED, NPA, CLOSED, WRITTEN_OFF, SETTLED |
| closure_date | DATE | Account closure date |
| closure_type | ENUM | NORMAL, FORECLOSURE, OTS, WRITE_OFF |
| nach_umrn | VARCHAR(20) | NACH mandate reference |
| created_date | TIMESTAMP | Account creation date |
| created_by | FK | User who created |
| last_modified_date | TIMESTAMP | Last modification date |
| last_modified_by | FK | User who last modified |

**LOAN_TRANSACTION:**

| Attribute | Type | Description |
|---|---|---|
| txn_id | VARCHAR(30) | Unique transaction ID |
| loan_account_id | FK | Loan account |
| txn_date | DATE | Transaction business date |
| value_date | DATE | Value date |
| txn_type | ENUM | EMI_RECEIPT, INTEREST_ACCRUAL, PRINCIPAL_PAYMENT, PENAL_CHARGE, FEE_CHARGE, PREPAYMENT, DISBURSEMENT, REVERSAL, WRITE_OFF, RECOVERY, INSURANCE_DEBIT, SUBVENTION_CREDIT, REFUND |
| txn_amount | DECIMAL(15,2) | Transaction amount |
| principal_component | DECIMAL(15,2) | Principal portion (for EMI) |
| interest_component | DECIMAL(15,2) | Interest portion (for EMI) |
| penal_component | DECIMAL(15,2) | Penal charges portion |
| fee_component | DECIMAL(15,2) | Fee portion |
| payment_mode | ENUM | NACH, UPI, NEFT, RTGS, CASH, CHEQUE, ONLINE, INTERNAL |
| payment_reference | VARCHAR(50) | UTR/UMRN/Cheque number |
| gl_posted | BOOLEAN | Whether GL entry has been posted |
| gl_posting_date | DATE | Date of GL posting |
| running_balance | DECIMAL(15,2) | Principal outstanding after this transaction |
| remarks | VARCHAR(500) | Transaction remarks |

---

## 25. INTEGRATION ARCHITECTURE

### 25.1 Integration Points Summary

**Requirement ID: LMS-INT-ARCH-001**
**Title:** System Integration Map
**Priority:** Must Have

| External System | Integration Type | Direction | Protocol | Data Format | Frequency |
|---|---|---|---|---|---|
| Core Banking System (CBS) | API + Batch | Bidirectional | REST API + SFTP | JSON + CSV | Real-time + Daily |
| NPCI NACH | Batch File | Bidirectional | SFTP (via sponsor bank) | Fixed-width text (ACH format) | Daily |
| NPCI UPI | API | Bidirectional | REST API (via PSP) | JSON | Real-time |
| Credit Bureaus (CIBIL, etc.) | API | Bidirectional | REST API / SOAP | JSON / XML (TUDF for push) | On-demand (pull) + Monthly (push) |
| CERSAI | API / Web | Push | REST API / Web portal | JSON / Form | On event (filing) |
| Account Aggregator | API | Pull | REST API (AA spec) | JSON (FIR) | On-demand (with consent) |
| Payment Gateways | API | Bidirectional | REST API | JSON | Real-time |
| Insurance Companies | API / Batch | Bidirectional | REST API / SFTP | JSON / CSV | On event + Monthly |
| GSTN | API | Pull/Push | REST API | JSON | Monthly |
| Income Tax (PAN Verification) | API | Pull | REST API (NSDL) | JSON | On-demand |
| Aadhaar eKYC / eSign | API | Pull | REST API (UIDAI via KUA) | JSON / XML | On-demand |
| SMS Gateway | API | Push | REST API | JSON | On event |
| Email Gateway | API | Push | SMTP / REST API | HTML/Text | On event |
| WhatsApp Business | API | Push | REST API (Meta BSP) | JSON | On event |
| RBI Returns Submission | Batch File / Portal | Push | SFTP / Web upload | XBRL / CSV / Excel | As per return schedule |
| NHB Returns | Batch File / Portal | Push | SFTP / Web upload | Excel / CSV | Quarterly/Annual |
| Legal Case Management | API | Bidirectional | REST API | JSON | On event |
| Valuation System | API | Bidirectional | REST API | JSON | On event |
| Document Management (DMS) | API | Bidirectional | REST API | JSON + Binary (documents) | On event |
| Data Warehouse / Analytics | Batch / CDC | Push | SFTP / Kafka / CDC | Parquet / CSV / JSON | Daily + Real-time (CDC) |

### 25.2 Integration Patterns

**Requirement ID: LMS-INT-ARCH-002**
**Title:** Standard Integration Patterns
**Priority:** Must Have

| Pattern | Use Case | Implementation |
|---|---|---|
| Synchronous API | Real-time queries, payment processing, bureau inquiry | REST API with timeout (30 sec default) |
| Asynchronous API | Long-running processes (report generation, bulk operations) | Request-acknowledgment pattern; status polling or webhook callback |
| Batch File Exchange | NACH files, bureau reporting files, regulatory returns | SFTP with PGP encryption; scheduled file exchange |
| Event-Driven | GL posting, classification changes, notifications | Message queue (Kafka/RabbitMQ); event sourcing for critical flows |
| CDC (Change Data Capture) | Data warehouse synchronization | Debezium / Oracle GoldenGate; real-time data replication |

**Error Handling Standard:**
- All integrations must handle: timeout, connection failure, invalid response, partial failure
- Retry policy: configurable per integration (max retries, backoff strategy)
- Dead letter queue: failed messages after max retries parked for manual investigation
- Circuit breaker pattern: if external system consistently fails, circuit opens to prevent cascade
- Fallback: for critical integrations (bureau, NACH), manual fallback process documented

---

## 26. REGULATORY COMPLIANCE MATRIX

**Requirement ID: LMS-COMP-001**
**Title:** Regulatory Compliance Traceability Matrix
**Priority:** Must Have

| Regulation / Circular | Key Requirement | LMS Module | Requirement ID(s) |
|---|---|---|---|
| RBI IRAC Master Direction (DOR.STR.REC.4/21.04.048/2021-22) | NPA recognition (90 DPD); asset classification; provisioning norms | Overdue Management, GL | LMS-OVD-002, LMS-OVD-006, LMS-GL-009 |
| RBI IRAC -- Interest Recognition | Non-recognition of interest on NPAs | Interest Computation | LMS-INT-014 |
| RBI Digital Lending Guidelines (Sep 2022) | KFS, APR disclosure, cooling-off, direct disbursement, FLDG | Digital Lending, Origination | LMS-DLC-001, LMS-DLC-002, LMS-ORG-004 |
| RBI FLDG Circular (Jun 2023) | FLDG cap at 5% | Digital Lending | LMS-DLC-002 |
| RBI Penal Charges Circular (Aug 2023, eff. Apr 2024) | Penal charges not penal interest; no compounding | Interest Computation | LMS-INT-010 |
| RBI EBLR Circular (Sep 2019) | External benchmark linking; quarterly reset; spread transparency | Interest Computation | LMS-INT-007, LMS-INT-008 |
| RBI Fair Practices Code | Transparency; prepayment norms; grievance redressal | Communication, Prepayment | LMS-COM-002, LMS-ECL-002 |
| RBI KYC Master Direction | eKYC, CKYC, V-CIP, periodic update | Origination | LMS-ORG-001 |
| SARFAESI Act 2002 | Enforcement of security interest | Overdue Management | LMS-OVD-005 |
| CERSAI Act 2011 | Registration of security interests | Collateral | LMS-COL-003 |
| IND AS 109 | ECL model; EIR; staging; fair value | GL, Interest Computation | LMS-GL-011, LMS-GL-004 |
| Income Tax Act -- Section 194A | TDS on interest | Interest Computation | LMS-INT-013 |
| Income Tax Act -- Section 80C/24/80E | Tax deduction certificates | Interest-Principal Allocation | LMS-IPA-002 |
| RBI Scale-Based Regulation (Oct 2022) | Layer-wise requirements for NBFCs | Architecture (entity config) | LMS-ARC-001 |
| NHB Directions | HFC-specific IRAC, LTV, reporting | Product Configuration, Reporting | LMS-ARC-003, LMS-REG-001 |
| DPDP Act 2023 | Data privacy; consent; purpose limitation | Security & Privacy | LMS-SEC-001 |
| RBI IT Framework / Cybersecurity | IT governance; VAPT; audit trail; DR | Security, DR, Audit | LMS-SEC-002, LMS-DR-001, LMS-AUD-001 |
| CERT-In Directions (Apr 2022) | 6-hour breach notification; log retention 180 days | Security | LMS-SEC-001 |
| RBI Data Localization (Apr 2018) | Payment data stored in India | Security | LMS-SEC-001 |
| RBI Framework for Resolution of Stressed Assets | Resolution timeline; ICA; additional provisioning | Overdue Management | LMS-OVD-002, LMS-PAY-016 |
| RBI Wilful Defaulter Circular | Identification; reporting; credit denial | Overdue Management | LMS-OVD-009 |
| GST Act | GST on fees, charges | Payment Processing | LMS-PAY-018, LMS-ECL-002 |
| PMAY-CLSS | Interest subvention for housing | Interest Computation | LMS-INT-012 |
| RBI Microfinance Guidelines (Mar 2022) | Pricing cap; assessment norms; no prepayment penalty | Product Configuration | LMS-ARC-003 |
| RBI Co-Lending Guidelines (Nov 2020) | CLM framework; 80:20 split; escrow | Reconciliation, Product | LMS-REC-009, LMS-ARC-003 |

---
