# PART 09 -- NON-FUNCTIONAL REQUIREMENTS, DATA MODEL, API CONTRACTS & GLOSSARY

---

## 18. NON-FUNCTIONAL REQUIREMENTS

### 18.1 Performance Requirements

| Req ID | Requirement | Metric | Target |
|---|---|---|---|
| COL-NFR-001 | Daily DPD batch calculation for entire portfolio | Throughput | 10 million accounts in <2 hours |
| COL-NFR-002 | On-demand DPD recalculation (event-triggered) | Latency | <500ms per account |
| COL-NFR-003 | Reconciliation auto-matching engine | Throughput | 1 million transactions matched in <30 minutes |
| COL-NFR-004 | Customer 360 page load | Latency | <3 seconds (95th percentile) including all data sources |
| COL-NFR-005 | Payment link generation | Latency | <1 second |
| COL-NFR-006 | Sentiment score computation (per interaction) | Latency | <5 minutes after interaction completion |
| COL-NFR-007 | Propensity model scoring (batch) | Throughput | 10 million accounts scored in <4 hours |
| COL-NFR-008 | Dashboard refresh | Latency | <5 minutes data lag from source event |
| COL-NFR-009 | AI chatbot response | Latency | <3 seconds per turn |
| COL-NFR-010 | Workflow step execution | Latency | Within ±5 minutes of scheduled time |
| COL-NFR-011 | Concurrent users | Capacity | 5,000 concurrent internal users + 500 agency users |
| COL-NFR-012 | API response time | Latency | <200ms (95th percentile) for read APIs; <500ms for write APIs |
| COL-NFR-013 | Event processing throughput | Throughput | 10,000 events/second sustained |

### 18.2 Scalability Requirements

| Req ID | Requirement | Target |
|---|---|---|
| COL-NFR-014 | Portfolio size support | Up to 50 million active loan accounts |
| COL-NFR-015 | Transaction volume | Up to 5 million reconciliation transactions per day |
| COL-NFR-016 | Communication volume | Up to 10 million messages per day across all channels |
| COL-NFR-017 | Horizontal scaling | All bounded contexts must support horizontal scaling (stateless services, partitioned data) |
| COL-NFR-018 | Database scaling | Support database sharding by entity/region if single-node capacity exceeded |
| COL-NFR-019 | Event bus scaling | Message broker must support 50,000 messages/second peak |

### 18.3 Availability & Reliability

| Req ID | Requirement | Target |
|---|---|---|
| COL-NFR-020 | System availability | 99.9% uptime (max ~8.7 hours downtime/year) |
| COL-NFR-021 | Reconciliation engine availability | 99.95% (critical for payment processing) |
| COL-NFR-022 | RPO (Recovery Point Objective) | <15 minutes (no more than 15 min data loss in disaster) |
| COL-NFR-023 | RTO (Recovery Time Objective) | <1 hour (system recovery within 1 hour of disaster) |
| COL-NFR-024 | Data replication | Synchronous replication within region; asynchronous cross-region |
| COL-NFR-025 | Disaster recovery | Active-passive DR in different geographic zone within India |
| COL-NFR-026 | Zero data loss for financial transactions | Reconciliation and payment events must have zero-loss guarantee (persistent messaging, WAL) |

### 18.4 Security Requirements

| Req ID | Requirement | Details |
|---|---|---|
| COL-NFR-027 | Authentication | SSO integration (SAML 2.0 / OIDC); MFA mandatory for all users |
| COL-NFR-028 | Authorization | RBAC with fine-grained permissions per bounded context and data entity |
| COL-NFR-029 | Data encryption at rest | AES-256 for all data stores; HSM for key management |
| COL-NFR-030 | Data encryption in transit | TLS 1.2+ for all API communication; mTLS for inter-service communication |
| COL-NFR-031 | PII protection | PAN, Aadhaar, bank account numbers masked in UI and logs (show last 4 digits only); tokenization in storage |
| COL-NFR-032 | API security | OAuth 2.0 + JWT; rate limiting; IP whitelisting for external APIs |
| COL-NFR-033 | Audit logging | All access to customer data logged (who, what, when, from where); immutable audit log |
| COL-NFR-034 | Vulnerability management | OWASP Top 10 compliance; quarterly VAPT; no critical/high vulnerabilities in production |
| COL-NFR-035 | Data residency | All data stored within India per RBI data localization directive (April 2018, reiterated 2019) |
| COL-NFR-036 | Session management | Session timeout: 15 minutes idle; absolute timeout: 8 hours; concurrent session limit per user |
| COL-NFR-037 | Call recording security | Call recordings encrypted at rest; access restricted to authorized roles; tamper detection |
| COL-NFR-038 | Agency data isolation | Agency users see only their allocated accounts; cross-agency data leakage prevention |

