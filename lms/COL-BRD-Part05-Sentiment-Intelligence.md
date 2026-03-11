# PART 05 -- SENTIMENT & INTELLIGENCE BOUNDED CONTEXT (BC-05)

---

## 11. SENTIMENT & INTELLIGENCE CONTEXT

### 11.1 Context Overview

| Attribute | Details |
|---|---|
| Context Name | Sentiment & Intelligence Context |
| Context ID | BC-05 |
| Domain Classification | Core Domain |
| Owner Team | Data Science & Analytics |
| Upstream Dependencies | Communication (BC-04) — ConversationCompleted, CustomerResponded; Delinquency (BC-02) — DPD/bucket data; Reconciliation (BC-01) — Payment patterns; Account (BC-06) — Customer data |
| Downstream Consumers | Workflow (BC-03) — StrategyRecommended; Agency (BC-07) — RiskCategoryUpdated; Compliance (BC-10) |
| Primary Objective | Build real-time customer intelligence through sentiment analysis, behavioral signals, and predictive models to optimize collection strategy and enable early warning |

### 11.2 Ubiquitous Language -- Sentiment-Specific Terms

| Term | Definition |
|---|---|
| **Sentiment Score** | Numerical score (-1.0 to +1.0) representing customer's emotional state and willingness to cooperate. Computed from multiple signal sources. |
| **Sentiment Label** | Categorical classification: POSITIVE (cooperative, willing to pay), NEUTRAL (responsive but non-committal), NEGATIVE (resistant, unhappy), HOSTILE (abusive, threatening), DISTRESSED (financial hardship, emotional distress). |
| **Behavioral Signal** | An observable pattern in customer's actions that indicates future behavior (e.g., payment pattern deterioration, bounce frequency increase). |
| **Propensity Score** | ML-predicted probability of a specific outcome (propensity to pay, propensity to default, propensity to respond). |
| **Risk Category** | Behavioral segmentation: COOPERATIVE (willing & able), HARDSHIP (willing but unable), UNRESPONSIVE (not engaging), DISPUTATIOUS (questioning obligations), STRATEGIC_DEFAULTER (able but unwilling), ABSCONDER (disappeared/unreachable). |
| **EWS Alert** | Early Warning System alert raised when pre-delinquency signals indicate likelihood of future default. |
| **Signal Source** | Origin of a data point used for analysis: CALL_RECORDING, CHAT_TRANSCRIPT, EMAIL_REPLY, WHATSAPP_REPLY, PAYMENT_BEHAVIOR, BUREAU_DATA, MANDATE_BOUNCE_HISTORY, SOCIAL_MEDIA, COMPLAINT_HISTORY. |
| **Treatment Recommendation** | AI-suggested collection approach based on sentiment and propensity analysis. |

### 11.3 Aggregates

#### 11.3.1 Aggregate: CustomerIntelligenceProfile

| Aspect | Details |
|---|---|
| Aggregate Root | `CustomerIntelligenceProfile` |
| Identity | `IntelligenceProfileId` (derived from CustomerId) |
| Description | Comprehensive intelligence profile for a customer aggregating all sentiment signals, behavioral patterns, and propensity scores. |

**Entities:**
- `CustomerIntelligenceProfile` — Root. Contains overall sentiment score, risk category, propensity scores, last updated timestamp.
- `SentimentHistory` — Time-series of sentiment scores with source attribution.
- `BehavioralSignal` — Individual signals detected from various sources.

**Value Objects:**
- `SentimentScore` — (score: float [-1.0 to +1.0], label: SentimentLabel, confidence: float, source: SignalSource, computedAt: timestamp)
- `RiskCategory` — Enum: COOPERATIVE, HARDSHIP, UNRESPONSIVE, DISPUTATIOUS, STRATEGIC_DEFAULTER, ABSCONDER
- `PropensityScores` — (payProbability: float, defaultProbability: float, respondProbability: float, settleProbability: float, modelVersion: string, computedAt: timestamp)
- `TreatmentRecommendation` — (recommendedTreatment: Treatment, recommendedChannel: Channel, recommendedTiming: TimeSlot, confidence: float, reasoning: string)

