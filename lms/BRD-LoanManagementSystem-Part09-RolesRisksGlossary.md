# USER ROLES & ACCESS CONTROL
# ASSUMPTIONS, RISKS & DEPENDENCIES
# GLOSSARY
# APPENDICES

---

## 27. USER ROLES AND ACCESS CONTROL

### 27.1 Role-Based Access Matrix

**Requirement ID: LMS-UAC-001**
**Title:** Role-Based Access Control Matrix
**Priority:** Must Have

**User Roles:**

| Role | Description | Access Level |
|---|---|---|
| Branch Officer | Loan booking, customer service, payment receipt, document management | Branch-level; own branch data only |
| Branch Manager | Branch-level approvals, NPA review, collection oversight | Branch-level; approval authority |
| Regional Manager | Regional portfolio review, escalation handling, higher approval authority | Region-level (multiple branches) |
| Credit Officer | Credit appraisal, underwriting, sanction recommendation | Cross-branch (assigned cases) |
| Credit Approver | Loan sanction approval, deviation approval | Based on delegation of authority |
| Operations Officer | NACH processing, reconciliation, disbursement, closure | Centralized operations |
| Operations Manager | Operations approval, suspense clearing, exception handling | Centralized operations; approval |
| Collections Officer | Overdue follow-up, PTP recording, field visit allocation | Assigned portfolio |
| Collections Manager | Collection strategy, agency allocation, OTS recommendation | Portfolio oversight |
| Legal Officer | SARFAESI processing, legal case management, auction management | Legal module access |
| Finance Officer | GL reconciliation, provision computation, regulatory returns preparation | Finance module; cross-entity |
| Finance Manager | GL closure approval, provision approval, return sign-off | Finance approval authority |
| Compliance Officer | Regulatory compliance monitoring, SMA/NPA oversight, audit support | Read access across all modules |
| Risk Manager | Portfolio risk analysis, ECL computation oversight, stress testing | Analytics; risk module |
| Product Manager | Product configuration, pricing, policy updates | Product master; configuration |
| System Administrator | User management, role assignment, system configuration | Admin module; no financial transactions |
| Audit User | Read-only access for internal/external/statutory/RBI audit | Read-only across all modules and audit trail |
| Customer (Self-Service) | View own loan details, statements, certificates; make payments | Own account data only |
| LSP/Partner (API) | Submit applications, receive status updates | API-only; specific endpoints |

### 27.2 Access Control Rules

**Requirement ID: LMS-UAC-002**
**Title:** Access Control Policies
**Priority:** Must Have

**Business Rules:**
- Principle of least privilege: users granted minimum access needed for their function
- Segregation of duties: maker-checker enforced; credit sanctioner cannot also be disbursement approver for same loan
- Geographic restriction: branch users access only their branch data; regional users access their region
- Product restriction: users can be restricted to specific product types
- Amount-based delegation: approval authority linked to loan amount thresholds (defined in Delegation of Authority matrix)
- Temporary access: for audit/review purposes, temporary elevated access with expiry date
- Inactive user lockout: account locked after 90 days of inactivity; disabled after 180 days
- Failed login lockout: account locked after 5 consecutive failed attempts; unlock by admin
- IP-based restriction: optional IP whitelisting for sensitive roles (admin, finance)
- Time-based restriction: system access can be restricted to business hours for specific roles

### 27.3 Delegation of Authority (DOA) Matrix

**Requirement ID: LMS-UAC-003**
**Title:** Financial Delegation of Authority
**Priority:** Must Have

**Loan Sanction DOA (example, configurable):**

| Authority Level | Loan Amount Threshold | Product Restrictions |
|---|---|---|
| Branch Manager | Up to Rs 25 Lakh | Retail products only |
| Regional Manager | Up to Rs 1 Crore | Retail + MSME |
| AGM / DGM | Up to Rs 5 Crore | All products |
| General Manager | Up to Rs 25 Crore | All products |
| ED / DMD | Up to Rs 100 Crore | All products |
| MD & CEO | Up to Rs 250 Crore | All products |
| Board / MC Committee | Above Rs 250 Crore | All products |

