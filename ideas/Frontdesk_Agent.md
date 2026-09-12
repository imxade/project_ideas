## Product Idea

AI Front Desk is a conversational coordination layer between a **service provider** and their **customers**.

Instead of behaving like a traditional customer-support chatbot that only answers FAQs, the system maintains operational context, has controlled access to service records and schedules, asks the service provider for missing information when necessary, and can execute predefined actions such as booking, rescheduling, notifications, reminders, and broadcasts.

The initial target market is **doctors and clinics**, where front-desk operations represent a significant recurring cost.

For healthcare deployments, the system will intentionally operate only on **service and scheduling records, not medical records**.

---

## Core Principle

The AI itself should never have unrestricted database access.

For every conversation, it receives only the minimum data required for the current user and operation.

The bot behaves primarily as an interface and reasoning layer. Actual changes are performed through restricted, deterministic backend functions.

For example:

Customer:

> Can you move my appointment to Friday afternoon?

The system identifies the customer from the verified contact number, retrieves only that customer's relevant appointments and the provider's available slots, presents valid alternatives, and invokes a restricted rescheduling operation after confirmation.

The model never receives arbitrary access to other customers or unrelated database records.

---

# User Roles

## Service Provider

Initially, the service provider may be a doctor or clinic.

A provider can:

* Define working days.
* Define working hours.
* Add or remove availability.
* Mark themselves unavailable for a period.
* Ask questions about their service schedule.
* Broadcast messages to selected or eligible customers.
* View numbers marked as spam.
* Mark a customer/contact as spam.
* Remove a number from the spam list.
* Define future reminder intervals for customers.

A provider **cannot directly reschedule individual customer appointments**.

Instead, the provider modifies their own availability.

If that availability change affects appointments, the system contacts the affected customers and lets them select another available slot.

---

## Customer

A customer can:

* Ask about provider availability.
* Book an appointment.
* Reschedule their own appointment.
* Cancel an appointment where permitted.
* View their own service/appointment records.
* Receive notifications.
* Confirm attendance.
* Respond to provider messages.
* Receive recurring or future service reminders.

The customer can only interact with information associated with their verified contact identity.

---

# Scheduling Model

Provider availability and customer appointments are deliberately separated.

The provider controls:

**Availability**

The customer controls:

**Appointments within that availability**

Example:

A doctor originally works:

Monday
Wednesday
Friday

The doctor changes the schedule to:

Monday–Friday, 10:00 AM–5:00 PM.

The system updates provider availability.

If instead the doctor removes Wednesday from their schedule, the system detects appointments affected by the change.

Affected customers receive a message such as:

> Your appointment on Wednesday is affected because the provider is unavailable. The next available slots are Thursday 11:00 AM, Thursday 2:30 PM, and Friday 10:00 AM.

The customer chooses a slot.

Only after confirmation does the system modify the appointment.

---

# Notifications

The system is event-driven.

Important events can generate notifications automatically.

Examples include:

### Provider availability changed

Determine which appointments are affected and contact those customers.

### Appointment created

Send confirmation to the customer.

### Appointment approaching

One day before the appointment, ask the customer to confirm attendance.

### Customer declines

Offer available alternatives and allow rescheduling.

### Routine follow-up due

If a provider recommends another visit after three months, six months, or another interval, schedule an appropriate reminder as per the provider's recommendation.

### Provider broadcast

Allow the provider to instruct the system to send a message to a defined group of customers.

---

# Context Escalation

One major distinction from conventional support bots is that the bot should not simply fail when information is missing.

If the customer asks something that the system cannot answer but the service provider can, the system can create a provider query.

For example:

Customer:

> Will the clinic be open during the holiday next month?

If no answer exists in the available records, the system can ask the provider.

The provider answers:

> Yes, but only until 1 PM.

The system can then:

1. Reply to the original customer.
2. Store the operational information.
3. Use that information when future customers ask the same question.

This creates an evolving operational knowledge base rather than a static FAQ chatbot.