**Invariants:**
- Sentiment score must be recomputed on every new signal (conversation, payment, bounce)
- Risk category must be reviewed if sentiment score changes by more than 0.3
- Distress detection must trigger immediate alert regardless of other processing
- Model predictions must include version and confidence for auditability

#### 11.3.2 Aggregate: EWSAlert

| Aspect | Details |
|---|---|
| Aggregate Root | `EWSAlert` |
| Identity | `EWSAlertId` (UUID) |
| Description | Early Warning System alert for an account showing pre-delinquency signals. |

**Entities:**
- `EWSAlert` — Root. Contains alert type, severity, signals that triggered it, recommended action.
- `EWSSignal` — Individual pre-delinquency signal contributing to the alert.

**Value Objects:**
- `AlertSeverity` — Enum: LOW, MEDIUM, HIGH, CRITICAL
- `AlertStatus` — Enum: OPEN, ACKNOWLEDGED, ACTION_TAKEN, FALSE_POSITIVE, CLOSED
- `EWSSignalType` — Enum: SALARY_CREDIT_MISSED, SALARY_REDUCED, BOUNCE_PATTERN_EMERGING, BUREAU_SCORE_DROP, OTHER_LOAN_DEFAULT, BUSINESS_DETERIORATION, GST_FILING_IRREGULAR, HIGH_CREDIT_UTILIZATION, MANDATE_CANCELLATION_ATTEMPT, CUSTOMER_COMPLAINT_SPIKE

### 11.4 Domain Events

| Event ID | Event Name | Published When | Payload | Consumed By |
|---|---|---|---|---|
| DE-SI-001 | `SentimentScoreUpdated` | New sentiment score computed from incoming signal | profileId, loanAccountId, previousScore, newScore, label, source | Workflow (BC-03), Agency (BC-07) |
| DE-SI-002 | `RiskCategoryChanged` | Customer's risk category reclassified | profileId, loanAccountId, previousCategory, newCategory, triggers | Workflow (BC-03), Agency (BC-07) |
| DE-SI-003 | `StrategyRecommended` | AI recommends changing collection strategy | profileId, loanAccountId, currentStrategy, recommendedStrategy, confidence, reasoning | Workflow (BC-03) |
| DE-SI-004 | `DistressDetected` | Customer shows signs of severe financial/emotional distress | profileId, loanAccountId, distressType, source, severity | Workflow (BC-03) — pause aggressive actions, Compliance (BC-10) |
| DE-SI-005 | `EWSAlertRaised` | Early warning signal detected for currently performing account | alertId, loanAccountId, alertType, severity, signals | Workflow (BC-03), Delinquency (BC-02) |
| DE-SI-006 | `PropensityScoreUpdated` | ML model recalculates propensity scores | profileId, loanAccountId, payProbability, defaultProbability, modelVersion | Analytics (BC-11) |
| DE-SI-007 | `AbusiveContentDetected` | Customer's communication contains abusive/threatening content | sessionId, loanAccountId, contentType, severity | Compliance (BC-10), Legal (BC-08) |

### 11.5 Domain Services

#### 11.5.1 SentimentAnalysisService

**Purpose:** Analyzes multiple signal sources to compute real-time sentiment scores.

**Signal Processing Pipeline:**

```
Signal Sources → Preprocessing → Analysis → Scoring → Aggregation → Action

1. VOICE SIGNALS (Call Recordings / AI Voice)
   - Speech-to-text transcription (Hindi, English, regional)
   - Tone analysis (pitch, speed, volume patterns)
   - Emotion detection (angry, frustrated, cooperative, resigned, distressed)
   - Keyword detection (threatening, abusive, hardship indicators)
   - Output: Voice sentiment score + detected emotions

2. TEXT SIGNALS (WhatsApp, SMS replies, Email, Chat)
   - NLP analysis of text content
   - Intent detection (willing to pay, needs time, refusing, disputing, threatening)
   - Language and tone analysis
   - Keyword/phrase detection for compliance triggers
   - Output: Text sentiment score + detected intents

3. BEHAVIORAL SIGNALS (Payment Patterns)
   - Payment regularity analysis (always on time → always late → stopped paying)
   - Bounce frequency and pattern (increasing, decreasing, sporadic)
   - Partial payment behavior (paying something → effort indicator)
   - Response to communications (responding → cooperative; no response → risk)
   - PTP adherence rate (keeps promises → trustworthy; breaks → unreliable)
   - Output: Behavioral sentiment score + payment propensity

4. EXTERNAL SIGNALS
   - Bureau score changes (declining score → risk)
   - Other lender defaults (bureau refresh showing new delinquencies)
   - Social media sentiment (if consented; primarily for business borrowers)
   - Output: External risk score

5. AGGREGATION
   Composite Score = w1 × voice_score + w2 × text_score
                   + w3 × behavioral_score + w4 × external_score

   Weights (configurable, default):
     Voice: 0.25, Text: 0.25, Behavioral: 0.35, External: 0.15

   Composite score mapped to label:
     +0.5 to +1.0 → POSITIVE
     +0.1 to +0.5 → NEUTRAL
     -0.3 to +0.1 → NEGATIVE
     -0.7 to -0.3 → HOSTILE
     -1.0 to -0.7 → DISTRESSED (+ hardship signals)
```