**Other DOA Items:**
- Write-off authority (see LMS-OVD-007)
- OTS authority (see LMS-OVD-008)
- Interest concession authority
- Fee/charge waiver authority
- NPA classification override authority
- Collateral release authority
- Provision override authority

---

## 28. ASSUMPTIONS, RISKS, AND DEPENDENCIES

### 28.1 Assumptions

| ID | Assumption | Impact if Invalid |
|---|---|---|
| A01 | Core Banking System (CBS) will be available for real-time API integration | GL posting, disbursement, and payment flows will need batch mode fallback |
| A02 | Existing loan portfolio data is clean and will be migrated per agreed data mapping | Data quality issues will delay go-live; reconciliation breaks post-migration |
| A03 | Credit bureaus (CIBIL, Experian, CRIF, Equifax) will provide API access | Bureau checks will be manual; digital lending workflow impacted |
| A04 | NACH sponsor bank will support e-mandate and NACH API integration | Manual NACH file processing; increased operational effort |
| A05 | Institution has documented and Board-approved policies for: NPA management, write-off, OTS, DOA, collection, provisioning | System cannot be configured without finalized policies |
| A06 | RBI regulatory norms considered are as of March 2026; any subsequent changes will be handled via change requests | New circulars may require system modifications post-go-live |
| A07 | Hardware/infrastructure procurement (servers, network, DC/DR) will be completed as per project plan | Infrastructure delays will impact testing and go-live |
| A08 | IND AS 109 PD/LGD models will be provided by the institution's risk team | ECL computation module cannot be tested without calibrated models |
| A09 | Empaneled valuers, advocates, and insurance companies will be onboarded before go-live | Origination and collateral management workflows will be incomplete |
| A10 | Existing staff will be trained on the new system before go-live; dedicated training period of minimum 4 weeks | User adoption issues; operational errors; increased support tickets |

### 28.2 Risks

| ID | Risk | Probability | Impact | Mitigation |
|---|---|---|---|---|
| R01 | Data migration from legacy system(s) causes data quality issues and reconciliation breaks | High | High | Comprehensive data profiling; iterative trial migrations; parallel run period of minimum 3 months |
| R02 | Regulatory changes post-design freeze require significant rework | Medium | High | Modular design with configurable rules; regulatory change management process; compliance team involvement throughout |
| R03 | Integration with CBS/external systems delays due to API readiness | High | High | Early integration testing; mock services for development; parallel workstreams for integration |
| R04 | EOD batch processing exceeds window due to data volume | Medium | High | Performance testing with production-scale data; parallel processing; batch optimization; SLA monitoring |
| R05 | Credit bureau API latency causes user experience degradation | Medium | Medium | Asynchronous bureau calls; caching of results; SLA monitoring with bureau providers |
| R06 | Cybersecurity vulnerabilities discovered during VAPT | Medium | High | Security-by-design; SAST/DAST in CI/CD; mandatory VAPT before UAT; remediation sprint |
| R07 | User resistance to new system affecting adoption | Medium | Medium | Change management program; training; champion user network; phased rollout |
| R08 | IND AS 109 ECL model calibration inadequate for production | Medium | High | Extensive backtesting; parallel run with existing ECL; independent validation |
| R09 | NACH/payment system integration failures during initial production | Medium | High | Pilot with limited accounts; fallback manual process; dedicated support team for first 3 months |
| R10 | Multi-entity configuration complexity leads to incorrect regulatory treatment | Low | High | Entity-specific configuration review by compliance; dedicated UAT cycle per entity |

### 28.3 Dependencies