Stored information should remain scoped to the service provider or organization that supplied it.

---

# Spam Management

Providers can mark contacts as spam.

Once blocked, the conversational system should no longer process normal requests from that contact.

The provider can ask:

> Show me blocked numbers.

The system retrieves the spam list through an authorized backend operation.

The provider can also say:

> Remove +91XXXXXXXXXX from spam.

The backend removes the entry and restores interaction permissions.

Blocking should preferably happen before expensive AI processing whenever possible.

---

# Data Access Model

The architecture follows **least-privilege access**.

The language model should never receive database credentials or unrestricted database queries.

Instead, it interacts with narrowly scoped backend tools such as:

`get_customer_appointments(customer_id)`

`get_provider_availability(provider_id)`

`book_appointment(customer_id, slot_id)`

`reschedule_appointment(customer_id, appointment_id, slot_id)`

`update_provider_availability(provider_id, schedule)`

`mark_spam(provider_id, contact_id)`

`remove_spam(provider_id, contact_id)`

`list_spam(provider_id)`

`create_reminder(customer_id, date, type)`

`broadcast_message(provider_id, audience, message)`

Authorization is validated by the backend rather than delegated to the AI.

The bot only sees the information returned by the permitted operation.

The LLM should therefore act as a **natural-language reasoning and tool-selection layer**, while deterministic application code remains responsible for authorization, validation, state transitions, and database modifications.

---

# Database

For Azure-hosted deployments, the primary relational database will be **Azure SQL**.

Azure SQL will store operational data such as:

* Organizations.
* Service providers.
* Customers.
* Contact identities.
* Provider availability.
* Appointments.
* Appointment status.
* Reminder schedules.
* Follow-up schedules.
* Spam and blocked-contact records.
* Provider/customer conversation metadata.
* Operational context.
* Broadcast definitions.
* Pending provider questions.
* AI configuration references.
* Audit records where required.

The LLM will never connect directly to Azure SQL.

All database access must pass through deterministic application services with explicit authorization and scope checks.

Conceptually:

```text
LLM
 │
 ▼
Tool Request
 │
 ▼
Authorization Layer
 │
 ▼
Application Service
 │
 ▼
Azure SQL
```

The application service determines the tenant, provider, customer, and permitted data scope before executing any query.

For self-hosted deployments, the database abstraction should remain sufficiently decoupled from Azure-specific infrastructure so that an equivalent supported relational database can be used when necessary.

---

# Identity

For messaging channels, the customer's verified contact identifier acts as the initial identity boundary.

Examples include:

* WhatsApp number.
* Telegram account.
* Future supported messaging identities.

The gateway maps the external identity to an internal customer identity.

The AI should not be allowed to arbitrarily specify another customer's identifier when requesting data.

Identity resolution and authorization must occur outside the LLM.

---

# Channel-Agnostic Messaging Gateway

Messaging providers should be separated from the core application.

Conceptually:

```text
WhatsApp ──┐
Telegram ──┤
SMS ───────┤
Discord ───┤
Web Chat ──┼──> Messaging Gateway
Other ─────┘
                  │
                  ▼
           Conversation Engine
                  │
          ┌───────┴────────┐
          ▼                ▼
   Authorization       AI Gateway
                           │
                           ▼
                        LiteLLM
                           │
                           ▼
                     Selected LLM
          │
          └────────┬─────────┘
                   ▼
           Deterministic Tools
                   │
                   ▼
              Data Layer
```

The initial MVP only needs:

* WhatsApp.
* Telegram.

Additional channels can later implement the same gateway interface.

The internal application should work with a normalized message format instead of depending directly on WhatsApp, Telegram, or any other messaging platform.

---

# AI Gateway

The application should not depend directly on one LLM vendor.

**LiteLLM** will act as the AI gateway between the application and the configured language model.

Conceptually:

```text
Conversation Engine
        │
        ▼
    AI Gateway
        │
        ▼
     LiteLLM
        │
        ├── Platform-provided LLM API
        │
        ├── Service provider's LLM API
        │
        ├── OpenAI-compatible endpoint
        │
        ├── Cloud-hosted model
        │
        └── Locally hosted model/API
```