#### 11.5.2 PropensityModelService

**Purpose:** ML models predicting customer behavior outcomes.

**Models:**

| Model | Output | Input Features | Refresh Frequency |
|---|---|---|---|
| **Propensity to Pay** | Probability (0-1) that customer will pay within next 30 days | DPD, bucket, payment history, bounce history, sentiment score, PTP history, product type, vintage, outstanding, employment type, bureau score | Daily (batch) + on-event |
| **Propensity to Default** | Probability (0-1) that account will become NPA within 90 days | Same as above + EWS signals, macro-economic indicators | Daily (batch) |
| **Propensity to Respond** | Probability (0-1) that customer will respond to communication within 48 hours | Contact history, channel engagement rates, time-of-day patterns, response history | Daily (batch) |
| **Propensity to Settle** | Probability (0-1) that customer will accept OTS within next 60 days | DPD, total outstanding, available settlement budget (if known), negotiation history | Weekly (batch) |
| **Best Channel Prediction** | Most effective channel for this customer | Historical channel response rates, delivery success per channel, payment conversion per channel | Weekly (batch) |
| **Best Time Prediction** | Optimal time of day/day of week to contact | Historical response patterns, pickup rates by time, payment conversion by timing | Weekly (batch) |

#### 11.5.3 EarlyWarningService

**Purpose:** Detect pre-delinquency signals in currently performing (0 DPD) accounts.

**EWS Signals & Detection Logic:**

| Signal | Detection Method | Data Source | Severity |
|---|---|---|---|
| Salary credit missed | Savings account analysis — regular salary credit not received by expected date + 5 days | CBS account transactions (with consent / within same bank) | HIGH |
| Salary amount reduced | Salary credit received but amount is >20% lower than 3-month average | CBS account transactions | MEDIUM |
| Bounce pattern emerging | 1st bounce on previously clean mandate | Reconciliation Context bounce events | LOW |
| Consecutive bounces | 2+ consecutive NACH bounces | Reconciliation Context | HIGH |
| Bureau score drop | Credit score dropped >50 points in last 3 months | Bureau refresh (CIBIL/Equifax) | MEDIUM |
| Other loan default | New delinquency appearing in bureau report for same customer | Bureau refresh | HIGH |
| High credit utilization | Credit card utilization >80% or new loans taken | Bureau refresh | MEDIUM |
| Business deterioration | GST filing irregular, GST returns showing revenue decline >30% | GST portal integration (with consent) | HIGH |
| Mandate cancellation attempt | Customer attempts to cancel NACH mandate | NPCI mandate status API | CRITICAL |
| Customer complaint spike | Multiple complaints raised in short period | Complaint management system | MEDIUM |
| Guarantor default | Co-borrower/guarantor showing delinquency | Bureau refresh for co-applicants | HIGH |

**EWS Alert → Action Mapping:**

| Alert Severity | Auto Action |
|---|---|
| LOW | Add to watchlist; flag for next review cycle |
| MEDIUM | Trigger proactive outreach (soft reminder); add to supervisor's review queue |
| HIGH | Trigger immediate outreach (call/WhatsApp); escalate to relationship manager; recommend mandate amount increase |
| CRITICAL | Block mandate cancellation (if possible); immediate senior management escalation; initiate enhanced monitoring |