| ID | Dependency | Owner | Required By |
|---|---|---|---|
| D01 | CBS API specifications and sandbox environment | CBS Vendor / IT | Design phase |
| D02 | Credit bureau API credentials and test environment | Bureau relationship team | Development phase |
| D03 | CERSAI API registration and test credentials | Legal / Operations | Development phase |
| D04 | NACH sponsor bank onboarding and test environment | Treasury / Ops | Integration testing |
| D05 | Board-approved policies (NPA, write-off, DOA, collection, provisioning) | Policy / Compliance | Configuration phase |
| D06 | IND AS 109 PD/LGD/EAD models | Risk Analytics team | UAT phase |
| D07 | Production infrastructure (DC/DR servers, network, certificates) | IT Infrastructure | Performance testing |
| D08 | Product configuration parameters for all loan products | Product teams | UAT phase |
| D09 | GL Chart of Accounts finalization | Finance team | Configuration phase |
| D10 | Legacy data extraction and mapping document | Legacy system team | Data migration |
| D11 | Regulatory return formats and submission portal access | Compliance team | UAT phase |
| D12 | SMS/Email/WhatsApp gateway setup | Vendor management | Integration testing |
| D13 | Digital certificate for eSign/estamp integration | IT Security | Development phase |
| D14 | Insurance company API integration credentials | Insurance partnerships | Integration testing |
| D15 | Account Aggregator FIU registration and test environment | AA licensing team | Development phase |

---

## 29. GLOSSARY

