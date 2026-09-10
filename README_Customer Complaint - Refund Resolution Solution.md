# Customer Complaint – Refund Resolution
## UiPath Maestro Flow – Business Demonstration Solution

> **Purpose:** End-to-end customer complaint and refund orchestration using an Manual trigger, Autonomous Agent, decision gateway, human approval, RPA refund processing, customer notification, and case logging.
>
> **Environment note:** Exact Studio Web labels, connector fields, and UI can vary by tenant, licensing, and product updates. Prefer the Studio Web variable/Insert picker over manually guessing object paths.

---

# 1. Business Scenario

A customer sends a complaint by email. Maestro detects the complaint, an Autonomous Agent analyzes it, and the process follows one of four routes:

- `AUTO_REFUND`
- `HUMAN_APPROVAL`
- `REJECT`
- `INFORMATION_REQUIRED`

The process ultimately notifies the customer and records the outcome.

### Business value

- AI-assisted complaint analysis
- Deterministic routing
- Human-in-the-loop governance
- Automated refund processing
- Automated customer communication
- Traceable case handling

---

# 2. Final High-Level Architecture

```text
Microsoft 365 Outlook
        |
        v
  Email Received
  Maestro Trigger
        |
        v
Customer Complaint Analysis Agent
        |
        v
  Determine Refund Route
      /    |      |       \
     /     |      |        \
AUTO   HUMAN  REJECT   INFORMATION
REFUND APPROVAL           REQUIRED
  |       |        |         |
  |       v        |         v
  |   Quick Form   |    Human Review /
  |   Approval     |    Request Info
  |       |        |         |
  |    Approve/    |         |
  |     Reject     |         |
  |       |        |         |
  +-------+--------+---------+
          |
          v
    Customer Notification
          |
          v
       Case Logging
          |
          v
         END
```

---

# 3. Create the Maestro Flow

## 3.1 Create project

Create a new Maestro Flow in Studio Web.

Recommended name:

```text
Customer Complaint - Refund Resolution
```

## 3.2 Add nodes

Recommended business-readable node names:

```text
Email Received
Customer Complaint Analysis Agent
Determine Refund Route
Quick Form Approval
Process Refund
Notify Customer
Log Case
```

---

# 4. Outlook Email Trigger

## 4.1 Connection

Use the Microsoft 365 Outlook connection for the mailbox that receives customer complaints.

The Microsoft Outlook 365 `Email Received` trigger starts execution when a new email arrives. UiPath documents Outlook as supporting the `Email Received` event.

Reference: https://docs.uipath.com/activities/other/latest/productivity/office365-trigger-new-email-received

## 4.2 Mail folder

For development/demo, use a dedicated Outlook folder:

```text
Flow Demo
```

Example:

```text
Outlook
  └── Flow Demo
       └── CUSTOMER COMPLAINT - Refund Request
```

## 4.3 Recommended initial configuration

| Property | Value |
|---|---|
| Connection | Microsoft 365 Outlook |
| Mailbox | Connected user's mailbox |
| Folder | `Flow Demo` |
| Subject filter | Add after initial trigger validation |
| Attachments | Enable if evidence processing will be added |
| Mark as read | Based on business preference |

Start without unnecessary filters.

## 4.4 Trigger testing

Send a matching email into `Flow Demo` and run/test the trigger.

Example:

**Subject**

```text
CUSTOMER COMPLAINT - Refund Request
```

**Body**

```text
Hello Customer Support,

I received my order ORD-45821 today, but the product
was damaged when I opened the package.

The order amount was ₹2499 and I would like a refund.

Regards,
John
```

If testing reports `DebugTriggerNoMatches` / `No Message events found`, create a matching email event in the configured folder and test again.

---

# 5. Data Model

Recommended complaint data:

| Name | Type | Purpose |
|---|---|---|
| `CustomerName` | String | Customer name |
| `CustomerEmail` | String | Customer email address |
| `EmailSubject` | String | Original complaint subject |
| `ComplaintText` | String | Email body |
| `OrderId` | String | Order identifier |
| `OrderAmount` | Number | Verified order amount where available |

Studio Web automatically exposes activity outputs as variables that can be used by downstream steps.

Reference: https://docs.uipath.com/studio-web/automation-cloud/latest/user-guide/passing-values-between-activities

---

# 6. Autonomous Agent

## 6.1 Purpose

The Agent should:

1. Read the complaint.
2. Classify the complaint.
3. Assess severity.
4. Assess sentiment.
5. Determine refund eligibility.
6. Determine the recommended refund amount.
7. Recommend the processing route.
8. Explain the recommendation.
9. Identify missing information when necessary.

The Agent should **not** send emails, issue refunds, or approve its own cases.

Reference: https://docs.uipath.com/studio-web/automation-cloud/latest/user-guide/building-an-agent-in-studio-web

---

# 7. Agent Model Settings

Recommended starting configuration:

| Property | Value |
|---|---|
| Harness | Standard harness |
| Model | `anthropic.claude-sonnet-5` if available |
| Temperature/creativity | Low, approximately 0–0.2 |
| Max iterations | 3 |

Use a low-variance configuration because this is a classification/routing use case.

---

# 8. Agent Inputs

Create these in the Agent Data Manager.

## `CustomerName`

Type: `String`

Description:

```text
The customer's name extracted from the incoming complaint email.
If unavailable, leave blank and do not invent it.
```

## `CustomerEmail`

Type: `String`

Description:

```text
The customer's email address.
```

## `EmailSubject`

Type: `String`

Description:

```text
The subject line of the incoming customer complaint email.
```

## `ComplaintText`

Type: `String`

Description:

```text
The complete body/content of the customer's complaint email.
```

## `OrderId`

Type: `String`

Description:

```text
The order ID mentioned in the complaint. Leave blank if not present.
```

## `OrderAmount`

Type: `Number`

Description:

```text
The verified order amount in INR when available.
Do not invent an amount.
```

---

# 9. Agent Outputs

Use the following outputs.

| Output | Type | Purpose |
|---|---|---|
| `ComplaintCategory` | String | Complaint type |
| `Severity` | String | LOW / MEDIUM / HIGH |
| `Sentiment` | String | POSITIVE / NEUTRAL / NEGATIVE |
| `RefundEligible` | Boolean | Refund eligibility |
| `RefundAmount` | Number | Recommended refund amount |
| `Decision` | String | Process route |
| `Reason` | String | Concise justification |
| `MissingInformation` | String | Information required from customer |

Expected `ComplaintCategory` values:

```text
DAMAGED_PRODUCT
DEFECTIVE_PRODUCT
WRONG_ITEM
MISSING_ITEM
LATE_DELIVERY
DUPLICATE_CHARGE
OTHER
```

Expected `Decision` values:

```text
AUTO_REFUND
HUMAN_APPROVAL
REJECT
INFORMATION_REQUIRED
```

`RiskLevel` is intentionally **not** an output. The Agent may assess risk internally and use it when determining `Decision`.

---

# 10. Agent System Prompt

```text
You are a Customer Complaint Analysis Agent for an e-commerce company.

Your responsibility is to analyze incoming customer complaints and recommend the correct refund-processing route.

You must:
1. Understand the customer's complaint.
2. Classify the complaint.
3. Assess severity and sentiment.
4. Determine whether the complaint appears eligible for a refund.
5. Determine the appropriate refund amount when sufficient information is available.
6. Assess risk internally.
7. Recommend one of the defined processing decisions.
8. Provide a concise business justification.
9. Identify missing information when a reliable decision cannot be made.

REFUND POLICY:

- Legitimate complaints involving damaged products, defective products, wrong items, missing items, or duplicate charges may be eligible for a refund.
- Refunds of ₹5,000 or less may be automatically processed when the case is suitable for automatic processing.
- Refunds above ₹5,000 require HUMAN_APPROVAL.
- High-risk or unusual cases require HUMAN_APPROVAL.
- Complaints that clearly do not qualify for a refund should be REJECTED.
- If critical information is missing and a reliable decision cannot be made, use INFORMATION_REQUIRED.
- Never invent customer, order, product, or financial information.

DECISION RULES:

AUTO_REFUND:
Use when the complaint is eligible and is suitable for automatic processing, including a refund amount of ₹5,000 or less.

HUMAN_APPROVAL:
Use when the complaint is eligible but the refund exceeds ₹5,000, the case is high-risk or unusual, or human judgment is required.

REJECT:
Use when the complaint clearly does not qualify for a refund.

INFORMATION_REQUIRED:
Use when essential information is missing and a reliable decision cannot be made.

IMPORTANT:

The Agent only analyzes and recommends a decision.

The Agent must NOT:
- send emails
- issue refunds
- modify customer records
- approve its own human-review cases
- invent missing information

The Decision output must contain exactly one of:
AUTO_REFUND
HUMAN_APPROVAL
REJECT
INFORMATION_REQUIRED

Do not add explanations, punctuation, or other values to the Decision field.

Return the requested structured outputs and keep Reason concise and business-readable.
```