The rest of the application communicates with LiteLLM rather than implementing provider-specific LLM integrations throughout the codebase.

This allows the underlying model infrastructure to be changed without changing the business logic.

---

# LLM Configuration Model

The system should support multiple ways of supplying the language model.

## Platform-Provided LLM

For the hosted SaaS version, the platform can provide a managed LLM configuration.

The service provider does not need to configure an external model.

The cost of AI inference can be incorporated into the SaaS pricing or usage limits.

---

## Bring Your Own LLM API

A service provider can provide their own LLM API credentials.

This may include supported commercial or private inference providers.

The service provider can select from the model options available through their configured endpoint.

The application itself should not need to know the implementation details of each provider.

LiteLLM handles the standardized interface.

---

## Custom or Local LLM Endpoint

A self-hosted deployment can configure its own LLM endpoint.

For example:

```text
http://localhost:8000/v1
```

or an endpoint available elsewhere on the organization's network.

This allows organizations to run their own model infrastructure and use AI Front Desk without sending inference requests through a platform-managed LLM service.

The configured endpoint may point to:

* A locally running model.
* An OpenAI-compatible inference server.
* A model running on another machine within the organization.
* A private cloud endpoint.
* A supported external LLM provider.

---

# AI Configuration Isolation

LLM configuration should be scoped to the organization or service provider.

One organization's API credentials, endpoint configuration, models, usage information, or requests must never become available to another organization.

LLM credentials should be stored securely and should never be exposed to customers or included directly in prompts.

The conversation engine should refer to an internal AI configuration identifier rather than handling raw provider credentials for each request.

Conceptually:

```text
Organization
     │
     ▼
AI Configuration
     │
     ├── Endpoint
     ├── Provider
     ├── Model
     ├── Credentials Reference
     └── Configuration Options
              │
              ▼
           LiteLLM
```

---

# Model Independence

Business logic must not depend on a particular model.

A model should be replaceable without changing scheduling, authorization, notification, or database logic.

For example, the same backend operation:

```text
reschedule_appointment(...)
```

should work regardless of whether the request was interpreted by:

* A platform-provided model.
* A service provider's commercial API.
* A self-hosted open model.
* A local OpenAI-compatible model server.

The model interprets the request and proposes an allowed tool invocation.

The backend determines whether the invocation is valid and authorized.

---

# Serverless Architecture

The hosted SaaS should use a **serverless, event-driven architecture**.

A permanently running application server for conversational workflows should not be required.

Azure Functions will be used for bounded backend execution in Azure deployments.

Incoming messages become events.

A simplified flow is:

```text
Incoming Message
      ↓
Channel Webhook
      ↓
Azure Function / Serverless Handler
      ↓
Normalize Message
      ↓
Authenticate / Resolve Identity
      ↓
Load Required Context
      ↓
LiteLLM / Configured LLM
      ↓
AI Decision
      ↓
Restricted Action
      ↓
Azure SQL / Event
      ↓
Outbound Notification
```

Each request should be designed as a bounded operation.

One incoming message should not become a long-running persistent process.

Long-running workflows should instead be divided into multiple events and serverless executions.

For example, notifying 500 customers should not require one function invocation to stay alive while all messages are sent.

Instead:

```text
Provider changes schedule
        ↓
ScheduleChanged Event
        ↓
Find affected appointments
        ↓
One notification event per customer
        ↓
Independent serverless executions
```

This keeps each execution short and makes the hosted system resilient to serverless execution limits.

---

# Scheduled Jobs

Scheduled and time-dependent workflows will be implemented using **timer-triggered Azure Functions** in Azure deployments.

Instead of keeping a worker, scheduler, cron server, or bot process continuously running, Azure Functions can execute at predefined intervals and discover jobs that have become due.

Examples include:

* Appointment reminders.
* Appointment confirmation requests.
* Routine follow-up reminders.
* Periodic service reminders.
* Deferred notifications.
* Expired pending actions.
* Scheduled broadcasts.
* Retry jobs.
* Cleanup and maintenance tasks where appropriate.

A timer-triggered function can periodically query Azure SQL for due work.

For example:

```text
Timer Trigger
      ↓
Azure Function
      ↓
Query Azure SQL
for due reminders
      ↓
Create notification events
      ↓
Independent message delivery
```

The scheduled function should preferably identify due jobs and enqueue or emit events rather than attempting to perform every downstream operation inside the same execution.

For example:

```text
Every N Minutes
      ↓
Reminder Scheduler Function
      ↓
SELECT due reminders
FROM Azure SQL
      ↓
ReminderDue Events
      ↓
One serverless execution
per notification
```

This maintains the event-driven model while avoiding a persistent scheduler service.

---

# Appointment Reminder Example

An appointment may contain a reminder timestamp stored in Azure SQL.

Example:

```text
Appointment
├── appointment_time
├── reminder_time
├── confirmation_status
└── reminder_status
```

A timer-triggered Azure Function periodically checks for reminders where:

```text
reminder_time <= current_time
AND reminder_status = pending
```

The scheduler then emits a notification event.

The actual WhatsApp or Telegram delivery can happen in another function execution.

```text
Timer Function
      ↓
Find due reminders
      ↓
ReminderDue
      ↓
Messaging Function
      ↓
WhatsApp / Telegram
      ↓
Update status in Azure SQL
```

This prevents the scheduler itself from becoming a long-running process.

---

# Routine Follow-Up Example

Suppose a doctor recommends that a patient return after six months.

The application stores a future follow-up date.

```text
FollowUp
├── customer_id
├── provider_id
├── due_at
├── status
└── reason/category
```

When the scheduled time arrives:

```text
Timer Trigger
      ↓
Find due follow-ups
      ↓
FollowUpDue Event
      ↓
Customer Notification
      ↓
Offer provider availability
      ↓
Customer can book appointment
```

The reminder record contains only operational information and does not require the bot to access medical records.

---

# Background and Deferred Work

Operations that do not need to complete during the original request should become events.

Examples include:

* Sending appointment reminders.
* Processing provider broadcasts.
* Contacting customers affected by schedule changes.
* Sending routine follow-up reminders.
* Asking providers unresolved customer questions.
* Returning provider answers to waiting customers.
* Retrying failed outbound messages.
* Processing scheduled jobs discovered by timer-triggered functions.

For example:

```text
Customer Question
       │
       ▼
Information Missing
       │
       ▼
ProviderQuestionCreated
       │
       ▼
Provider Receives Question

        ...later...

Provider Reply
       │
       ▼
ProviderQuestionAnswered
       │
       ▼
Operational Context Updated
       │
       ▼
Customer Notification
```

This prevents conversational requests from becoming unnecessarily long-lived.

---

# Infrastructure Direction

Primary stack:

* Next.js.
* DaisyUI.
* Docker.
* Terraform.
* LiteLLM.
* floci-az.
* Azure.
* Azure Functions.
* Timer-triggered Azure Functions.
* Azure SQL.
* Event-driven/serverless services.

Infrastructure should be reproducible using Terraform wherever applicable.

The application should remain sufficiently decoupled from Azure that the same core product can operate either on Azure infrastructure or independently through a local/self-hosted environment.

---

# floci-az

**floci-az** is an important part of the infrastructure strategy.

It is not only intended for testing an Azure deployment.

It will also be used for:

* Local development.
* Local validation.
* Validation of Azure-oriented infrastructure and application behavior.
* Running the system without Azure.
* Local hosting.
* Self-hosted deployments.
* Testing production-oriented workflows before deployment.

The architecture should therefore avoid treating floci-az merely as a temporary development emulator.

Instead, it forms part of the project's deployment abstraction.

Conceptually:

```text
                  AI Front Desk
                       │
                Application Layer
                       │
                Infrastructure APIs
                  ┌────┴────┐
                  │         │
                  ▼         ▼
              floci-az     Azure
                  │         │
                  ▼         ▼
            Local/Self     Hosted
              Hosted        SaaS
```

The same application should be capable of operating against either environment with minimal environment-specific changes.

Where Azure uses components such as Azure Functions and Azure SQL, the local/self-hosted environment should provide compatible or abstracted equivalents so that core application logic does not require rewriting.

---

# Azure Deployment

Azure will be used for testing and validating the actual hosted production deployment.

The Azure-hosted architecture will include:

* Azure Functions for serverless execution.
* Timer-triggered Azure Functions for scheduled jobs.
* Azure SQL for relational operational data.
* Event-driven processing for asynchronous workflows.
* Messaging integrations for supported channels.
* LiteLLM for AI provider abstraction.

The project will initially prioritize Azure services with persistent free-tier availability where possible.

The architecture should remain:

* Serverless.
* Event-driven.
* Cost-conscious.
* Horizontally distributable.
* Independent of continuously running application servers wherever practical.

The Azure Student offering can be used during development and deployment validation.

However, Azure should remain **a supported deployment target rather than a mandatory runtime dependency for the entire product**.

---

# Azure-Independent Operation

A deployment should also be capable of running without Azure.

A service provider or organization should be able to host the system within its own infrastructure using:

* Docker.
* floci-az.
* Local or private infrastructure.
* A compatible relational database.
* A locally accessible LLM endpoint.
* Their own messaging and networking configuration.

This allows the product to support environments where organizations:

* Do not want to use Azure.
* Require local deployment.
* Need greater control over their infrastructure.
* Want to use locally hosted models.
* Want to keep operational traffic within their own network.

The application architecture should abstract Azure-specific services sufficiently that equivalent local implementations can replace:

* Azure Functions.
* Timer-triggered scheduling.
* Azure SQL.
* Event infrastructure.

without changing the higher-level business rules.

---

# Deployment Model

Two primary distribution models are planned.

## Hosted SaaS

The project operates the infrastructure and organizations subscribe to the service.

The hosted version may provide:

* Azure-hosted infrastructure.
* Azure Functions.
* Azure SQL.
* Scheduled timer-triggered functions.
* Updates.
* Managed messaging integrations.
* Managed AI configuration.
* Platform-provided LLM access.
* Operational monitoring.
* Managed deployment.

This provides the project's sustainable revenue model.

A service provider may still optionally configure their own LLM API instead of using the platform-provided model.

---

## Self-Hosted

Organizations can deploy the system within infrastructure they control.

The self-hosted version may use:

* Docker.
* Terraform where applicable.
* floci-az.
* A compatible relational database.
* Local scheduling equivalents.
* The organization's own network.
* The organization's own LLM API.
* A locally hosted LLM endpoint.

A self-hosted installation should not inherently require Azure.

Business logic, authorization, scheduling, AI integration, and messaging adapters should therefore avoid unnecessary dependencies on proprietary cloud infrastructure.

---

# Deployment and AI Independence

Infrastructure choice and AI-provider choice should be independent.

For example, all of the following configurations should be possible:

```text
Azure Deployment
+ Azure SQL
+ Azure Functions
+ Platform LLM
```

```text
Azure Deployment
+ Azure SQL
+ Azure Functions
+ Service Provider's LLM API
```

```text
Azure Deployment
+ Private LLM Endpoint
```

```text
floci-az / Self-Hosted Deployment
+ External LLM API
```

```text
floci-az / Self-Hosted Deployment
+ Local LLM
```

This creates two independent configuration dimensions:

```text
Infrastructure
├── Azure
│   ├── Azure Functions
│   ├── Timer-triggered Functions
│   └── Azure SQL
│
└── Local / Self-Hosted / floci-az

AI
├── Platform LLM
├── Customer LLM API
└── Local / Private LLM API
```

Neither decision should unnecessarily constrain the other.

---