### 18.5 Integration Requirements

| Req ID | Integration Point | Protocol | Direction | Frequency |
|---|---|---|---|---|
| COL-NFR-039 | Core Banking System (CBS) / LMS | REST API + SFTP | Bidirectional | Real-time (API) + Daily (file) |
| COL-NFR-040 | NPCI — NACH | SFTP (ISO 8583 / NPCI format) | Bidirectional | 4x daily (batch files) |
| COL-NFR-041 | NPCI — UPI AutoPay | REST API (UPI 2.0) | Bidirectional | Real-time |
| COL-NFR-042 | Payment Gateway (Razorpay, BillDesk, etc.) | REST API + Webhooks | Bidirectional | Real-time + Daily settlement |
| COL-NFR-043 | BBPS (Bharat Bill Payment System) | REST API | Inbound | Real-time |
| COL-NFR-044 | Credit Bureaus (CIBIL, Equifax, Experian, CRIF) | REST API + SFTP | Bidirectional | Monthly (outbound file) + On-demand (inquiry API) |
| COL-NFR-045 | RBI — CRILC | SFTP (RBI prescribed format) | Outbound | Weekly (SMA) + Quarterly (full) |
| COL-NFR-046 | SMS Gateway | REST API | Outbound | Real-time |
| COL-NFR-047 | WhatsApp BSP (Gupshup/Infobip) | REST API + Webhooks | Bidirectional | Real-time |
| COL-NFR-048 | IVR/Telephony (Ozonetel, Exotel, etc.) | REST API + SIP | Bidirectional | Real-time |
| COL-NFR-049 | Email Service (SES, SendGrid) | REST API + SMTP | Outbound + Webhooks | Real-time |
| COL-NFR-050 | AI/ML Platform (model serving) | REST API (gRPC optional) | Internal | Real-time + Batch |
| COL-NFR-051 | CERSAI (Central Registry) | REST API | Outbound | On-event (charge satisfaction) |
| COL-NFR-052 | TRAI DND Registry | REST API | Outbound | Before each communication |
| COL-NFR-053 | DLT Platform (SMS template) | REST API | Outbound | On template registration |
| COL-NFR-054 | e-Auction Platform (for SARFAESI) | REST API | Bidirectional | On-event |
| COL-NFR-055 | GST Portal (for EWS) | REST API | Inbound | Monthly (with borrower consent) |

---

## 19. DATA MODEL HIGHLIGHTS PER CONTEXT

### 19.1 Core Entities — Entity Relationship Summary

```
┌─────────────────┐       ┌──────────────────┐       ┌───────────────────┐
│   CUSTOMER       │1────M│   LOAN_ACCOUNT    │1────M │   EMI_SCHEDULE     │
│                  │       │                   │       │                    │
│ customer_id (PK) │       │ loan_account_id   │       │ emi_id (PK)        │
│ name             │       │ customer_id (FK)  │       │ loan_account_id    │
│ pan_number       │       │ product_type      │       │ emi_number         │
│ mobile           │       │ sanctioned_amount │       │ due_date           │
│ email            │       │ outstanding       │       │ emi_amount         │
│ preferred_lang   │       │ interest_rate     │       │ principal_portion  │
│ risk_category    │       │ emi_amount        │       │ interest_portion   │
│ sentiment_score  │       │ dpd               │       │ status             │
└─────────────────┘       │ bucket            │       │ paid_amount        │
                           │ asset_class       │       │ paid_date          │
                           │ npa_date          │       └───────────────────┘
                           └──────────────────┘
                               │1          │1
                               │           │
                               M│          M│
                    ┌───────────────┐  ┌──────────────────┐
                    │   MANDATE      │  │ COLLECTION_CASE  │
                    │                │  │                  │
                    │ mandate_id     │  │ case_id (PK)     │
                    │ loan_acct_id   │  │ loan_account_id  │
                    │ mandate_type   │  │ status           │
                    │ umrn           │  │ priority         │
                    │ amount         │  │ assigned_to      │
                    │ status         │  │ opened_date      │
                    │ dest_bank      │  │ current_dpd      │
                    │ dest_account   │  │ current_bucket   │
                    └───────────────┘  └──────────────────┘
                                            │1
                                            │
                                            M│
                              ┌─────────────────────────┐
                              │   CASE_ACTIVITY          │
                              │                          │
                              │ activity_id (PK)         │
                              │ case_id (FK)             │
                              │ activity_type            │
                              │ channel                  │
                              │ disposition              │
                              │ performed_by             │
                              │ performed_at             │
                              │ notes                    │
                              └─────────────────────────┘
```