---

# 11. Agent User Prompt

Use Studio Web's Insert/variable picker for the inputs.

```text
Analyze the following customer complaint.

Customer Name:
@CustomerName

Customer Email:
@CustomerEmail

Email Subject:
@EmailSubject

Complaint:
@ComplaintText

Order ID:
@OrderId

Order Amount:
@OrderAmount

Apply the refund policy defined in your system instructions.

Return the structured outputs defined for this agent.
```

---

# 12. Agent Testing

### AUTO_REFUND test

```text
Hello Customer Support,

I received my order ORD-45821 today, but the product
was damaged when I opened the package.

The order amount was ₹2499 and I would like a refund.

Regards,
John
```

Expected core output:

```text
ComplaintCategory = DAMAGED_PRODUCT
RefundEligible = true
RefundAmount = 2499
Decision = AUTO_REFUND
```

For routing, `Decision` is the authoritative output.

---

# 13. Determine Refund Route – Decision Gateway

Configure the Gateway to use the Agent's `Decision` output.

Do not route using the Agent's natural-language `Reason`.

Recommended routes:

### Route 1 – AUTO_REFUND

```text
Decision == "AUTO_REFUND"
```

### Route 2 – HUMAN_APPROVAL

```text
Decision == "HUMAN_APPROVAL"
```

### Route 3 – REJECT

```text
Decision == "REJECT"
```

### Route 4 – INFORMATION_REQUIRED

```text
Decision == "INFORMATION_REQUIRED"
```

### Default

Route unexpected values to human review rather than automatic refund.

---

# 14. AUTO_REFUND Branch

Architecture:

```text
AUTO_REFUND
     ↓
Process Refund
     ↓
Refund Result
     ↓
Reply to Email
```

The refund operation should be a separate RPA Workflow.

---

# 15. Process Refund – RPA Workflow

Create a reusable RPA Workflow named:

```text
Process Refund
```

### Inputs

```text
Customer
OrderId
RefundAmount
```

### Outputs

```text
RefundStatus
RefundTransactionId
RefundAmount
```

### Demonstration implementation

Use a mock/simulated refund operation so the business demo does not execute real financial transactions.

Example output:

```text
RefundStatus = SUCCESS
RefundTransactionId = REF-20260909-001
RefundAmount = 2499
```

The workflow can later be replaced by a real payment/ERP/API integration.

---

# 16. HUMAN_APPROVAL Branch

Use the Quick Form / human approval node.

### Recommended inputs

```text
CustomerName
OrderId
ComplaintText
AgentRefundAmount
AgentReason
```

### Recommended outputs

```text
ApprovalResult
ApprovedRefundAmount
ApprovalComments
```

### Quick Form fields

**Approval Result**

```text
Approve
Reject
```

**Approved Refund Amount**

```text
Number
```

**Approval Comments**

```text
String
```

### Approval screen

```text
Customer: Rahul Sharma
Order: ORD-78214
Complaint: Product arrived damaged.
AI Recommended Refund: ₹12,500
AI Recommendation: HUMAN_APPROVAL

[Approve / Reject]
Approved Refund Amount: [Number]
Reviewer Comments: [Text]
```

---

# 17. Effective Refund Amount After Approval

If the human can change the refund amount, use `ApprovedRefundAmount`.

Recommended expression:

```javascript
$vars.quickForm1.output?.ApprovedRefundAmount ?? $vars.autonomousAgent1.output.RefundAmount
```

Behavior:

- Human supplies an amount → use it.
- Quick Form output is null → fall back to Agent refund amount.
- Approved amount is `0` → treat `0` as a real value.

Use a variable name without spaces. Prefer:

```text
ApprovedRefundAmount
```

rather than:

```text
New Refund Amount
```

---

# 18. HUMAN_APPROVAL → Approve

Route:

```text
Quick Form
   ↓
Approve
   ↓
Process Refund
   ↓
Reply to Email
```

The RPA workflow should process the effective approved amount.

---

# 19. HUMAN_APPROVAL → Reject

Route:

```text
Quick Form
   ↓
Reject
   ↓
Reply to Email
```

Do not call the refund workflow on this path.

### Subject

```text
RE: {{$vars.EmailSubject}}
```

### Body