| Term | Full Form / Definition |
|---|---|
| AA | Account Aggregator -- RBI-licensed entity enabling consent-based financial data sharing |
| ACH | Automated Clearing House |
| AD | Acknowledgment Due (registered postal mail requiring delivery confirmation) |
| AML | Anti-Money Laundering |
| ANBC | Adjusted Net Bank Credit -- base for PSL target computation |
| APR | Annual Percentage Rate -- all-inclusive cost of borrowing expressed as annualized rate |
| BL | Base Layer (NBFC Scale-Based Regulation) |
| BPLR | Benchmark Prime Lending Rate (legacy interest rate benchmark) |
| BSR | Basic Statistical Return (RBI regulatory return) |
| CBS | Core Banking System |
| CCIL | Clearing Corporation of India Limited |
| CDC | Change Data Capture |
| CERSAI | Central Registry of Securitisation Asset Reconstruction and Security Interest of India |
| CIBIL | Credit Information Bureau (India) Limited (now TransUnion CIBIL) |
| CIN | Corporate Identification Number |
| CKYC | Central KYC -- centralized KYC records registry operated by CERSAI |
| CLM | Co-Lending Model |
| CoA | Chart of Accounts |
| CPC | Centralised Processing Centre |
| CRIF | CRIF High Mark (credit bureau) |
| CRILC | Central Repository of Information on Large Credits |
| CRR | Cash Reserve Ratio |
| CTS | Cheque Truncation System |
| D1/D2/D3 | Doubtful Asset categories (1, 2, 3 based on period in NPA) |
| DBIE | Database on Indian Economy (RBI) |
| DCCO | Date of Commencement of Commercial Operations |
| DL | Digital Lending |
| DLF | Digital Lending App (First Loss Default Guarantee provider) |
| DND | Do Not Disturb (TRAI registry) |
| DOA | Delegation of Authority |
| DPD | Days Past Due |
| DPDP | Digital Personal Data Protection (Act, 2023) |
| DR | Disaster Recovery |
| DRT | Debt Recovery Tribunal |
| EBLR | External Benchmark Lending Rate |
| ECB | External Commercial Borrowing |
| ECL | Expected Credit Loss (IND AS 109) |
| ECS | Electronic Clearing Service (legacy; replaced by NACH) |
| ED | Executive Director |
| EIR | Effective Interest Rate (IND AS 109) |
| EMI | Equated Monthly Installment |
| EOD | End of Day (batch processing cycle) |
| FBIL | Financial Benchmarks India Private Limited |
| FCNR | Foreign Currency Non-Resident (deposit/loan) |
| FEDAI | Foreign Exchange Dealers Association of India |
| FEMA | Foreign Exchange Management Act |
| FITL | Funded Interest Term Loan |
| FIU | Financial Information User (in Account Aggregator ecosystem) |
| FLDG | First Loss Default Guarantee |
| FOIR | Fixed Obligations to Income Ratio |
| FTP | Fund Transfer Pricing |
| GL | General Ledger |
| GST | Goods and Services Tax |
| GSTN | Goods and Services Tax Network |
| HFC | Housing Finance Company |
| HSM | Hardware Security Module |
| HUDCO | Housing and Urban Development Corporation |
| ICA | Inter-Creditor Agreement |
| IDV | Insured Declared Value (vehicle insurance) |
| IGMS | Integrated Grievance Management System (RBI) |
| IMPS | Immediate Payment Service |
| IND AS | Indian Accounting Standards (converged with IFRS) |
| IoI | Interest on Interest |
| IRAC | Income Recognition and Asset Classification |
| IRDAI | Insurance Regulatory and Development Authority of India |
| IVR | Interactive Voice Response |
| JLG | Joint Liability Group |
| KCC | Kisan Credit Card |
| KFS | Key Fact Statement |
| KUA | KYC User Agency (Aadhaar ecosystem) |
| KYC | Know Your Customer |
| LAP | Loan Against Property |
| LBS | Locational Banking Statistics |
| LGD | Loss Given Default (ECL parameter) |
| LMS | Loan Management System |
| LOS | Loan Origination System |
| LSP | Lending Service Provider (digital lending intermediary) |
| LTV | Loan-to-Value ratio |
| MCLR | Marginal Cost of Funds Based Lending Rate |
| MDR | Merchant Discount Rate |
| MFI | Microfinance Institution |
| MIS | Management Information System |
| MITC | Most Important Terms and Conditions |
| ML | Middle Layer (NBFC Scale-Based Regulation) |
| MSME | Micro, Small and Medium Enterprises |
| MT940 | SWIFT message format for bank statement |
| NABARD | National Bank for Agriculture and Rural Development |
| NACH | National Automated Clearing House (NPCI) |
| NBFC | Non-Banking Financial Company |
| NBFC-BL | NBFC Base Layer |
| NBFC-ML | NBFC Middle Layer |
| NBFC-UL | NBFC Upper Layer |
| NEFT | National Electronic Funds Transfer |
| NHB | National Housing Bank |
| NOC | No Objection Certificate |
| NPA | Non-Performing Asset |
| NPCI | National Payments Corporation of India |
| NTP | Network Time Protocol |
| OTS | One-Time Settlement / Compromise Settlement |
| OTR | One-Time Restructuring |
| PAN | Permanent Account Number (Income Tax) |
| PAM | Privileged Access Management |
| PC | Profit Center |
| PD | Probability of Default (ECL parameter) |
| PDC | Post-Dated Cheque |
| PEMI | Pre-EMI Interest |
| PGP | Pretty Good Privacy (encryption for file exchange) |
| PMAY | Pradhan Mantri Awas Yojana |
| PMFBY | Pradhan Mantri Fasal Bima Yojana |
| PSL | Priority Sector Lending |
| PSLC | Priority Sector Lending Certificate |
| PSP | Payment Service Provider |
| PTP | Promise to Pay |
| RC | Registration Certificate (vehicle) |
| RBI | Reserve Bank of India |
| RE | Regulated Entity (RBI-regulated bank/NBFC) |
| RFA | Red Flagged Account |
| RIDF | Rural Infrastructure Development Fund |
| RLLR | Repo Linked Lending Rate |
| RPO | Recovery Point Objective |
| RRB | Regional Rural Bank |
| RTGS | Real Time Gross Settlement |
| RTO | (1) Recovery Time Objective; (2) Regional Transport Office |
| SARFAESI | Securitisation and Reconstruction of Financial Assets and Enforcement of Security Interest Act, 2002 |
| SBR | Scale-Based Regulation (for NBFCs) |
| SCB | Scheduled Commercial Bank |
| SEBI | Securities and Exchange Board of India |
| SFB | Small Finance Bank |
| SFTP | Secure File Transfer Protocol |
| SHG | Self Help Group |
| SIEM | Security Information and Event Management |
| SL | Sub-Ledger |
| SLR | Statutory Liquidity Ratio |
| SMA | Special Mention Account |
| SOA | Statement of Account |
| TDE | Transparent Data Encryption |
| TDS | Tax Deducted at Source |
| TRAI | Telecom Regulatory Authority of India |
| TUDF | TransUnion Data Format (CIBIL reporting format) |
| UAT | User Acceptance Testing |
| UCB | Urban Cooperative Bank |
| UIDAI | Unique Identification Authority of India |
| UL | Upper Layer (NBFC Scale-Based Regulation) |
| UMRN | Unique Mandate Reference Number (NACH) |
| UPI | Unified Payments Interface |
| UTR | Unique Transaction Reference |
| V-CIP | Video-based Customer Identification Process |
| VAPT | Vulnerability Assessment and Penetration Testing |
| WAF | Web Application Firewall |
| XBRL | eXtensible Business Reporting Language |
| XIRR | Extended Internal Rate of Return |