### 19.2 Key Entities per Bounded Context

| Context | Key Entities | Key Relationships |
|---|---|---|
| BC-01 Reconciliation | ReconciliationBatch, IncomingTransaction, MatchedPayment, SuspenseEntry, BounceRecord | Transaction → MatchedPayment → LoanAccount + EMI |
| BC-02 Delinquency | DelinquencyProfile, BucketMovement, CollectionCase, CaseActivity, PTPromise, ProvisionRecord | LoanAccount → DelinquencyProfile → CollectionCase → Activities |
| BC-03 Workflow | CollectionStrategy, WorkflowTemplate, WorkflowStep, WorkflowExecution, StepExecution, Campaign | Strategy → Template → Steps; Execution → Case |
| BC-04 Communication | CommunicationRecord, ConversationSession, ConversationTurn, MessageTemplate | Communication → LoanAccount + Customer; Session → Turns |
| BC-05 Sentiment | IntelligenceProfile, SentimentHistory, BehavioralSignal, EWSAlert, PropensityScore | Profile → Customer; Alert → LoanAccount |
| BC-06 Account | LoanAccountSnapshot, EMIScheduleEntry, CustomerProfile, Mandate, InteractionRecord | Customer → Accounts → EMIs; Customer → Mandates |
| BC-07 Agency | CollectionAgency, AgencyContract, AccountAllocation, FieldVisit, FieldAgent, AllocationActivity | Agency → Allocations → LoanAccount; Visit → Agent + Account |
| BC-08 Legal | LegalCase, LegalNotice, Hearing, CourtOrder, SARFAESIAction, Settlement, WriteOff | Case → LoanAccount; SARFAESI → Collateral; Settlement → Approvals |
| BC-09 Payment | PaymentLink, PaymentAttempt, RepaymentPlan, PlanInstallment, MandateRepresentation | Link → LoanAccount; Plan → Installments |
| BC-10 Compliance | ComplianceRule, ComplianceViolation, AuditTrail, RegulatoryReport, ComplaintRecord | Violation → Rule + Account; Report → Submission |
| BC-11 Analytics | PortfolioSnapshot, RollRateMatrix, VintageAnalysis, CERMetric, AgencyScorecard, CampaignMetric | Aggregate/materialized views across all contexts |

---

## 20. SAMPLE API CONTRACTS

### 20.1 Customer 360 API

```
GET /api/v1/collections/customer-360/{customerId}

Response 200:
{
  "customerId": "CUST-2024-001234",
  "demographics": {
    "name": "Rajesh Kumar Sharma",
    "dateOfBirth": "1985-03-15",
    "pan": "XXXXX1234X",
    "mobile": "+91-98XXXXXX90",
    "email": "r****a@gmail.com",
    "preferredLanguage": "hi",
    "kycStatus": "VERIFIED"
  },
  "loanAccounts": [
    {
      "loanAccountId": "ACC/2024/HL/001234",
      "productType": "HOME_LOAN",
      "sanctionedAmount": 5000000,
      "disbursedAmount": 5000000,
      "principalOutstanding": 4250000,
      "totalOutstanding": 4325000,
      "emiAmount": 45000,
      "interestRate": 8.75,
      "dpd": 45,
      "bucket": "SMA-1",
      "assetClassification": "STANDARD",
      "overdueAmount": {
        "principal": 35000,
        "interest": 28500,
        "penalInterest": 1200,
        "charges": 500,
        "total": 65200
      },
      "nextEmiDueDate": "2026-04-05",
      "mandates": [
        {
          "mandateId": "MND-001",
          "type": "NACH",
          "umrn": "NACH00001234567890",
          "status": "ACTIVE",
          "amount": 45000,
          "healthScore": "AMBER",
          "successRate": 83.3,
          "lastPresentationDate": "2026-03-05",
          "lastPresentationResult": "BOUNCE",
          "lastBounceCode": "01",
          "consecutiveBounces": 2
        }
      ],
      "collectionCase": {
        "caseId": "CASE-2026-045678",
        "status": "IN_PROGRESS",
        "priority": "HIGH",
        "assignedAgency": "ABC Collections Pvt Ltd",
        "ptpHistory": [
          {
            "promisedDate": "2026-03-08",
            "promisedAmount": 65200,
            "status": "BROKEN",
            "capturedVia": "AI_VOICE"
          }
        ]
      }
    }
  ],
  "sentimentProfile": {
    "currentScore": -0.15,
    "label": "NEGATIVE",
    "riskCategory": "HARDSHIP",
    "propensityToPay": 0.42,
    "trend": "DETERIORATING",
    "lastUpdated": "2026-03-10T14:30:00Z"
  },
  "recentInteractions": [
    {
      "date": "2026-03-10",
      "channel": "AI_VOICE",
      "direction": "OUTBOUND",
      "disposition": "PTP_RECEIVED",
      "summary": "Customer expressed financial difficulty due to medical emergency. Promised ₹65,200 by March 8. Tone: cooperative but stressed."
    },
    {
      "date": "2026-03-05",
      "channel": "SMS",
      "direction": "OUTBOUND",
      "disposition": "DELIVERED",
      "summary": "EMI reminder sent with payment link"
    }
  ],
  "legalStatus": null,
  "coBorrowers": [],
  "collateral": {
    "type": "RESIDENTIAL_PROPERTY",
    "address": "Flat 302, ..., Mumbai - 400001",
    "lastValuation": 7500000,
    "lastValuationDate": "2025-09-15",
    "insuranceStatus": "ACTIVE"
  }
}
```