# High-Level Architecture

```text
                     CUSTOMER / PROVIDER
                              │
             ┌────────────────┴────────────────┐
             │                                 │
         WhatsApp                          Telegram
             │                                 │
             └────────────────┬────────────────┘
                              ▼
                    Messaging Gateway
                              │
                              ▼
                    Identity Resolution
                              │
                              ▼
                      Authorization
                              │
                              ▼
                  Conversation Orchestrator
                       ┌──────┴──────┐
                       │             │
                       ▼             ▼
                  Context Layer   AI Gateway
                                    │
                                    ▼
                                  LiteLLM
                                    │
                   ┌────────────────┼────────────────┐
                   │                │                │
                   ▼                ▼                ▼
              Platform API     Provider API     Local API
                   │                │                │
                   └────────────────┴────────────────┘
                                    │
                                    ▼
                              Tool Decision
                                    │
                                    ▼
                         Deterministic Backend
                                    │
          ┌─────────────────────────┼─────────────────────────┐
          │                         │                         │
          ▼                         ▼                         ▼
     Scheduling                Notifications               Records
          │                         │                         │
          └─────────────────────────┼─────────────────────────┘
                                    ▼
                                Azure SQL
                                    │
                 ┌──────────────────┴──────────────────┐
                 │                                     │
                 ▼                                     ▼
             Events                         Scheduled Job Records
                 │                                     │
                 ▼                                     ▼
          Azure Functions                  Timer-triggered
                                             Functions
```

The same application architecture should be usable in both Azure-hosted and floci-az/self-hosted environments.

---

# Request Lifecycle

A typical customer request should follow a structure similar to:

```text
1. Receive message
        ↓
2. Normalize channel-specific payload
        ↓
3. Resolve organization
        ↓
4. Resolve user identity
        ↓
5. Check spam/block status
        ↓
6. Determine authorization scope
        ↓
7. Retrieve minimum required context from Azure SQL
        ↓
8. Resolve organization's AI configuration
        ↓
9. Send request through LiteLLM
        ↓
10. LLM determines intent/tool request
        ↓
11. Backend validates requested operation
        ↓
12. Execute deterministic operation
        ↓
13. Persist state/event in Azure SQL
        ↓
14. Generate response
        ↓
15. Send through original messaging channel
```

At no point should the LLM itself become the authority for identity, authorization, database integrity, or scheduling state.

---

# Scheduled Job Lifecycle

A scheduled workflow should follow a different path from an interactive request.

```text
1. Timer-triggered Azure Function executes
        ↓
2. Query Azure SQL for due jobs
        ↓
3. Claim/mark jobs for processing
        ↓
4. Emit one event per due operation
        ↓
5. Independent Azure Function handles event
        ↓
6. Perform deterministic action
        ↓
7. Send notification if required
        ↓
8. Persist completion/status to Azure SQL
```

This prevents one scheduler invocation from becoming responsible for an entire batch of long-running work.

Scheduled execution should remain idempotent where possible so that retries do not result in duplicate reminders or duplicate state changes.

---

# MVP

The first meaningful version should focus on a narrow workflow rather than attempting to replace an entire receptionist immediately.

The MVP consists of:

## Provider

* Configure availability.
* Change availability.
* View schedule.
* Mark/unmark spam contacts.
* View spam contacts.
* Broadcast messages.
* Respond to escalated customer questions.
* Define follow-up reminder periods.

## Customer

* View available appointments.
* Book appointments.
* Reschedule their own appointments.
* Cancel where permitted.
* Confirm upcoming appointments.
* Ask operational questions.
* Receive provider responses.

## Automation

* Appointment confirmation.
* One-day-before appointment reminder.
* Appointment confirmation request.
* Provider schedule-change notifications.
* Rescheduling workflows for affected appointments.
* Periodic follow-up reminders.
* Timer-triggered scheduled jobs.
* Deferred provider-question workflows.
* Provider broadcasts.
* Retry processing for failed notifications.

## Channels

