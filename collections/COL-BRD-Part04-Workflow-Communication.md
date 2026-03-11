# PART 04 -- WORKFLOW & CAMPAIGN + COMMUNICATION BOUNDED CONTEXTS (BC-03 & BC-04)

---

## 9. WORKFLOW & CAMPAIGN CONTEXT (BC-03)

### 9.1 Context Overview

| Attribute | Details |
|---|---|
| Context Name | Workflow & Campaign Context |
| Context ID | BC-03 |
| Domain Classification | Core Domain |
| Owner Team | Collection Strategy |
| Upstream Dependencies | Delinquency (BC-02) — BucketChanged, CollectionCaseOpened; Sentiment (BC-05) — StrategyRecommended |
| Downstream Consumers | Communication (BC-04), Agency (BC-07), Legal (BC-08) |
| Primary Objective | Orchestrate collection actions through configurable, bucket-driven workflows with intelligent escalation and campaign management |

### 9.2 Ubiquitous Language -- Workflow-Specific Terms

| Term | Definition |
|---|---|
| **Collection Strategy** | A named set of workflow rules defining the sequence, timing, and channel of collection actions for a specific combination of bucket, product type, and customer segment. |
| **Workflow Template** | A reusable blueprint defining the steps, timing, and conditions for a collection workflow. |
| **Workflow Step** | A single action within a workflow (e.g., "Send SMS reminder on DPD+3"). |
| **Escalation Rule** | A condition that triggers movement to a more aggressive collection action (e.g., broken PTP → escalate to field visit). |
| **Campaign** | A bulk execution of a workflow step against a filtered portfolio segment (e.g., "Send WhatsApp to all SMA-1 accounts with EMI > ₹10,000"). |
| **Suppression Rule** | A rule that prevents a communication from being sent (e.g., customer in DND, legal hold, active dispute, recent payment). |
| **Throttle** | Rate limiting on communications per customer per day/week to comply with fair practices code. |
| **Treatment** | The overall collection approach for an account (Soft/Medium/Aggressive/Legal). Derived from bucket, sentiment, and behavior. |

### 9.3 Aggregates

#### 9.3.1 Aggregate: CollectionStrategy

| Aspect | Details |
|---|---|
| Aggregate Root | `CollectionStrategy` |
| Identity | `StrategyId` (UUID) |
| Description | Defines the collection approach for a specific segment. Maps bucket × product × customer_segment to a workflow template. |

**Entities:**
- `CollectionStrategy` — Root. Contains name, applicability rules, effective dates, approval status.
- `StrategyRule` — Individual rule mapping a condition to a workflow template.

**Value Objects:**
- `SegmentCriteria` — (bucket, productType, customerSegment, exposureRange, region, vintage)
- `StrategyStatus` — Enum: DRAFT, PENDING_APPROVAL, ACTIVE, PAUSED, RETIRED
- `EffectivePeriod` — (fromDate, toDate)

#### 9.3.2 Aggregate: WorkflowTemplate

| Aspect | Details |
|---|---|
| Aggregate Root | `WorkflowTemplate` |
| Identity | `WorkflowTemplateId` (UUID) |
| Description | Blueprint of collection steps to execute in sequence with timing and conditions. |

**Entities:**
- `WorkflowTemplate` — Root. Contains name, description, version, steps.
- `WorkflowStep` — Ordered step with action type, timing, conditions, channel, template reference.
- `EscalationRule` — Condition that triggers skip or jump to a different step.

**Value Objects:**
- `StepTiming` — (triggerType [DPD_DAY/DAYS_AFTER_PREVIOUS/EVENT_DRIVEN], triggerValue, timeOfDay, dayOfWeek)
- `StepAction` — Enum: SEND_SMS, SEND_WHATSAPP, SEND_EMAIL, TRIGGER_IVR, TRIGGER_AI_VOICE_CALL, TRIGGER_AI_CHAT, ASSIGN_FIELD_VISIT, GENERATE_DEMAND_NOTICE, GENERATE_LEGAL_NOTICE, ALLOCATE_TO_AGENCY, ESCALATE_TO_SUPERVISOR
- `StepCondition` — (conditionType [IF_NO_PAYMENT/IF_PTP_BROKEN/IF_NO_RESPONSE/IF_SENTIMENT_NEGATIVE], evaluationPeriodDays)