### 20.2 DPD Calculation API (Internal)

```
POST /api/v1/delinquency/calculate-dpd

Request:
{
  "loanAccountId": "ACC/2024/HL/001234",
  "calculationDate": "2026-03-11",
  "trigger": "PAYMENT_EVENT"
}

Response 200:
{
  "loanAccountId": "ACC/2024/HL/001234",
  "calculationDate": "2026-03-11",
  "dpd": 45,
  "previousDpd": 44,
  "oldestUnpaidEmi": {
    "emiNumber": 13,
    "dueDate": "2026-01-25",
    "amount": 45000,
    "paidAmount": 0
  },
  "bucket": "SMA-1",
  "previousBucket": "SMA-1",
  "bucketChanged": false,
  "assetClassification": "STANDARD",
  "provisioningRate": 0.40
}
```

### 20.3 Workflow Trigger API (Internal)

```
POST /api/v1/workflow/trigger

Request:
{
  "triggerType": "BUCKET_CHANGED",
  "loanAccountId": "ACC/2024/PL/005678",
  "customerId": "CUST-2024-005678",
  "fromBucket": "SMA-0",
  "toBucket": "SMA-1",
  "dpd": 31,
  "productType": "PERSONAL_LOAN",
  "outstandingAmount": 150000,
  "overdueAmount": 18500
}

Response 200:
{
  "workflowExecutionId": "WF-EXEC-2026-789012",
  "strategyId": "STR-MED-PL-SAL",
  "templateId": "WF-MED-MULTI-CHANNEL",
  "firstStepAction": "SEND_SMS",
  "firstStepScheduledAt": "2026-03-11T10:00:00+05:30",
  "totalSteps": 8
}
```

### 20.4 Payment Link API

```
POST /api/v1/payment/generate-link

Request:
{
  "loanAccountId": "ACC/2024/PL/005678",
  "amount": 18500,
  "allowPartialPayment": true,
  "expiryHours": 72,
  "supportedModes": ["UPI", "NETBANKING", "DEBIT_CARD"],
  "channel": "WHATSAPP"
}

Response 200:
{
  "paymentLinkId": "PL-2026-456789",
  "shortUrl": "https://pay.bank.in/s/aBcDeF",
  "qrCodeUrl": "https://api.bank.in/qr/PL-2026-456789.png",
  "upiDeepLink": "upi://pay?pa=loans@bank&pn=BankName&am=18500&tr=PL-2026-456789",
  "expiresAt": "2026-03-14T10:00:00+05:30",
  "status": "ACTIVE"
}
```

---

## 21. INTEGRATION ARCHITECTURE

### 21.1 Event-Driven Architecture

```
                     ┌─────────────────────────┐
                     │   EVENT BUS / BROKER     │
                     │   (Kafka / RabbitMQ)     │
                     │                          │
                     │  Topics:                 │
                     │  - reconciliation.events  │
                     │  - delinquency.events     │
                     │  - workflow.events         │
                     │  - communication.events    │
                     │  - sentiment.events        │
                     │  - account.events          │
                     │  - agency.events            │
                     │  - legal.events             │
                     │  - payment.events           │
                     │  - compliance.audit         │
                     └─────────────────────────┘
                          ▲               │
                          │ Publish       │ Subscribe
                          │               ▼
              ┌───────────────────────────────────────┐
              │        BOUNDED CONTEXT SERVICES         │
              │                                         │
              │  Each context:                          │
              │  ├── API Gateway (REST)                 │
              │  ├── Application Service                │
              │  ├── Domain Layer                       │
              │  ├── Infrastructure (DB, Cache, Queue)  │
              │  └── Event Publisher / Subscriber       │
              └───────────────────────────────────────┘
```