---

## 30. APPENDICES

### Appendix A: Sample EMI Calculation Comparison

**Loan Parameters:** Rs 10,00,000 | 12% p.a. | 5 years (60 months)

| Method | Monthly Payment | Total Interest | Total Payment |
|---|---|---|---|
| Reducing Balance (EMI) | Rs 22,244 | Rs 3,34,667 | Rs 13,34,667 |
| Flat Rate | Rs 26,667 | Rs 6,00,000 | Rs 16,00,000 |
| Rule of 78 (Flat Rate with front-loaded interest) | Rs 26,667 (same as flat, but interest distribution differs) | Rs 6,00,000 | Rs 16,00,000 |

**Effective Rate Comparison:**
- Reducing Balance: 12.00% (stated = effective)
- Flat Rate 12%: Effective reducing balance equivalent approximately 21.2%
- This demonstrates why APR disclosure is critical per RBI Digital Lending Guidelines

### Appendix B: NPA Provisioning Calculation Example

**Scenario:** Home Loan account classified as Doubtful D1

| Parameter | Value |
|---|---|
| Outstanding Principal | Rs 45,00,000 |
| Collateral (Property) Market Value | Rs 60,00,000 |
| Distress Value Factor | 75% |
| Realizable Value of Security | Rs 45,00,000 (60L x 75%) |
| Secured Portion | Rs 45,00,000 |
| Unsecured Portion | Rs 0 (Outstanding <= Realizable) |
| Provision (D1 -- 25% of Secured + 100% of Unsecured) | Rs 11,25,000 |

**If Outstanding were Rs 50,00,000:**
- Unsecured Portion = Rs 50,00,000 - Rs 45,00,000 = Rs 5,00,000
- Provision = (Rs 45,00,000 x 25%) + (Rs 5,00,000 x 100%) = Rs 11,25,000 + Rs 5,00,000 = Rs 16,25,000

### Appendix C: ECL Computation Example (IND AS 109)

**Stage 1 -- 12-Month ECL:**

| Parameter | Value | Source |
|---|---|---|
| Exposure at Default (EAD) | Rs 45,00,000 | Outstanding + committed undrawn |
| 12-Month PD | 1.5% | PD model (TTC adjusted to PIT) |
| LGD | 25% | Historical loss data; collateral recovery |
| 12-Month ECL | Rs 45,00,000 x 1.5% x 25% = Rs 1,68,750 | PD x LGD x EAD |

**Stage 2 -- Lifetime ECL (same account, migrated to Stage 2):**

| Parameter | Value |
|---|---|
| EAD | Rs 45,00,000 |
| Lifetime PD (cumulative over remaining tenure of 15 years) | 8.5% |
| LGD | 25% |
| Lifetime ECL (simplified) | Rs 45,00,000 x 8.5% x 25% = Rs 9,56,250 |

Note: Actual lifetime ECL computation involves marginal PD term structure, discounting at EIR, and probability-weighted macroeconomic scenarios. The above is simplified for illustration.

### Appendix D: NACH File Format Reference

**Presentation File (Debit Instruction):**

| Field | Position | Length | Description |
|---|---|---|---|
| Record Type | 1-2 | 2 | "DN" (Debit New) |
| Destination Bank IFSC | 3-13 | 11 | Customer's bank IFSC |
| Beneficiary Account No | 14-28 | 15 | Customer's bank account number |
| Transaction Amount | 29-41 | 13 | Amount (in paise) |
| UMRN | 42-61 | 20 | Unique Mandate Reference Number |
| Transaction Reference | 62-81 | 20 | LMS transaction reference |
| Utility Code | 82-99 | 18 | Lender's NACH utility code |
| Purpose | 100-109 | 10 | "LOAN EMI" |