* Telegram.
* WhatsApp.

## AI

* LiteLLM gateway.
* Platform-managed LLM option.
* Bring-your-own-LLM API option.
* Custom/local LLM endpoint option.
* Natural-language intent understanding.
* Clarification questions.
* Authorized context retrieval.
* Selection of permitted backend operations.
* Provider escalation when information is missing.

## Database

* Azure SQL for hosted deployments.
* Relational storage for appointments, schedules, reminders, identities, context, and operational state.
* No direct LLM-to-database access.
* Tenant-scoped deterministic data access.

## Infrastructure

* Serverless/event-driven hosted architecture.
* Azure Functions.
* Timer-triggered Azure Functions.
* Azure SQL.
* Azure deployment support.
* floci-az local validation.
* floci-az local/self-hosted operation.
* Docker-based deployment.
* Terraform-managed infrastructure where applicable.
* No mandatory Azure dependency for self-hosting.

---

# What the AI Must Not Do

The AI must not:

* Access medical records.
* Diagnose a patient.
* Provide medical advice as part of the front-desk system.
* Read records belonging to unrelated customers.
* Execute arbitrary database queries.
* Connect directly to Azure SQL.
* Modify provider availability on behalf of a customer.
* Modify another customer's appointment.
* Reschedule an individual customer's appointment because the provider requested it.
* Invent availability.
* Claim an operation succeeded before the backend confirms it.
* Circumvent spam or authorization rules.
* Determine its own authorization scope.
* Select arbitrary customer identities to access.
* Receive database credentials.
* Receive other organizations' LLM credentials.
* Expose LLM API credentials to customers.
* Bypass deterministic backend validation.
* Treat generated text as confirmation that a state-changing operation succeeded.

The application backend remains the authority for permissions and state.

---

# Architectural Principles

The project should follow several core architectural principles:

1. **Least privilege**
   Every request receives only the data required to perform that operation.

2. **Deterministic authorization**
   Permissions are enforced by application code rather than natural-language reasoning.

3. **Model independence**
   LiteLLM separates application logic from the underlying language model provider.

4. **Infrastructure independence**
   Azure is a supported hosted target, while floci-az enables local and self-hosted operation.

5. **Serverless execution**
   Interactive and asynchronous work is decomposed into short, bounded function executions.

6. **Event-driven workflows**
   Long-running operations are represented as events rather than persistent processes.

7. **Scheduled serverless execution**
   Time-based workflows use timer-triggered Azure Functions rather than continuously running schedulers.

8. **Relational operational state**
   Azure SQL acts as the primary hosted source of truth for schedules, appointments, reminders, identity relationships, and operational records.

9. **Channel independence**
   Telegram and WhatsApp are adapters around a common messaging interface.

10. **Provider/customer separation**
    Providers control availability; customers control their appointments within that availability.

11. **Minimal healthcare scope**
    The initial healthcare implementation manages service operations and scheduling, not medical records or clinical decision-making.

12. **AI as an interface, not an authority**
    The LLM understands language and proposes actions, while deterministic services control what actually happens.

---

# Product Positioning

The product is not primarily:

> A chatbot for answering customer questions.

It is:

> An AI-operated front desk that coordinates customers and service providers through existing messaging platforms while executing tightly controlled operational workflows.

Its advantage is the combination of:

**Conversation + operational context + scheduling + controlled actions + provider escalation + proactive notifications + scheduled automation + deployment independence + model independence.**

Instead of ending the conversation when information is unavailable, the system can obtain the missing information from the service provider and continue the workflow.

Instead of keeping persistent processes alive for reminders and scheduled work, timer-triggered serverless functions can periodically discover due work from Azure SQL and dispatch independent executions.

Instead of binding the product to one commercial AI vendor, LiteLLM allows the platform or the service provider to supply the model infrastructure.

Instead of binding the product to Azure, floci-az allows the same architecture to be validated, operated, and self-hosted locally.

This makes the system a persistent, deployable intermediary between both parties rather than another FAQ interface.