### 21.2 Deployment Architecture

| Component | Technology Recommendation | Justification |
|---|---|---|
| API Gateway | Kong / AWS API Gateway | Rate limiting, auth, routing |
| Service Runtime | Kubernetes (EKS/AKS/GKE) or OpenShift | Container orchestration, auto-scaling |
| Event Broker | Apache Kafka | High throughput, durability, event replay |
| Primary Database | PostgreSQL (per context) | ACID compliance, JSON support, mature |
| Read Cache | Redis | Customer 360 caching, session management |
| Search | Elasticsearch | Full-text search across interactions, audit logs |
| Object Storage | S3-compatible (MinIO on-prem) | Call recordings, documents, evidence photos |
| ML Platform | MLflow + custom serving | Model versioning, A/B testing, monitoring |
| Monitoring | Prometheus + Grafana | Metrics, alerting, dashboards |
| Logging | ELK Stack (Elasticsearch, Logstash, Kibana) | Centralized logging, audit analysis |
| CI/CD | Jenkins / GitLab CI | Automated build, test, deploy |

---

## 22. REGULATORY COMPLIANCE MATRIX

| Regulation | Applicable Module(s) | Key Requirements | System Compliance Mechanism |
|---|---|---|---|
| RBI IRAC Norms (Master Direction Sep 2024) | Delinquency (BC-02) | 90-day NPA classification; asset classification; provisioning norms | Automated DPD calculation; auto-classification; provisioning computation |
| RBI Fair Practices Code | Communication (BC-04), Compliance (BC-10) | No harassment; call timing restrictions; agent identification; no third-party disclosure | Real-time enforcement engine; call monitoring; template compliance |
| RBI CRILC Framework | Delinquency (BC-02), Compliance (BC-10) | Weekly SMA reporting; quarterly large credit reporting | Auto-generated reports per RBI format |
| RBI Digital Lending Guidelines (Sep 2022) | Payment (BC-09), Communication (BC-04) | LSP disclosure; data privacy; cooling-off period; digital consent | Compliance checks in payment and communication flows |
| SARFAESI Act, 2002 | Legal (BC-08) | 60-day notice; possession; auction procedures | Automated SARFAESI workflow with compliance checkpoints |
| Recovery of Debts Act, 1993 (DRT) | Legal (BC-08) | DRT filing for ≥₹20 lakh | Case management with threshold validation |
| Limitation Act, 1963 | Legal (BC-08) | 3-year limitation for money recovery suits | Limitation tracking with auto-alerts |
| TRAI Regulations (DND, DLT) | Communication (BC-04), Compliance (BC-10) | DND registry check; DLT-registered SMS templates | Pre-send DND check; DLT template management |
| IT Act, 2000 & Rules | All contexts | Data privacy; consent; data localization | Encryption, access control, audit logging, data residency |
| IND AS 109 | Analytics (BC-11), Delinquency (BC-02) | ECL provisioning; staging (1/2/3) | ECL computation engine; staging alignment |
| Negotiable Instruments Act, Section 138 | Legal (BC-08) | Cheque bounce prosecution (₹1 lakh threshold removed) | Section 138 case workflow |
| CERSAI Regulations | Legal (BC-08) | Charge registration and satisfaction | CERSAI integration for charge satisfaction on settlement/closure |
| PMLA (Prevention of Money Laundering Act) | Reconciliation (BC-01), Compliance (BC-10) | Suspicious transaction reporting for collections >₹10 lakh | AML flag on large/unusual collections |
| RBI Guidelines on Wilful Defaulters (Jul 2024) | Delinquency (BC-02), Legal (BC-08) | Identification criteria; reporting; penal provisions | Auto-identification with committee review workflow |
| RBI Framework for Resolution of Stressed Assets (Jun 2019) | Delinquency (BC-02) | 30-day review period; ICA for ≥₹100 Cr exposures; resolution plan implementation | Restructuring workflow; ICA tracking |
| NHB Directions (for HFCs) | All contexts | Similar to RBI IRAC with HFC-specific modifications | Configurable per entity type |

---

## 23. ASSUMPTIONS, RISKS, AND DEPENDENCIES

### 23.1 Key Assumptions