### 11.6 Functional Requirements

#### 11.6.1 Sentiment Analysis

| Req ID | Requirement | Priority | Acceptance Criteria |
|---|---|---|---|
| COL-SI-001 | System shall compute sentiment score for every customer interaction (call, chat, WhatsApp, email) within 5 minutes of interaction completion. | P0 | Score available within 5 minutes; accuracy >80% vs human labeling |
| COL-SI-002 | System shall support Hindi, English, and Hinglish (mixed) sentiment analysis for voice and text. Minimum 5 additional regional languages within 12 months. | P0 | Hindi/English/Hinglish at launch; regional languages in Phase 2 |
| COL-SI-003 | System shall detect customer distress signals (financial hardship expressions, suicidal ideation keywords, crying detection in voice) and trigger immediate escalation. | P0 | Distress detection with <99.5% recall (minimize missed distress signals) |
| COL-SI-004 | System shall detect abusive/threatening content in customer communications and flag for compliance. | P0 | Abusive content detection accuracy >95% |
| COL-SI-005 | System shall maintain sentiment history with trend visualization (improving, stable, deteriorating). | P0 | Trend visible on customer 360 dashboard |
| COL-SI-006 | System shall map sentiment + behavior to risk categories (Cooperative, Hardship, Unresponsive, Disputatious, Strategic Defaulter, Absconder). | P0 | Risk category updated within 1 hour of significant signal change |
| COL-SI-007 | System shall auto-recommend collection strategy adjustments based on sentiment (e.g., distressed customer → empathetic approach, reduce communication frequency; strategic defaulter → firm approach, escalate faster). | P0 | Recommendations generated; logged for audit |

#### 11.6.2 Propensity Models

| Req ID | Requirement | Priority | Acceptance Criteria |
|---|---|---|---|
| COL-SI-008 | System shall compute Propensity to Pay score for all delinquent accounts daily (batch) and on significant events (payment, bounce, conversation). | P0 | Scores available by 8 AM daily; event-triggered within 15 minutes |
| COL-SI-009 | System shall compute Propensity to Default score for all active accounts (including performing) daily. | P0 | Scores for 100% active portfolio by 8 AM |
| COL-SI-010 | System shall predict best communication channel and best time-of-day per customer. | P1 | Prediction accuracy: >60% improvement over random baseline |
| COL-SI-011 | All ML model predictions shall include confidence score and model version for auditability. | P0 | Version and confidence in every score record |
| COL-SI-012 | System shall support model retraining pipeline with A/B testing of model versions. | P1 | New model versions deployable with traffic split |
| COL-SI-013 | System shall monitor model performance (accuracy, AUC, precision, recall) with drift detection and auto-alerting. | P0 | Model performance dashboard; alert on >5% performance degradation |

#### 11.6.3 Early Warning System

| Req ID | Requirement | Priority | Acceptance Criteria |
|---|---|---|---|
| COL-SI-014 | System shall monitor all performing accounts (DPD=0) for pre-delinquency signals daily. | P0 | 100% performing portfolio scanned daily |
| COL-SI-015 | System shall detect salary credit anomalies (missed, reduced) for salaried borrowers within same-bank accounts. | P0 | Anomaly detected within 1 day of expected salary date |
| COL-SI-016 | System shall integrate with credit bureaus for periodic refresh (monthly for NPA, quarterly for performing) to detect bureau score drops and other lender defaults. | P0 | Bureau refresh completed per schedule; alerts raised within 24 hours of refresh |
| COL-SI-017 | System shall support GST data integration (with borrower consent) for business loan EWS. | P1 | GST data ingested; deterioration signals detected |
| COL-SI-018 | System shall raise EWS alerts with severity classification and recommended actions. | P0 | Alerts raised within 24 hours of signal detection |
| COL-SI-019 | EWS shall achieve detection rate of >70% of future NPAs at least 30 days before actual NPA date. | P0 | Detection rate validated in back-testing; monitored monthly |
| COL-SI-020 | System shall provide EWS dashboard with alert inventory, severity distribution, and action status. | P0 | Dashboard with drill-down capabilities |

---

## END OF PART 05

Next: **COL-BRD-Part06-Mandate-Account360.md** -- Mandate & Account 360 Bounded Context.