```text
Dear {{$vars.CustomerName}},

We have carefully reviewed your complaint regarding order {{$vars.OrderId}}.

After reviewing the available information, we are unable to approve the requested refund at this time.

Review comments:
{{$vars.quickForm1.output?.ApprovalComments ?? "The request did not meet the applicable refund criteria."}}

If you believe additional information may help us reconsider the case, please reply to this email with the relevant details.

Regards,
Customer Support Team
```

---

# 20. REJECT Branch

Route:

```text
Agent
 ↓
REJECT
 ↓
Customer Email
```

### Subject

```text
RE: {{$vars.EmailSubject}}
```

### Body

```text
Dear {{$vars.CustomerName}},

We have carefully reviewed your complaint regarding order {{$vars.OrderId}}.

After reviewing the available information, we are unable to approve the requested refund based on our current refund policy.

Reason:
{{$vars.Reason}}

If you believe additional information may help us reconsider the case, please reply to this email with the relevant details.

Regards,
Customer Support Team
```

Avoid exposing internal AI implementation details to the customer.

---

# 21. INFORMATION_REQUIRED Branch

Route:

```text
Agent
 ↓
INFORMATION_REQUIRED
 ↓
Request Information Email
```

The Agent should populate `MissingInformation`, for example:

```text
Order ID and order amount
```

### Subject

```text
RE: {{$vars.EmailSubject}} - Additional Information Required
```

### Body

```text
Dear {{$vars.CustomerName}},

We have received your complaint and need some additional information before we can process your request.

Please reply to this email with the following information:

{{$vars.MissingInformation}}

Once we receive the required information, our customer support team will review the case and proceed with the appropriate resolution.

We appreciate your cooperation.

Regards,
Customer Support Team
```

---

# 22. Outlook Reply / Send Configuration

## Reply to original email

Use the Outlook reply operation when the business wants to continue the original email thread.

Conceptually:

```text
Original Message ID
Subject
Message
ReplyAll = false
```

Do **not** use `CustomerEmail` as the `EmailID` for a reply.

`CustomerEmail` is an email address; a reply operation needs the identifier of the original message.

Reference: https://docs.uipath.com/activities/other/latest/productivity/office365-email-reply-to-email-connections

## Send a new email

Use a send-email operation when the process intentionally creates a new message.

Then configure:

```text
To = CustomerEmail
Subject = ...
Body = ...
```

---

# 23. AUTO_REFUND Email

### Subject

```text
RE: {{$vars.EmailSubject}}
```

### Body

```text
Dear {{$vars.CustomerName}},

We have reviewed your complaint regarding order {{$vars.OrderId}}.

Your refund request has been approved and the refund of ₹{{$vars.RefundAmount}} has been initiated successfully.

Refund Reference:
{{$vars.ProcessRefund.output.RefundTransactionId}}

The amount will be credited according to your payment provider's normal processing timeline.

We apologize for the inconvenience caused.

Regards,
Customer Support Team
```

Use the exact output path exposed by the RPA Workflow node in your Studio Web tenant.

---

# 24. HUMAN_APPROVAL → Approve Email

### Subject

```text
RE: {{$vars.EmailSubject}}
```

### Body

```text
Dear {{$vars.CustomerName}},

We have reviewed your complaint regarding order {{$vars.OrderId}}.

Your refund request has been approved following our internal review.

Approved Refund Amount:
₹{{$vars.quickForm1.output?.ApprovedRefundAmount ?? $vars.autonomousAgent1.output.RefundAmount}}

The refund has now been initiated. You will receive the amount according to your payment provider's normal processing timeline.

Refund Reference:
{{$vars.ProcessRefund.output.RefundTransactionId}}

We apologize for the inconvenience caused and appreciate your patience.

Regards,
Customer Support Team
```

---

# 25. Case Logging

Add a final logging step for traceability.

Possible targets:

- SharePoint List
- Dataverse
- Excel Online
- CRM
- Database
- Case-management system

Recommended fields:

| Field | Example |
|---|---|
| `ComplaintId` | CMP-1001 |
| `CustomerName` | John Smith |
| `CustomerEmail` | john@example.com |
| `OrderId` | ORD-45821 |
| `ComplaintCategory` | DAMAGED_PRODUCT |
| `RefundEligible` | true |
| `RequestedRefundAmount` | 2499 |
| `ApprovedRefundAmount` | 2499 |
| `Decision` | AUTO_REFUND |
| `RefundStatus` | SUCCESS |
| `RefundTransactionId` | REF-20260909-001 |
| `FinalStatus` | Completed |
| `Reason` | Damaged product qualifies for refund |
| `ProcessedDateTime` | Runtime timestamp |