#### 9.3.3 Aggregate: WorkflowExecution

| Aspect | Details |
|---|---|
| Aggregate Root | `WorkflowExecution` |
| Identity | `WorkflowExecutionId` (UUID) |
| Description | Runtime instance of a workflow template for a specific loan account/collection case. |

**Entities:**
- `WorkflowExecution` — Root. Links to collection case, strategy, template. Tracks current step, status.
- `StepExecution` — Execution record for each step (status, executedAt, result, retries).

**Value Objects:**
- `ExecutionStatus` — Enum: ACTIVE, PAUSED, COMPLETED, CANCELLED, SUPERSEDED
- `PauseReason` — Enum: CUSTOMER_PTP, DISPUTE_RAISED, LEGAL_HOLD, PAYMENT_RECEIVED, MANUAL_PAUSE, SENTIMENT_DISTRESS
- `StepResult` — (outcome [SUCCESS/FAILED/SKIPPED/SUPPRESSED], suppressionReason, retryCount)

**Invariants:**
- Only one active workflow execution per collection case at a time
- Workflow must be paused if payment received (auto-resume if payment insufficient)
- Workflow must be paused if dispute is raised
- Communication steps must respect suppression rules (DND, time-of-day, frequency limits)

### 9.4 Domain Events

| Event ID | Event Name | Published When | Payload | Consumed By |
|---|---|---|---|---|
| DE-WF-001 | `WorkflowStarted` | New workflow execution begins for a collection case | executionId, caseId, loanAccountId, strategyId, templateId | Analytics (BC-11) |
| DE-WF-002 | `WorkflowStepTriggered` | A step in the workflow is ready for execution | executionId, stepId, actionType, channel, loanAccountId | Communication (BC-04), Agency (BC-07) |
| DE-WF-003 | `WorkflowStepCompleted` | Step execution completed (success or failure) | executionId, stepId, result, channel | Analytics (BC-11) |
| DE-WF-004 | `WorkflowPaused` | Workflow paused due to payment, dispute, or manual hold | executionId, pauseReason, pausedBy | Analytics (BC-11) |
| DE-WF-005 | `WorkflowEscalated` | Escalation triggered (broken PTP, no response, etc.) | executionId, escalationReason, fromStep, toStep | Agency (BC-07), Legal (BC-08) |
| DE-WF-006 | `CampaignExecuted` | Bulk campaign executed against portfolio segment | campaignId, segmentCriteria, accountCount, channel | Communication (BC-04), Analytics (BC-11) |
| DE-WF-007 | `WorkflowCompleted` | All steps completed or case resolved | executionId, caseId, resolution | Analytics (BC-11) |

### 9.5 Domain Services

#### 9.5.1 StrategyAssignmentService

**Purpose:** Determines which collection strategy/workflow to apply to a given account based on its attributes.

**Strategy Selection Matrix (Sample):**

| Bucket | Product | Customer Segment | Outstanding | Treatment | Workflow Template |
|---|---|---|---|---|---|
| SMA-0 (1-30) | Personal Loan | Salaried | Any | Soft | WF-SOFT-SMS-WHATSAPP |
| SMA-0 (1-30) | Home Loan | Any | Any | Soft | WF-SOFT-SMS-EMAIL |
| SMA-1 (31-60) | Personal Loan | Salaried | <₹5L | Medium | WF-MED-MULTI-CHANNEL |
| SMA-1 (31-60) | Personal Loan | Self-Employed | Any | Medium | WF-MED-CALL-VISIT |
| SMA-2 (61-90) | Any | Any | Any | Aggressive | WF-AGG-FULL-ESCALATION |
| NPA-SS (91-365) | Secured | Any | >₹25L | Aggressive+Legal | WF-LEGAL-SARFAESI |
| NPA-SS (91-365) | Unsecured | Any | Any | Aggressive | WF-AGG-AGENCY-LEGAL |
| NPA-D1+ | Any | Any | Any | Legal/Write-off | WF-LEGAL-RECOVERY |