| # | Assumption | Impact if Invalid |
|---|---|---|
| A1 | Core LMS/CBS will provide real-time loan data, EMI schedule, and customer data via APIs | LCMS data accuracy compromised; DPD calculation errors |
| A2 | NPCI NACH response files will be available 4x daily as per current schedule | Reconciliation delays; DPD stale for part of day |
| A3 | Institution has existing NACH sponsorship and UPI AutoPay integration | Mandate management module limited in scope |
| A4 | Cloud or on-premise infrastructure meets specified NFR requirements | Performance and availability targets missed |
| A5 | Regulatory norms current as of March 2026 will remain broadly stable during implementation | Mid-implementation regulatory changes require scope change |
| A6 | AI/ML capabilities (speech-to-text, NLU, sentiment analysis) are available via third-party or in-house platforms | Sentiment and AI conversation features delayed |
| A7 | Collection agencies will adopt digital portal (may resist change from spreadsheet-based) | Agency management benefits reduced; parallel manual processes needed |

### 23.2 Key Risks

| # | Risk | Probability | Impact | Mitigation |
|---|---|---|---|---|
| R1 | Integration complexity with legacy CBS | High | High | Anti-Corruption Layer (ACL) design; phased integration; adapter pattern |
| R2 | Data quality issues in LMS/CBS data | High | High | Data quality checks at ingestion; reconciliation breaks for data errors; data cleansing sprint |
| R3 | AI model accuracy below target in Indian language sentiment analysis | Medium | Medium | Start with Hindi/English; partner with Indian language NLP providers; continuous model improvement |
| R4 | Regulatory changes during implementation | Medium | Medium | Modular compliance engine; configurable rules; regulatory change management process |
| R5 | Change resistance from collection teams | Medium | High | Change management program; training; phased rollout; champion users |
| R6 | Agency data security breach | Low | Critical | Data isolation; access controls; contractual penalties; regular audits |
| R7 | Performance degradation at scale | Medium | High | Load testing at 2x projected volume; horizontal scaling design; performance monitoring |
| R8 | AI chatbot/voice agent compliance violation | Low | Critical | Hardcoded compliance guardrails; 100% recording; regular audit; human escalation |

### 23.3 Key Dependencies

| # | Dependency | Owner | Impact of Delay |
|---|---|---|---|
| D1 | LMS/CBS API availability | Core Banking Team | Blocks all contexts; LCMS cannot function without loan data |
| D2 | NACH integration setup | Treasury/Payments | Blocks reconciliation for NACH rail |
| D3 | WhatsApp Business API approval | Digital Team | WhatsApp channel unavailable |
| D4 | IVR/Telephony platform procurement | IT Infra | IVR and AI Voice features blocked |
| D5 | ML model training data availability | Data Engineering | EWS and propensity models delayed |
| D6 | Agency digital onboarding | Agency Management | Agency portal benefits delayed |
| D7 | Legal template approval | Legal Department | Legal notice generation blocked |

---

## 24. FULL GLOSSARY WITH UBIQUITOUS LANGUAGE