---

# 26. Final Flow Structure

```text
Email Received
      ↓
Customer Complaint Analysis Agent
      ↓
Determine Refund Route
      │
      ├────────────── AUTO_REFUND ────────────────┐
      │                                            │
      │                                            ▼
      │                                      Process Refund
      │                                            │
      │                                            ▼
      │                                      Customer Email
      │
      ├──────────── HUMAN_APPROVAL ────────────────┐
      │                                             │
      │                                             ▼
      │                                      Quick Form Approval
      │                                             │
      │                              ┌──────────────┴──────────────┐
      │                              │                             │
      │                              ▼                             ▼
      │                           Approve                       Reject
      │                              │                             │
      │                              ▼                             │
      │                        Process Refund                     │
      │                              │                             │
      │                              └──────────────┬──────────────┘
      │                                             │
      │                                             ▼
      │                                      Customer Email
      │
      ├────────────── REJECT ───────────────────────┐
      │                                             ▼
      │                                      Customer Email
      │
      ├──────── INFORMATION_REQUIRED ──────────────┐
      │                                             ▼
      │                                      Request Information
      │
      └──────────── DEFAULT ────────────────────────┐
                                                    ▼
                                               Human Review

All completed routes
        ↓
     Log Case
        ↓
       END
```

---

# 27. Test Scenario 1 — AUTO_REFUND

### Email

```text
Hello Customer Support,

I received my order ORD-45821 today, but the product arrived damaged and does not work.

The order amount was ₹2499 and I would like a refund.

Regards,
John
```

### Expected

```text
ComplaintCategory = DAMAGED_PRODUCT
RefundEligible = true
RefundAmount = 2499
Decision = AUTO_REFUND
```

### Route

```text
Agent
 ↓
AUTO_REFUND
 ↓
Process Refund
 ↓
Customer Email
 ↓
Log Case
```

---

# 28. Test Scenario 2 — HUMAN_APPROVAL → Approve

### Email

```text
Hello Customer Support,

I received my order ORD-78214, but the product arrived damaged and cannot be used.

The order amount was ₹12500 and I would like a full refund.

Please review my complaint and process the refund.

Regards,
Rahul Sharma
```

### Expected Agent result

```text
RefundEligible = true
RefundAmount = 12500
Decision = HUMAN_APPROVAL
```

### Quick Form

```text
ApprovalResult = Approve
ApprovedRefundAmount = 11000
```

### Expected route

```text
Quick Form
 ↓
Approve
 ↓
Process Refund = ₹11,000
 ↓
Customer Email
 ↓
Log Case
```

---

# 29. Test Scenario 3 — HUMAN_APPROVAL → Reject

Use the same ₹12,500 complaint.

Quick Form:

```text
ApprovalResult = Reject
ApprovalComments = Refund not approved after manual review.
```

Expected:

```text
Quick Form
 ↓
Reject
 ↓
Rejection Email
 ↓
Log Case
```

No refund should be processed.

---

# 30. Test Scenario 4 — REJECT

### Email

```text
Hello Customer Support,

I purchased order ORD-45120 about eight months ago.

The product is working perfectly, but I no longer need it and would like a full refund because I have changed my mind.

The order amount was ₹2999.

Please issue the refund.

Regards,
Amit Verma
```

### Expected

```text
ComplaintCategory = OTHER
RefundEligible = false
RefundAmount = 0
Decision = REJECT
```

---

# 31. Test Scenario 5 — INFORMATION_REQUIRED

### Email

```text
Hello Customer Support,

I received a damaged product from my recent order and I would like a refund.

The product was damaged when I opened the package, but I no longer have the order details available.

Please help me get this resolved.

Regards,
Neha
```

### Expected

```text
Decision = INFORMATION_REQUIRED
MissingInformation = Order ID and/or order amount
```

---

# 32. Test Scenario 6 — Unusual / Human Review

### Email

```text
Hello Customer Support,

I received order ORD-99382, but the product is not what I expected and I want a refund.

The order amount was ₹4500.

I have already received a replacement for this order, but I would still like the full ₹4500 refunded.

Please process this immediately.

Regards,
Vikram
```

Expected:

```text
Decision = HUMAN_APPROVAL
```

This tests human review for an unusual case rather than only for high-value refunds.

---

# 33. Business Demo Narrative

Recommended presentation sequence:

1. A customer complaint arrives by email.
2. Maestro detects the event and starts the process.
3. The Autonomous Agent analyzes the complaint and recommends a route.
4. Maestro uses deterministic routing rather than allowing the Agent to directly issue a refund.
5. Suitable low-value cases are processed automatically.
6. Higher-value or exceptional cases go to human approval.
7. The human can approve, reject, or change the refund amount.
8. The customer is notified automatically.
9. The result is logged for traceability.

### Key business message

> **AI provides the intelligence; Maestro provides the orchestration and control.**

---

# 34. Recommended Production Enhancements

After the demo is stable, consider:

- Order verification against an authoritative order system
- Attachment/evidence analysis
- Refund policy grounding through an approved knowledge source
- SLA timers and escalation
- Retry/error handling for external systems
- Duplicate complaint detection
- Fraud/risk controls
- Centralized audit logging
- Environment-specific configuration

---

# 35. Connections Summary

| Component | Connection |
|---|---|
| Email Received | Microsoft 365 Outlook |
| Customer notification | Microsoft 365 Outlook |
| Quick Form Approval | UiPath Apps / human task |
| Process Refund | RPA Workflow / future payment or ERP API |
| Case Logging | SharePoint / Dataverse / CRM / database |
| Agent | Model available in tenant |

---

# 36. Variable and Expression Conventions

Prefer names without spaces:

```text
CustomerName
CustomerEmail
EmailSubject
ComplaintText
OrderId
OrderAmount
ApprovedRefundAmount
ApprovalResult
ApprovalComments
RefundStatus
RefundTransactionId
MissingInformation
```

Prefer the Studio Web variable/Insert picker whenever possible.

For optional nested output:

```javascript
$vars.quickForm1.output?.ApprovedRefundAmount ?? $vars.autonomousAgent1.output.RefundAmount
```

Studio Web provides an Expression Editor for complex expressions and variable/property selection.

Reference: https://docs.uipath.com/studio-web/automation-cloud/latest/user-guide/configuring-agentic-process-elements

---

# 37. Error Handling

| Error | Recommended handling |
|---|---|
| No matching email | Wait for / generate matching trigger event |
| Agent failure | Retry or route to human/manual review |
| Unexpected Decision | Gateway Default → Human Review |
| Refund workflow failure | Retry + support notification + log |
| Email failure | Retry + log communication failure |
| Quick Form timeout | SLA escalation |
| Missing order details | `INFORMATION_REQUIRED` |
| External system unavailable | Retry + human escalation |

Never route an unexpected AI response directly to an automatic financial action.

---

# 38. Security and Governance

For a demonstration, use a mock refund workflow unless real financial execution has been explicitly approved.

Recommended demo pattern:

```text
Customer Email
      ↓
AI Analysis
      ↓
Mock Refund Workflow
      ↓
Simulated Refund Transaction ID
```

For production:

- Use approved connections.
- Apply least-privilege permissions.
- Restrict who can approve refunds.
- Log financial actions.
- Protect customer personal information.
- Use human approval for configured thresholds/exceptions.
- Keep the Agent away from direct financial side effects unless explicitly designed and governed.

---

# 39. Final Acceptance Criteria

- [ ] Email can start the Maestro process.
- [ ] Complaint text reaches the Autonomous Agent.
- [ ] Agent outputs are structured and populated.
- [ ] `Decision` uses only approved values.
- [ ] Gateway routes correctly for every decision.
- [ ] Automatic refund path executes a mock refund.
- [ ] Human approval displays relevant information.
- [ ] Human reviewer can change the refund amount.
- [ ] Human approval result controls the next step.
- [ ] Reject path does not issue a refund.
- [ ] Information-required path asks for missing information.
- [ ] Customer notification is generated dynamically.
- [ ] Original email is correctly referenced when replying.
- [ ] Case outcome is logged.
- [ ] Unexpected Agent decisions cannot produce an automatic refund.

---

# 40. Solution Principle

```text
             AI
              |
              v
      Analyze + Recommend
              |
              v
           Maestro
              |
              v
       Enforce + Orchestrate
              |
         +----+----+
         |         |
         v         v
      Automation Human
         |         |
         +----+----+
              |
              v
           Outcome
              |
              v
        Notification
              |
              v
           Audit Log
```

The Agent **recommends**.

Maestro **orchestrates and controls**.

The refund system **executes**.

The human **handles approvals and exceptions**.

The communication layer **notifies the customer**.

The logging layer **preserves traceability**.
