# Supplier Onboarding -- UiPath Maestro Flow

## Overview

This project demonstrates an end-to-end **Supplier Onboarding process**
orchestrated using **UiPath Maestro Flow**.The process starts when a supplier onboarding email is received. The latest email is retrieved and analyzed by an AI Agent. Based on the supplier's annual spend and the configured procurement threshold, the request follows either an automatic approval path or a manager approval
path.
Approved requests are then sent to an enterprise procurement system through an API. The API response is checked, and the flow either completes the onboarding or routes the request to an integration exception path.

------------------------------------------------------------------------

## Business Scenario

A supplier sends an onboarding request by email.

The process needs to:

1.  Detect that a supplier onboarding email has been received.
2.  Retrieve the newest email.
3.  Analyze the supplier request using an AI Agent.
4.  Check the supplier's annual spend against the procurement threshold.
5.  Automatically approve requests within the threshold.
6.  Send requests requiring additional approval to a manager.
7.  Send approval or rejection notifications.
8.  Call the enterprise procurement system through an API.
9.  Check the API status code.
10. Complete supplier onboarding when the integration succeeds.
11. Handle integration failures using retry or escalation.

------------------------------------------------------------------------

## End-to-End Flow

``` text
Email Received
      |
      v
Get Newest Email
      |
      v
Supplier Request Analyzer
      |
      v
Spend Threshold Check
      |
      +----------------------------+
      |                            |
     True                        False
      |                            |
      v                            v
Manager Approval             Auto Approval Email
      |                            |
   +--+--+                         |
   |     |                         |
Approve Reject                      |
   |     |                         |
   v     v                         |
Send     Send                       |
Approval Rejection                  |
Email    Email                      |
   |       |                        |
   |       +------------------------+
   |
   v
Enterprise System Response Check
(API Calling)
   |
   v
Check API Status Code
   |
   +--------------------+
   |                    |
  True                 False
   |                    |
   v                    v
Supplier Onboarding   Supplier
Completed             Integration
   |                  Exception
   v                    |
  End              Retry / Escalate
                        |
                        v
              Supplier Onboarding
                   Exception
```

------------------------------------------------------------------------

## Flow Components

1. Email Received

The flow starts when a supplier onboarding email is received.

This makes the process event-driven rather than requiring a user to
manually start the flow.

2. Get Newest Email

Retrieves the newest supplier onboarding email that will be processed.

3. Supplier Request Analyzer

An AI Agent analyzes the supplier request and extracts relevant supplier
information.

The information used in the process includes:

-   Supplier Name
-   Supplier Type
-   Annual Spend
-   Payment Terms

4. Spend Threshold Check

The flow evaluates the supplier's annual spend against the configured procurement threshold. The decision determines whether the request can follow the automatic
approval path or requires manager approval.

5. Manager Approval

Requests that require additional approval are sent to a manager.
The manager has two possible outcomes:

**Approve** - Send Approval Email - Continue to the enterprise-system
integration

**Reject** - Send Rejection Email

6. Auto Approval Email

Requests that fall within the automatic approval path are notified through the Auto Approval Email step. The flow then continues toward the enterprise procurement system integration.

7. Send Approval Email

Notifies the relevant recipient that the supplier request has been approved by the manager.

8. Send Rejection Email

Notifies the relevant recipient that the supplier request has been rejected.

9. Enterprise System Response Check -- API Calling

The flow calls the enterprise procurement system using an API. This represents the integration step where the approved supplier is processed by the enterprise system.

10. Check API Status Code

The response from the enterprise system is evaluated to determine whether the integration was successful.

An example condition used in the flow is:

``` text
$vars.httpRequest1.output?.code == 200
```

A successful response continues to **Supplier Onboarding Completed**.

An unsuccessful response follows the exception path.

11. Supplier Onboarding Completed

Represents the successful completion of supplier onboarding after the enterprise-system integration succeeds.

The flow then reaches **End**.

12. Supplier Integration Exception

Handles an unsuccessful enterprise-system integration. The exception path provides two possible actions:

-   **Retry** -- attempt the integration again.
-   **Escalate** -- move the issue toward exception handling.

13. Supplier Onboarding Exception

Represents the exception outcome when the supplier integration cannot be completed successfully.

------------------------------------------------------------------------

## Supplier Information

The supplier request contains information such as:

  Field              Description
  ------------------ -------------------------------------------
  Supplier Name      Name of the supplier
  Supplier Type      Type/category of supplier
  Annual Spend       Expected annual supplier spend
  Payment Terms      Requested payment terms
  Manager Comments   Comments provided during manager approval

Some fields may be optional, so expressions should safely handle null
values where required.

------------------------------------------------------------------------

## Approval Logic

The business logic can be represented as:

``` text
Spend Threshold Check

IF request is within the defined threshold
    → Auto Approval Email
ELSE
    → Manager Approval
```

For manager approval:

``` text
Manager Approves
    → Send Approval Email
    → Enterprise System API
    → Check API Status Code
    → Completed / Exception
```

For rejection:

``` text
Manager Rejects
    → Send Rejection Email
```

------------------------------------------------------------------------

## Integration Logic

After the approval path, the flow reaches:

``` text
Enterprise System Response Check
            |
            v
    Check API Status Code
          /       \
       True       False
        |           |
        v           v
   Completed    Integration
                  Exception
                  /      \
               Retry    Escalate
```

Example successful HTTP response condition:

``` text
$vars.httpRequest1.output?.code == 200
```

------------------------------------------------------------------------

## Testing and Debugging

The Maestro Flow Editor provides several useful areas for testing and
troubleshooting.

### Executions

Shows individual flow executions and their status.

You can select a run to inspect what happened during that execution.

### Execution Trace

Shows the sequence of activities/nodes that executed and the time taken
by each step.

This is particularly useful during the demo to explain how the request
moves through the orchestration.

### Variables

Allows inspection of values available during an execution.

### Incidents

Shows errors or incidents that occurred during execution.

### Datasets

Provides test data that can be used for evaluation/testing scenarios.

### Evaluators

Provides evaluation capabilities for configured evaluation scenarios.

------------------------------------------------------------------------

## Connections

The flow uses configured connections for capabilities such as:

-   Email
-   Enterprise system/API integration
-   UiPath GenAI / AI capabilities

Before deployment, the required connections should be authenticated and
available in the target environment.

------------------------------------------------------------------------

## Deployment

The flow is authored in the **UiPath Maestro VS Code experience** and
can be packaged and deployed to an Orchestrator target.

The demo package version is:

``` text
1.0.8
```

Typical deployment process:

1.  Validate the Flow.
2.  Build/package the project.
3.  Select the required Orchestrator tenant/folder/target.
4.  Publish/deploy the Flow.
5.  Verify the deployed version.
6.  Run the deployed Flow.
7.  Review the execution in Maestro/Orchestrator.

> Deployment configuration depends on the target Orchestrator
> environment and the connections configured there.

------------------------------------------------------------------------

## Technologies / UiPath Capabilities

-   UiPath Maestro Flow
-   UiPath AI Agents
-   Email integration
-   REST/API integration
-   Orchestrator
-   VS Code
-   Human approval/task handling

------------------------------------------------------------------------

## Project Structure

``` text
SupplierOnboarding_MaestroFlow/
│
├── SupplierOnboarding_MaestroFlow.flow
├── project configuration files
└── supporting resources
```

------------------------------------------------------------------------

## Notes

This project is a demonstration/reference implementation for a supplier onboarding scenario. Connection names, email addresses, API endpoints, procurement
thresholds, and other environment-specific configuration should be reviewed and configured for the target environment before production use.