#### 9.5.2 WorkflowSchedulerService

**Purpose:** Manages timing of workflow step execution respecting all constraints.

**Timing Rules:**

| Rule ID | Rule | RBI/Regulatory Reference |
|---|---|---|
| COL-WF-TR-001 | No outbound calls before 8:00 AM or after 7:00 PM (customer's local time) | RBI Fair Practices Code; Supreme Court guidelines |
| COL-WF-TR-002 | Maximum 3 communication attempts per day per customer across all channels | Internal policy (best practice) |
| COL-WF-TR-003 | Maximum 1 phone call per day unless customer requests callback | RBI Fair Practices Code |
| COL-WF-TR-004 | No communication on national holidays and designated restricted days | Internal policy |
| COL-WF-TR-005 | Minimum 48-hour gap between consecutive demand notices | Legal requirement |
| COL-WF-TR-006 | SARFAESI Section 13(2) notice: 60-day response period before further action | SARFAESI Act, 2002 |

### 9.6 Functional Requirements

#### 9.6.1 Strategy & Workflow Configuration

| Req ID | Requirement | Priority | Acceptance Criteria |
|---|---|---|---|
| COL-WF-001 | System shall support configurable collection strategies mapped to bucket × product × segment × exposure combinations. | P0 | Strategies configurable via admin UI with maker-checker |
| COL-WF-002 | System shall support workflow templates with ordered steps, each defining: action type, channel, timing, conditions, and escalation triggers. | P0 | Templates support 20+ steps; drag-and-drop UI for step ordering |
| COL-WF-003 | System shall support versioning of strategies and templates with effective date management. | P0 | New version activatable on future date; old version auto-retired |
| COL-WF-004 | System shall support A/B testing of workflow templates (split portfolio between variants, measure effectiveness). | P1 | A/B split configurable; per-variant metrics tracked |
| COL-WF-005 | System shall support strategy override by Sentiment & Intelligence context (AI recommends different approach). | P1 | AI recommendation accepted/rejected by system; override logged |
| COL-WF-006 | System shall support multi-language workflow templates (Hindi, English, Tamil, Telugu, Kannada, Bengali, Marathi, Gujarati, Malayalam, Odia, Punjabi — minimum 11 languages). | P0 | Template content in all configured languages |

#### 9.6.2 Workflow Execution

| Req ID | Requirement | Priority | Acceptance Criteria |
|---|---|---|---|
| COL-WF-007 | System shall auto-start workflow execution when CollectionCaseOpened or BucketChanged event is received. | P0 | Workflow starts within 5 minutes of triggering event |
| COL-WF-008 | System shall execute workflow steps based on configured timing (DPD day, days after previous step, event-driven). | P0 | Steps fire at correct time; ±5 minute tolerance |
| COL-WF-009 | System shall auto-pause workflow when payment is received for the account. Resume if payment insufficient to clear all dues. | P0 | Pause within 1 minute of payment event; resume within 5 minutes of insufficiency determination |
| COL-WF-010 | System shall auto-pause workflow when customer raises dispute. Resume on dispute resolution (if unfounded). | P0 | Dispute pause immediate; resume on resolution event |
| COL-WF-011 | System shall enforce all suppression rules before executing any communication step (DND, time-of-day, frequency, legal hold). | P0 | Suppressed steps logged with reason; no violation of timing rules |
| COL-WF-012 | System shall support escalation triggers: broken PTP, no response after N attempts, sentiment deterioration, threshold DPD reached. | P0 | Escalation fires correctly per configured triggers |
| COL-WF-013 | System shall supersede active workflow when account changes bucket (new bucket → new strategy → new workflow). | P0 | Old workflow cancelled; new workflow started with context carried forward |

#### 9.6.3 Campaign Management

| Req ID | Requirement | Priority | Acceptance Criteria |
|---|---|---|---|
| COL-WF-014 | System shall support bulk campaign creation targeting filtered portfolio segments (by bucket, product, region, DPD range, amount range). | P0 | Campaign reaches all targeted accounts; segment configurable via UI |
| COL-WF-015 | System shall support campaign scheduling (immediate, scheduled date/time, recurring). | P0 | Campaigns execute at scheduled time |
| COL-WF-016 | System shall track campaign metrics: total targeted, delivered, opened (email), responded, payments received within 7 days. | P0 | Real-time campaign dashboard |
| COL-WF-017 | System shall support campaign approval workflow (maker-checker) before execution. | P0 | No campaign without approval |

---

## 10. COMMUNICATION CONTEXT (BC-04)

### 10.1 Context Overview

| Attribute | Details |
|---|---|
| Context Name | Communication Context |
| Context ID | BC-04 |
| Domain Classification | Supporting Domain |
| Owner Team | Communication Platform / Digital |
| Upstream Dependencies | Workflow (BC-03) — WorkflowStepTriggered; Account (BC-06) — Contact details, preferences |
| Downstream Consumers | Sentiment (BC-05), Compliance (BC-10), Analytics (BC-11) |
| Primary Objective | Deliver collection communications across all channels with tracking, compliance, and unified conversation history |

### 10.2 Aggregates

#### 10.2.1 Aggregate: CommunicationRecord

| Aspect | Details |
|---|---|
| Aggregate Root | `CommunicationRecord` |
| Identity | `CommunicationId` (UUID) |
| Description | Represents a single outbound or inbound communication with a customer across any channel. |

**Entities:**
- `CommunicationRecord` — Root. Contains channel, direction, content, delivery status, metadata.
- `DeliveryAttempt` — Track each delivery attempt (for retries).

**Value Objects:**
- `Channel` — Enum: SMS, IVR, WHATSAPP, EMAIL, AI_VOICE, AI_CHAT, FIELD_VISIT, PHYSICAL_LETTER, PUSH_NOTIFICATION
- `Direction` — Enum: OUTBOUND, INBOUND
- `DeliveryStatus` — Enum: QUEUED, SENT, DELIVERED, READ, RESPONDED, FAILED, BOUNCED, OPTED_OUT
- `MessageContent` — (templateId, language, parameters, renderedContent, attachments)
- `ComplianceCheck` — (dndChecked, timeWindowChecked, frequencyChecked, allPassed, failureReasons)

#### 10.2.2 Aggregate: ConversationSession

| Aspect | Details |
|---|---|
| Aggregate Root | `ConversationSession` |
| Identity | `SessionId` (UUID) |
| Description | An interactive conversation session with a customer (AI chatbot, AI voice, or human agent). |

**Entities:**
- `ConversationSession` — Root. Contains channel, startTime, endTime, participants, outcome.
- `ConversationTurn` — Each message/utterance in the conversation (speaker, content, timestamp, intent detected).
- `ConversationOutcome` — Final disposition (PTP, dispute, callback requested, payment made, escalated, etc.).

**Value Objects:**
- `SessionType` — Enum: AI_CHAT, AI_VOICE, HUMAN_TELE, HUMAN_FIELD
- `SentimentSnapshot` — (score, label [POSITIVE/NEUTRAL/NEGATIVE/HOSTILE], confidence)
- `DetectedIntent` — (intentCode [PAY_NOW/NEED_TIME/DISPUTE/FINANCIAL_HARDSHIP/ABUSIVE], confidence)
- `EscalationTrigger` — (reason [CUSTOMER_REQUEST/SENTIMENT_HOSTILE/COMPLIANCE_RISK/COMPLEX_QUERY], escalatedTo)

### 10.3 Domain Events

| Event ID | Event Name | Published When | Payload | Consumed By |
|---|---|---|---|---|
| DE-COM-001 | `MessageSent` | Outbound message dispatched | communicationId, loanAccountId, channel, templateId | Compliance (BC-10) |
| DE-COM-002 | `MessageDelivered` | Delivery confirmation received | communicationId, deliveredAt, channel | Analytics (BC-11) |
| DE-COM-003 | `MessageFailed` | Delivery failed after retries | communicationId, failureReason, channel | Workflow (BC-03) — trigger alternate channel |
| DE-COM-004 | `CustomerResponded` | Customer responded (reply SMS, WhatsApp message, email reply) | communicationId, responseContent, channel | Sentiment (BC-05), Delinquency (BC-02) |
| DE-COM-005 | `ConversationStarted` | AI or human conversation session begins | sessionId, loanAccountId, channel, sessionType | Analytics (BC-11) |
| DE-COM-006 | `ConversationCompleted` | Conversation session ends | sessionId, outcome, disposition, sentimentScore, duration | Sentiment (BC-05), Delinquency (BC-02), Analytics (BC-11) |
| DE-COM-007 | `ConversationEscalated` | AI conversation handed off to human agent | sessionId, escalationReason, fromAgent, toAgent | Compliance (BC-10) |
| DE-COM-008 | `PTCapturedViaConversation` | PTP captured during AI/human conversation | sessionId, loanAccountId, promisedAmount, promisedDate | Delinquency (BC-02) |
| DE-COM-009 | `CustomerOptedOut` | Customer opted out of a channel | loanAccountId, channel, optOutDate | Workflow (BC-03), Compliance (BC-10) |

### 10.4 Functional Requirements

#### 10.4.1 SMS Channel

| Req ID | Requirement | Priority | Acceptance Criteria |
|---|---|---|---|
| COL-COM-001 | System shall send SMS via DLT-registered templates (TRAI compliance) with registered sender IDs (headers). | P0 | All SMS via DLT-registered templates; sender ID pre-approved |
| COL-COM-002 | System shall support multi-language SMS (transliterated and Unicode). | P0 | SMS in customer's preferred language |
| COL-COM-003 | System shall track SMS delivery status (sent, delivered, failed) via DLR (Delivery Level Report) from SMS gateway. | P0 | Delivery status updated within 5 minutes |
| COL-COM-004 | System shall include payment link (short URL) in SMS where applicable. | P0 | Payment link functional; tracks click-through |
| COL-COM-005 | System shall support SMS shortcode for two-way communication (customer replies with PTP date). | P1 | Inbound SMS parsed; PTP captured |

#### 10.4.2 IVR (Interactive Voice Response) Channel

| Req ID | Requirement | Priority | Acceptance Criteria |
|---|---|---|---|
| COL-COM-006 | System shall support outbound IVR campaigns with auto-dialer (predictive/progressive dialing modes). | P0 | Auto-dialer connects answered calls to IVR flow |
| COL-COM-007 | IVR shall play pre-recorded multilingual messages (minimum 5 languages) with customer name and amount personalization (TTS). | P0 | Personalized message plays in customer's language |
| COL-COM-008 | IVR shall capture customer input via DTMF (press 1 to pay now, press 2 for PTP, press 3 to speak to agent, press 4 for dispute). | P0 | DTMF inputs correctly routed |
| COL-COM-009 | IVR shall support transfer to live agent with full context (screen pop with customer 360). | P0 | Agent receives screen pop within 2 seconds of transfer |
| COL-COM-010 | All IVR calls shall be recorded and stored (minimum 7 years for regulatory compliance). | P0 | 100% call recording; accessible for audit |
| COL-COM-011 | System shall comply with TRAI calling time restrictions (8 AM - 7 PM) and autodial regulations. | P0 | No calls outside permitted hours |

#### 10.4.3 WhatsApp Business Channel

| Req ID | Requirement | Priority | Acceptance Criteria |
|---|---|---|---|
| COL-COM-012 | System shall integrate with WhatsApp Business API (via BSP — Business Solution Provider such as Gupshup, Infobip, Kaleyra). | P0 | Messages sent/received via WhatsApp Business API |
| COL-COM-013 | System shall send template messages (pre-approved by Meta) for outbound collection reminders. | P0 | Templates pre-approved; sent successfully |
| COL-COM-014 | System shall support interactive messages (quick reply buttons: "Pay Now", "Request Callback", "Need More Time") and list messages. | P0 | Interactive elements functional; responses captured |
| COL-COM-015 | System shall support rich media: PDF attachments (SOA, demand notice), images, payment QR codes. | P0 | Attachments delivered; viewable by customer |
| COL-COM-016 | System shall support two-way conversation within 24-hour session window (respond to customer queries about outstanding, EMI details, settlement options). | P0 | Two-way conversation within session window |
| COL-COM-017 | System shall track message status: sent, delivered, read, responded. | P0 | Status tracking via WhatsApp webhooks |

#### 10.4.4 Email Channel

| Req ID | Requirement | Priority | Acceptance Criteria |
|---|---|---|---|
| COL-COM-018 | System shall send HTML-formatted emails with institution branding, using authenticated domain (SPF/DKIM/DMARC). | P0 | Emails pass spam filters; render correctly in major clients |
| COL-COM-019 | System shall support email attachments: SOA (PDF), demand notice (PDF), payment receipt (PDF). | P0 | Attachments generated dynamically; accurate content |
| COL-COM-020 | System shall track email delivery (sent, delivered, bounced), opens, and link clicks. | P0 | Tracking via email service provider webhooks |
| COL-COM-021 | System shall support unsubscribe link per CAN-SPAM / IT Act compliance (with distinction: transactional collection notices are exempt from unsubscribe but must be clearly identified). | P1 | Transactional classification correct |

#### 10.4.5 Agentic Conversation — AI Text (Chatbot)

| Req ID | Requirement | Priority | Acceptance Criteria |
|---|---|---|---|
| COL-COM-022 | System shall provide AI-powered chatbot for collections conversations via WhatsApp and web portal. | P0 | Chatbot handles 80%+ of routine collection conversations without human escalation |
| COL-COM-023 | AI chatbot shall handle: payment reminders, EMI breakup queries, outstanding balance queries, PTP capture, payment link generation, settlement/restructuring inquiry, dispute intake. | P0 | All listed intents handled correctly |
| COL-COM-024 | AI chatbot shall maintain empathetic, professional tone. No threatening language, no harassment. Compliance guardrails hardcoded. | P0 | 100% compliance with fair practices; validated via regular audit |
| COL-COM-025 | AI chatbot shall detect customer distress signals (expressions of financial hardship, suicidal ideation, abusive language) and immediately escalate to human supervisor. | P0 | Distress detection with <5 second escalation |
| COL-COM-026 | AI chatbot shall authenticate customer before sharing account-specific details (OTP verification or knowledge-based authentication). | P0 | No data leakage without authentication |
| COL-COM-027 | AI chatbot shall support negotiation within approved settlement parameters (pre-configured min/max OTS ranges per bucket/product). | P1 | Negotiation within approved limits; supervisor approval for exceptions |
| COL-COM-028 | AI chatbot shall generate conversation summary with extracted intents, dispositions, and PTP for the collection case. | P0 | Summary auto-posted to collection case |

#### 10.4.6 Agentic Conversation — AI Voice

| Req ID | Requirement | Priority | Acceptance Criteria |
|---|---|---|---|
| COL-COM-029 | System shall support AI-powered outbound voice calls for collection conversations (voice agent). | P0 | AI voice agent makes outbound calls; natural conversational flow |
| COL-COM-030 | AI voice agent shall use natural language understanding (NLU) to interpret customer responses in Hindi, English, and configurable regional languages. | P0 | NLU accuracy >90% for supported languages |
| COL-COM-031 | AI voice agent shall dynamically adjust conversation script based on customer responses and real-time sentiment detection. | P0 | Script adaptation visible in conversation log |
| COL-COM-032 | AI voice agent shall capture PTP (amount and date) via natural conversation and confirm with customer. | P0 | PTP captured accurately; confirmation obtained |
| COL-COM-033 | AI voice agent shall seamlessly hand off to human agent when: customer requests it, sentiment turns hostile, complex query detected, or compliance risk identified. | P0 | Handoff within 5 seconds; full context transferred |
| COL-COM-034 | AI voice agent shall identify itself as an AI assistant at the beginning of the call (transparency requirement). | P0 | Disclosure in every call |
| COL-COM-035 | All AI voice calls shall be recorded, transcribed, and analyzed for compliance and sentiment. | P0 | Recording and transcription for 100% calls |
| COL-COM-036 | AI voice agent shall comply with all TRAI and RBI calling regulations (time restrictions, disclosure, no threatening). | P0 | Zero regulatory violations |

#### 10.4.7 Unified Communication Management

| Req ID | Requirement | Priority | Acceptance Criteria |
|---|---|---|---|
| COL-COM-037 | System shall maintain unified conversation history across all channels per customer/loan account. | P0 | Single timeline view showing all communications regardless of channel |
| COL-COM-038 | System shall support channel preference management per customer (preferred channel, opted-out channels). | P0 | Preferences respected in all workflow executions |
| COL-COM-039 | System shall support channel fallback (if primary channel fails, auto-try next channel per configured sequence). | P0 | Fallback executes within 30 minutes of primary failure |
| COL-COM-040 | System shall enforce DND (Do Not Disturb) registry compliance (TRAI NCPR) — check DND status before promotional communications. Note: loan collection reminders for existing obligations are generally exempt from DND as "transactional" but must use registered templates. | P0 | DND check before every communication; exemption correctly applied |
| COL-COM-041 | System shall enforce communication frequency limits (configurable: default max 3 communications/day, max 10/week across all channels). | P0 | Frequency limits enforced; excess suppressed with reason logged |
| COL-COM-042 | System shall track cost per communication per channel and optimize channel selection for cost-effectiveness. | P1 | Cost tracked; cheapest effective channel preferred |

### 10.5 Sample Workflow -- SMA-1 Personal Loan (31-60 DPD)

```
Workflow Template: WF-MED-MULTI-CHANNEL
Product: Personal Loan | Bucket: SMA-1 | Treatment: Medium

Day 31 (DPD=31, bucket changed to SMA-1):
  Step 1: Send SMS reminder
    Template: "Dear {name}, your EMI of ₹{amount} for loan {loanId} is overdue
    by {dpd} days. Please pay immediately to avoid late charges.
    Pay now: {paymentLink}. Call {helpline} for assistance."
    Suppression: Check DND, check frequency limit

Day 33 (DPD=33):
  Step 2: Send WhatsApp message with SOA attachment
    Template: Interactive message with "Pay Now" and "Request Callback" buttons
    Attachment: PDF Statement of Account
    Condition: Execute only if no payment received since Step 1

Day 36 (DPD=36):
  Step 3: AI Voice Call
    Script: Empathetic reminder, offer payment options, capture PTP
    Condition: Execute only if no payment received AND no response to WhatsApp
    Timing: Between 10 AM and 6 PM customer local time
    Sentiment monitoring: Active

Day 40 (DPD=40):
  Step 4: Send formal email with demand notice PDF
    Condition: Execute only if no payment AND no PTP received

Day 45 (DPD=45):
  Step 5: IVR Call
    Condition: Execute if PTP was broken OR no payment

Day 50 (DPD=50):
  Step 6: AI Chat initiation via WhatsApp
    Objective: Negotiate payment plan, offer restructuring if eligible
    Condition: Execute if no resolution from previous steps

Day 55 (DPD=55):
  Step 7: Escalate — Assign to tele-calling agent (human)
    Condition: AI channels exhausted without resolution
    Action: Create tele-calling queue item with full interaction history

Day 58 (DPD=58):
  Step 8: Escalate — Assign to field visit
    Condition: Tele-calling unsuccessful (not reachable or refused)
    Action: Create field visit task with route optimization

[If DPD reaches 61 → Bucket changes to SMA-2 → New workflow WF-AGG-FULL-ESCALATION supersedes]
```

---

## END OF PART 04

Next: **COL-BRD-Part05-Sentiment-Intelligence.md** -- Sentiment & Intelligence Bounded Context.