| Term | Full Form | Definition | Used In Context(s) |
|---|---|---|---|
| ACH | Automated Clearing House | Electronic payment network for batch processing | BC-01 |
| ACL | Anti-Corruption Layer | DDD pattern to isolate external system models | All |
| AML | Anti-Money Laundering | Regulations to prevent money laundering | BC-01, BC-10 |
| BBPS | Bharat Bill Payment System | NPCI's integrated bill payment system | BC-01, BC-09 |
| BSP | Business Solution Provider | WhatsApp Business API provider | BC-04 |
| CAG | Comptroller and Auditor General | Government audit authority (relevant for PSU banks) | BC-08 |
| CBS | Core Banking System | Primary banking platform | External |
| CDC | Change Data Capture | Pattern for streaming database changes | BC-11 |
| CER | Collection Efficiency Ratio | Collections KPI: collected / demand | BC-11 |
| CERSAI | Central Registry of Securitisation Asset Reconstruction and Security Interest | Central registry for security interests | BC-08 |
| CIBIL | Credit Information Bureau (India) Limited | Credit bureau (now TransUnion CIBIL) | BC-05, BC-10 |
| CRILC | Central Repository of Information on Large Credits | RBI's large credit database | BC-02, BC-10 |
| CTS | Cheque Truncation System | Electronic cheque clearing system | BC-01 |
| DLR | Delivery Level Report | SMS delivery status report | BC-04 |
| DLT | Distributed Ledger Technology | TRAI's SMS template registration platform | BC-04, BC-10 |
| DND | Do Not Disturb | TRAI registry for blocking unsolicited calls/SMS | BC-04, BC-10 |
| DOA | Delegation of Authority | Approval hierarchy matrix | BC-08 |
| DPD | Days Past Due | Calendar days since oldest unpaid installment due date | BC-02 |
| DRT | Debt Recovery Tribunal | Tribunal for recovery of debts ≥₹20 lakh | BC-08 |
| DTMF | Dual-Tone Multi-Frequency | Telephone keypad input signaling | BC-04 |
| EBLR | External Benchmark Linked Rate | Floating rate linked to RBI repo rate | BC-06 |
| ECL | Expected Credit Loss | Forward-looking credit loss estimate (IND AS 109) | BC-02, BC-11 |
| ECS | Electronic Clearing Service | Predecessor to NACH for recurring debits | BC-01 |
| EMI | Equated Monthly Installment | Fixed monthly loan payment | All |
| EWS | Early Warning System | Pre-delinquency signal detection | BC-05 |
| FLDG | First Loss Default Guarantee | Guarantee in co-lending/LSP models | BC-01 |
| FPC | Fair Practices Code | RBI code for lending/collection practices | BC-10 |
| GL | General Ledger | Primary accounting record | BC-01 |
| GPS | Global Positioning System | Location tracking for field visits | BC-07 |
| GST | Goods and Services Tax | Indirect tax (relevant for business loan EWS) | BC-05 |
| HFC | Housing Finance Company | Company providing home loans (NHB regulated) | All |
| HSM | Hardware Security Module | Cryptographic key management device | BC-10 |
| IBC | Insolvency and Bankruptcy Code | Law for insolvency resolution | BC-08 |
| IFSC | Indian Financial System Code | Bank branch identifier | BC-06 |
| IND AS | Indian Accounting Standards | Accounting standards (converged with IFRS) | BC-02, BC-11 |
| IRAC | Income Recognition and Asset Classification | RBI's NPA classification norms | BC-02 |
| IVR | Interactive Voice Response | Automated phone system with voice/DTMF input | BC-04 |
| KYC | Know Your Customer | Customer identity verification | BC-06 |
| LCMS | Loan Collection Management System | This system | All |
| LMS | Loan Management System | Core lending platform (parent system) | External |
| LSP | Lending Service Provider | Digital lending intermediary | BC-01, BC-09 |
| MCLR | Marginal Cost of Funds Based Lending Rate | Bank's internal benchmark rate | BC-06 |
| MIS | Management Information System | Reporting and analytics | BC-11 |
| mTLS | Mutual TLS | Two-way TLS authentication | NFR |
| NACH | National Automated Clearing House | NPCI's electronic payment system | BC-01, BC-06 |
| NBFC | Non-Banking Financial Company | RBI-regulated non-bank lender | All |
| NCPR | National Customer Preference Register | TRAI's DND registry | BC-04 |
| NHB | National Housing Bank | Regulator for HFCs | All |
| NLU | Natural Language Understanding | AI capability for interpreting human language | BC-04, BC-05 |
| NPA | Non-Performing Asset | Loan classified as non-performing (90+ DPD) | BC-02 |
| NPCI | National Payments Corporation of India | Payment infrastructure organization | BC-01, BC-06, BC-09 |
| OTS | One-Time Settlement | Negotiated settlement to close defaulted loan | BC-08 |
| PII | Personally Identifiable Information | Sensitive personal data | BC-10 |
| PMLA | Prevention of Money Laundering Act | Anti-money laundering legislation | BC-10 |
| PSU | Public Sector Undertaking | Government-owned entity | All |
| PTP | Promise to Pay | Customer's commitment to pay by specific date | BC-02, BC-07 |
| QR | Quick Response (Code) | 2D barcode for UPI payment | BC-09 |
| RBAC | Role-Based Access Control | Access control mechanism | BC-10 |
| RBI | Reserve Bank of India | Central bank and primary regulator | All |
| RPO | Recovery Point Objective | Maximum data loss tolerance in disaster | NFR |
| RTO | Recovery Time Objective | Maximum downtime tolerance in disaster | NFR |
| SARFAESI | Securitisation and Reconstruction of Financial Assets and Enforcement of Security Interest Act | Secured creditor recovery law | BC-08 |
| SBR | Scale-Based Regulation | RBI's tiered NBFC regulation framework | All |
| SFB | Small Finance Bank | RBI-licensed bank for financial inclusion | All |
| SI | Standing Instruction | Automatic transfer order at bank | BC-01, BC-06 |
| SMA | Special Mention Account | Pre-NPA classification (SMA-0/1/2) | BC-02 |
| SOA | Statement of Account | Detailed account transaction statement | BC-06 |
| STP | Straight-Through Processing | Fully automated processing without manual intervention | BC-01 |
| TDS | Tax Deducted at Source | Income tax deduction | BC-06 |
| TRAI | Telecom Regulatory Authority of India | Telecom regulator | BC-04, BC-10 |
| TSP | Traveling Salesman Problem | Route optimization algorithm | BC-07 |
| TTS | Text-to-Speech | Converting text to spoken audio | BC-04 |
| UCB | Urban Cooperative Bank | Cooperative bank in urban areas | All |
| UMRN | Unique Mandate Reference Number | NPCI's unique ID for NACH mandates | BC-01, BC-06 |
| UPI | Unified Payments Interface | NPCI's real-time payment system | BC-01, BC-09 |
| UTR | Unique Transaction Reference | Unique ID for NEFT/RTGS transactions | BC-01 |
| VA | Virtual Account | Virtual account number mapped to loan for payment identification | BC-01 |
| VAPT | Vulnerability Assessment and Penetration Testing | Security testing | NFR |
| WAL | Write-Ahead Log | Database durability mechanism | NFR |
| XBRL | eXtensible Business Reporting Language | Standardized financial reporting format | BC-10 |