(Actual format as per NPCI NACH procedural guidelines; the above is representative.)

### Appendix E: Key RBI Circular References

| Circular | Date | Subject |
|---|---|---|
| DOR.STR.REC.4/21.04.048/2021-22 | Nov 12, 2021 | Master Direction -- IRAC Norms (updated) |
| DOR.CRE.REC.66/21.07.001/2022-23 | Sep 2, 2022 | Digital Lending Guidelines |
| DOR.FIN.REC.No.20/03.10.038/2023-24 | Jun 8, 2023 | FLDG Guidelines |
| DoR.MCS.REC.28/01.01.001/2023-24 | Aug 18, 2023 | Fair Lending Practice -- Penal Charges |
| DOR.CRE.REC.60/03.10.001/2021-22 | Oct 22, 2021 | Scale-Based Regulation for NBFCs |
| DOR.AML.REC.10/14.01.001/2023-24 | Updated | KYC Master Direction |
| DBOD.No.Dir.BC.67/13.03.00/2018-19 | Sep 4, 2019 | External Benchmark Linked Lending |
| DOR.STR.REC.21/21.04.048/2018-19 | Jun 7, 2019 | Framework for Resolution of Stressed Assets |
| DNBS.CC.PD.No.266/03.10.01/2012-13 | 2012 | Prepayment of Loans by NBFCs -- No Penalty on Floating Rate |
| CERSAI Act & Rules | 2011 | Central Registry -- Security Interest Registration |
| SARFAESI Act | 2002 | Securitisation and Enforcement of Security Interest |
| IT Act -- Section 194A | -- | TDS on Interest |
| IT Act -- Section 80C, 24(b), 80E | -- | Tax deductions for loan repayment |
| DPDP Act | 2023 | Digital Personal Data Protection |
| CERT-In Directions | Apr 28, 2022 | Incident reporting; log retention |

---

## 31. DOCUMENT APPROVAL

| Section | Reviewed By | Review Status | Comments |
|---|---|---|---|
| GL Integration | Head -- Finance | Pending | -- |
| Payment Processing | Head -- Operations | Pending | -- |
| Reconciliation | Head -- Operations | Pending | -- |
| Interest Computation | Head -- Finance, Head -- Product | Pending | -- |
| Overdue Management | Head -- Collections, CRO | Pending | -- |
| Early Closure | Head -- Operations | Pending | -- |
| Interest-Principal Allocation | Head -- Finance | Pending | -- |
| Origination | Head -- Credit, Head -- Retail | Pending | -- |
| Collateral Management | Head -- Legal, Head -- Operations | Pending | -- |
| Insurance | Head -- Operations, Head -- Partnerships | Pending | -- |
| Regulatory Reporting | CCO, Head -- Compliance | Pending | -- |
| Bureau Integration | Head -- Risk, Head -- IT | Pending | -- |
| Digital Lending | CCO, Head -- Digital | Pending | -- |
| Customer Communication | Head -- Customer Experience | Pending | -- |
| Audit & Maker-Checker | Head -- Internal Audit | Pending | -- |
| Architecture | CTO, Head -- Enterprise Architecture | Pending | -- |
| Security & Privacy | CISO | Pending | -- |
| DR/BCP | CTO, Head -- IT Operations | Pending | -- |
| NFR | CTO, Head -- QA | Pending | -- |
| Data Model | Head -- Data Engineering, DBA Lead | Pending | -- |

**Final Approval:**

| Approver | Designation | Signature | Date |
|---|---|---|---|
| [TBD] | Chief Technology Officer | __________ | __________ |
| [TBD] | Chief Risk Officer | __________ | __________ |
| [TBD] | Chief Compliance Officer | __________ | __________ |
| [TBD] | Head -- Retail/MSME Lending | __________ | __________ |
| [TBD] | Managing Director & CEO | __________ | __________ |

---

**END OF DOCUMENT**

---

*This document is confidential and proprietary. Unauthorized reproduction or distribution is prohibited. All regulatory references are as of March 2026. Users are advised to verify current regulatory applicability with the Compliance team before implementation.*

---