---

## 25. ACCEPTANCE CRITERIA — SYSTEM LEVEL

| # | Acceptance Criterion | Validation Method |
|---|---|---|
| AC-01 | DPD calculated correctly for 100% of accounts (validated against manual calculation for 1,000 sample accounts across products) | UAT + parallel run |
| AC-02 | Auto-match rate ≥97% for NACH; ≥90% overall across all payment rails | Production metrics after 30-day stabilization |
| AC-03 | Zero suspense entries older than 90 days without management escalation | Monthly audit |
| AC-04 | All communication respects timing rules (8 AM - 7 PM) with zero violations | Compliance audit of 100% communications |
| AC-05 | Customer 360 loads in <3 seconds with complete data from all contexts | Load testing at 2x projected concurrent users |
| AC-06 | AI chatbot handles 80%+ of routine conversations without human escalation | Production metrics after 60-day stabilization |
| AC-07 | EWS detects ≥70% of future NPAs at least 30 days before NPA date | Back-testing on 12 months historical data |
| AC-08 | CRILC reports generated accurately and on time (weekly SMA, quarterly full) | Parallel run with existing system for 2 quarters |
| AC-09 | Complete audit trail for every action; immutable; retained for 8 years | Audit review |
| AC-10 | SARFAESI workflow enforces 60-day notice period with zero violations | UAT + legal team review |
| AC-11 | Agency data isolation verified — no cross-agency data leakage | Security testing (VAPT) |
| AC-12 | System handles 10 million accounts with all NFR targets met | Load and performance testing |

---

*This document is confidential and proprietary. Unauthorized reproduction or distribution is prohibited.*

*All regulatory references are as of March 2026. Users are advised to verify current regulatory applicability with the Compliance team before implementation.*

*Architecture approach: Domain-Driven Design with 11 Bounded Contexts, Event-Driven Architecture, API-First Design.*

---

## END OF PART 09 — FINAL PART

### Document Complete

| Part | File | Content |
|---|---|---|
| Part 01 | COL-BRD-Part01-DocControl-ContextMap.md | Document control, executive summary, DDD context map, ubiquitous language |
| Part 02 | COL-BRD-Part02-Reconciliation.md | Reconciliation bounded context (BC-01) |
| Part 03 | COL-BRD-Part03-Delinquency.md | Delinquency bounded context (BC-02) — DPD, buckets, NPA |
| Part 04 | COL-BRD-Part04-Workflow-Communication.md | Workflow (BC-03) + Communication (BC-04) — notifications, agentic conversations |
| Part 05 | COL-BRD-Part05-Sentiment-Intelligence.md | Sentiment & Intelligence (BC-05) — sentiment analysis, propensity models, EWS |
| Part 06 | COL-BRD-Part06-Mandate-Account360.md | Mandate & Account (BC-06) — mandate management, customer 360 |
| Part 07 | COL-BRD-Part07-Agency-Field-Legal.md | Agency & Field (BC-07) + Legal & Recovery (BC-08) |
| Part 08 | COL-BRD-Part08-Payment-Compliance-Analytics.md | Payment (BC-09) + Compliance (BC-10) + Analytics (BC-11) |
| Part 09 | COL-BRD-Part09-NFR-DataModel-Glossary.md | NFR, data model, API contracts, compliance matrix, glossary |
