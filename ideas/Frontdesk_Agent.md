# AI Front Desk

## Product Idea

AI Front Desk is a conversational coordination layer between a **service provider** and their **customers**.

Instead of behaving like a traditional customer-support chatbot that only answers FAQs, the system maintains operational context, has controlled access to service records and schedules, asks the service provider for missing information when necessary, and can execute predefined actions such as booking, rescheduling, notifications, reminders, and broadcasts.

The initial target market is **doctors and clinics**, where front-desk operations represent a significant recurring cost.

For healthcare deployments, the system will intentionally operate only on **service and scheduling records, not medical records**.

---

# Core Principle

The AI itself should never have unrestricted database access.

For every conversation, it receives only the minimum data required for the current user and operation.

The bot behaves primarily as an interface, reasoning layer, and tool-selection layer. Actual state changes are performed through restricted, deterministic backend functions.

For example:

Customer:

> Can you move my appointment to Friday afternoon?

The system:

1. Identifies the customer from their verified messaging identity.
2. Retrieves only that customer's relevant appointments.
3. Retrieves the provider's available slots.
4. Presents appropriate alternatives.
5. Receives confirmation from the customer.
6. Invokes a restricted rescheduling operation.
7. Confirms success only after the database operation succeeds.

The model never receives arbitrary access to other customers or unrestricted database records.

---

# User Roles

## Service Provider

Initially, the service provider may be a doctor or clinic.

A provider can:

- Define working days.
- Define working hours.
- Add or remove availability.
- Mark themselves unavailable for a period.
- Ask questions about their service schedule.
- Request their schedule or appointments conversationally.
- Broadcast messages to selected or eligible customers.
- View numbers marked as spam.
- Mark a customer/contact as spam.
- Remove a number from the spam list.
- Define future reminder intervals for customers.
- Answer customer questions escalated by the AI.
- Provide operational information that can be reused for future customer questions.

A provider **cannot directly reschedule individual customer appointments**.

Instead, the provider modifies their own availability.

If that availability change affects appointments, the system contacts the affected customers and lets them select another available slot.

---

## Customer

A customer can:

- Ask about provider availability.
- Book an appointment.
- Reschedule their own appointment.
- Cancel an appointment where permitted.
- View their own service/appointment records.
- Request their upcoming appointments conversationally.
- Receive notifications.
- Confirm attendance.
- Respond to provider messages.
- Receive recurring or future service reminders.
- Ask operational questions about the service provider.

The customer can only interact with information associated with their verified identity and information that the provider has made generally available.

---

# Conversational Schedule Presentation

Providers and customers may ask for appointments, schedules, availability, or related information directly through the messaging interface.

No separate UI capability is required for this.

The AI may convert authorized database records into readable structured plain text suitable for WhatsApp, Telegram, and similar messaging platforms.

For example:

```text
Your upcoming appointments:

12 Sep 2026
- 10:30 AM — Confirmed
- 2:00 PM — Awaiting confirmation

14 Sep 2026
- 11:00 AM — Confirmed
```

A provider may receive:

```text
Schedule — 12 Sep 2026

09:00 AM — Available
10:00 AM — Booked
11:00 AM — Booked
12:00 PM — Available
01:00 PM — Break
02:00 PM — Available
```

Formatting is performed by the AI.

Retrieval, authorization, and filtering of the underlying records remain deterministic backend responsibilities.

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

## Provider Availability Changed

Determine which appointments are affected and contact those customers.

## Appointment Created

Send confirmation to the customer.

## Appointment Approaching

One day before the appointment, ask the customer to confirm attendance.

## Customer Declines

Offer available alternatives and allow rescheduling.

## Routine Follow-Up Due

If a provider recommends another visit after three months, six months, or another interval, schedule an appropriate reminder according to the provider's recommendation.

## Provider Broadcast

Allow the provider to instruct the system to send a message to a defined group of customers.

---

# Context Escalation

One major distinction from conventional support bots is that the bot should not simply fail when information is missing.

If a customer asks something that the system cannot answer from existing authorized information, but the service provider can answer it, the system creates a provider query.

For example:

Customer:

> Will the clinic be open during the holiday next month?

If no answer exists in the available operational knowledge, the system asks the provider.

The provider answers:

> Yes, but only until 1 PM.

The system can then:

1. Reply to the original customer.
2. Store the provider's response as operational knowledge.
3. Generate an embedding for semantic retrieval.
4. Use that information when appropriate future questions are asked.

This creates an evolving operational knowledge base rather than a static FAQ chatbot.

---

# Operational Knowledge and Vector Search

Azure-hosted deployments will use **Azure SQL's native vector capabilities** to store and retrieve semantic operational knowledge.

Provider responses that may be useful in future conversations can be stored together with an embedding.

The primary use case is information that the provider supplied after the system could not answer a customer's question.

Examples include:

- Holiday opening hours.
- Service availability.
- Office procedures.
- Preparation instructions that are operational rather than medical.
- Parking information.
- Payment procedures.
- General scheduling rules.
- Whether a particular service is offered.
- Temporary operational changes.

This information can subsequently be retrieved through semantic similarity rather than requiring exact wording.

---

# Generic Provider Knowledge

If a provider's response applies generally to customers, it is stored as **provider-level operational knowledge**.

For example:

Customer:

> Are you open on Diwali?

Provider response:

> Yes, but only from 9 AM until 1 PM.

A knowledge record may conceptually contain:

```text
OperationalKnowledge
├── id
├── organization_id
├── provider_id
├── customer_id = NULL
├── content
├── embedding
├── created_at
├── updated_at
├── active
└── source/reference metadata
```

Because `customer_id` is absent, the information may be retrieved for other customers interacting with the same provider.

Semantic retrieval must still remain scoped to the organization and provider.

Conceptually:

```text
Customer Question
       ↓
Generate Query Embedding
       ↓
Azure SQL Vector Search
       ↓
FILTER organization_id
       ↓
FILTER provider_id
       ↓
FILTER customer_id IS NULL
       ↓
Closest Relevant Knowledge
       ↓
AI Response
```

---

# Customer-Specific Knowledge

Some provider responses apply only to a particular customer.

Those responses should also be available for future conversations, but **must never become general provider knowledge**.

For example:

Customer:

> Can I come 30 minutes earlier than usual tomorrow?

Provider:

> Yes, for Rahul tomorrow you can come at 9:30 AM.

That information is specific to that customer.

A corresponding record may contain:

```text
OperationalKnowledge
├── id
├── organization_id
├── provider_id
├── customer_id
├── content
├── embedding
├── created_at
├── expires_at (optional)
├── active
└── source/reference metadata
```

The customer association is stored as ordinary relational metadata.

The embedding represents the semantic meaning of the content.

The system **must not rely on the customer's name being embedded into the vector to enforce customer isolation**.

Instead, retrieval must deterministically constrain the query:

```text
organization_id = current_organization
AND provider_id = current_provider
AND (
    customer_id IS NULL
    OR customer_id = current_customer
)
```

Only after this scope is established should semantic similarity determine which permitted records are relevant.

This ensures that vector similarity can never accidentally expose another customer's context.

---

# Knowledge Retrieval Rules

Operational knowledge retrieval follows several rules.

## Customer Request

A customer's retrieval scope may include:

```text
1. Generic knowledge for the current provider
2. Knowledge specifically associated with this customer
```

It must exclude:

```text
1. Another customer's specific knowledge
2. Another provider's knowledge
3. Another organization's knowledge
```

## Provider Request

A provider may retrieve:

- Their own generic operational knowledge.
- Customer-specific knowledge where the operation and authorization allow it.
- Pending questions addressed to them.

Provider access remains subject to application authorization.

---

# Knowledge Lifecycle

Provider responses should not automatically become permanent truth forever.

Knowledge records should support metadata such as:

- Creation timestamp.
- Last modification timestamp.
- Optional expiration.
- Active/inactive status.
- Source conversation.
- Provider identity.
- Customer scope.
- Organization scope.

For temporary information such as:

> We're closed next Tuesday.

an expiration date can prevent obsolete information from being returned later.

The system may therefore combine:

**semantic relevance + scope + validity period + current state**

when retrieving knowledge.

---

# Embedding Generation

Embedding generation is separate from conversational model generation.

When reusable operational knowledge is created or modified:

```text
Provider Response
       ↓
Determine Knowledge Scope
       ↓
Generic or Customer-Specific
       ↓
Generate Embedding
       ↓
Store Text + Vector + Scope Metadata
       ↓
Azure SQL
```

The configured deployment may determine which supported embedding model or endpoint is used.

Embedding generation should remain behind the AI/model abstraction so deployments are not unnecessarily tied to a single commercial provider.

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

`search_operational_knowledge(scope, query)`

`store_operational_knowledge(scope, content)`

Authorization is validated by the backend rather than delegated to the AI.

The AI only sees the information returned by permitted operations.

The model therefore acts as a **natural-language reasoning and tool-selection layer**, while deterministic application code remains responsible for:

- Authentication.
- Authorization.
- Tenant isolation.
- Scope filtering.
- Validation.
- State transitions.
- Database modifications.
- Vector search boundaries.

---

# Database

For Azure-hosted deployments, the primary database will be **Azure SQL**.

Azure SQL acts as both:

1. The relational operational database.
2. The vector-backed operational knowledge store.

This avoids introducing a separate vector database for the initial architecture.

Azure SQL stores data such as:

- Organizations.
- Service providers.
- Customers.
- Messaging identities.
- Provider availability.
- Appointments.
- Appointment status.
- Reminder schedules.
- Follow-up schedules.
- Spam records.
- Conversation metadata.
- Operational knowledge.
- Knowledge embeddings.
- Provider questions.
- Broadcast definitions.
- AI configuration references.
- Audit records where required.

The AI does not connect directly to Azure SQL.

---

# Prisma

**Prisma** will be used as the primary application ORM and database access layer for normal relational application data.

Conceptually:

```text
Application Services
       ↓
     Prisma
       ↓
   Azure SQL
```

Prisma will handle ordinary database operations such as:

- Organizations.
- Providers.
- Customers.
- Appointments.
- Availability.
- Reminders.
- Spam state.
- Messaging identities.
- AI configuration metadata.
- Operational knowledge metadata.

Application code should preferably use Prisma rather than spreading SQL queries throughout the codebase.

---

# Prisma and Vector Operations

Azure SQL vector operations are database-specific functionality.

The architecture should isolate those operations behind a dedicated repository/service abstraction.

Conceptually:

```text
Knowledge Service
       │
       ├── Normal metadata CRUD
       │        ↓
       │      Prisma
       │
       └── Vector operations
                ↓
       Typed SQL / Raw SQL /
       Stored Procedure where required
                ↓
             Azure SQL
```

This allows Prisma to remain the primary ORM without requiring the core application to depend on whether Prisma directly exposes every Azure SQL vector feature.

The rest of the application should call functions such as:

```text
searchKnowledge(...)
storeKnowledge(...)
updateKnowledge(...)
removeKnowledge(...)
```

rather than directly issuing vector SQL.

This also provides an abstraction for self-hosted implementations.

---

# Database Access Boundary

The intended access path is:

```text
AI
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
Prisma / Knowledge Repository
 │
 ▼
Azure SQL
```

The application determines:

- Organization.
- Provider.
- Customer.
- Role.
- Permitted action.
- Permitted data scope.

before any database query is executed.

---

# Identity

For messaging channels, the customer's verified messaging identifier acts as the initial identity boundary.

Examples include:

- WhatsApp number.
- Telegram account.
- Future supported messaging identities.

The messaging gateway maps the external identity to an internal customer identity.

The AI should not be allowed to arbitrarily specify another customer's identifier when requesting data.

Identity resolution and authorization occur outside the model.

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
          ┌───────┴─────────┐
          ▼                 ▼
   Authorization         AI Layer
                           │
                           ▼
                         AI SDK
          │
          └────────┬─────────┘
                   ▼
           Deterministic Tools
                   │
                   ▼
          Application Services
                   │
                   ▼
                 Prisma
                   │
                   ▼
               Azure SQL
```

The initial MVP only needs:

- WhatsApp.
- Telegram.

Additional channels can later implement the same gateway interface.

The internal application works with a normalized message format instead of depending directly on WhatsApp, Telegram, or another messaging platform.

---

# AI Layer

The application should not depend directly on one model vendor.

**AI SDK** will provide the application-level AI abstraction.

Conceptually:

```text
Conversation Engine
        │
        ▼
      AI SDK
        │
        ├── Platform-provided model
        │
        ├── Service provider's API
        │
        ├── OpenAI-compatible API
        │
        ├── Custom provider adapter
        │
        └── Local/private endpoint
```

The application uses the AI SDK for capabilities such as:

- Model invocation.
- Structured outputs where useful.
- Tool calling.
- Streaming where appropriate.
- Provider abstraction.
- Custom model integrations.

Business logic should not depend on a particular underlying model.

---

# LLM Configuration Model

The system supports multiple ways of supplying the conversational model.

## Platform-Provided Model

For the hosted SaaS version, the platform can provide a managed AI configuration.

The service provider does not need to supply their own AI credentials.

Inference costs can be incorporated into SaaS pricing or usage limits.

---

## Bring Your Own AI API

A service provider can supply their own AI provider credentials.

The provider can use model options available through their configured integration.

The application's business logic remains unchanged.

---

## Custom or Local Model Endpoint

A deployment may configure a private or locally hosted model API.

For example:

```text
http://localhost:8000/v1
```

or another endpoint available within the organization's network.

This can point to:

- A locally hosted model.
- An OpenAI-compatible inference server.
- A model running on another machine.
- A private cloud endpoint.
- A supported commercial provider.
- A custom AI SDK provider integration.

This makes AI-provider choice independent of infrastructure choice.

---

# AI Configuration Isolation

AI configuration must be scoped to the organization or service provider.

One organization's:

- API credentials.
- Endpoint.
- Model selection.
- Usage information.
- Requests.

must never become available to another organization.

Credentials should be stored securely and must never be included directly in prompts.

Conceptually:

```text
Organization
     │
     ▼
AI Configuration
     │
     ├── Provider
     ├── Endpoint
     ├── Model
     ├── Credentials Reference
     ├── Embedding Configuration
     └── Provider Options
              │
              ▼
            AI SDK
```

---

# Model Independence

Business logic must not depend on a specific model.

For example:

```text
reschedule_appointment(...)
```

should work regardless of whether the request was interpreted by:

- A platform-provided model.
- The service provider's commercial model API.
- A self-hosted open model.
- A private inference endpoint.

The AI interprets the request and proposes an allowed operation.

The backend decides whether that operation is valid and authorized.

---

# Serverless Architecture

The hosted SaaS should use a **serverless, event-driven architecture**.

A permanently running conversational application server should not be required.

Azure Functions will perform bounded backend executions.

Incoming messages become events.

A simplified flow is:

```text
Incoming Message
      ↓
Channel Webhook
      ↓
Azure Function
      ↓
Normalize Message
      ↓
Resolve Identity
      ↓
Authorize Request
      ↓
Retrieve Context
      ↓
Retrieve Relevant Vector Knowledge
      ↓
AI SDK
      ↓
AI Decision / Tool Call
      ↓
Deterministic Application Service
      ↓
Prisma / Azure SQL
      ↓
Database Event
      ↓
Outbound Notification
```

Each incoming request should remain bounded.

Long-running workflows should be decomposed into independent events.

---

# Event-Driven Workflows

For example, notifying 500 customers should not require a single function to remain active while all messages are delivered.

Instead:

```text
Provider changes schedule
        ↓
ScheduleChanged Event
        ↓
Find affected appointments
        ↓
One notification event
per affected customer
        ↓
Independent executions
```

This keeps function execution short and makes workflows resilient to serverless execution limits.

---

# Scheduled Jobs

Scheduled and time-dependent workflows will use **timer-triggered Azure Functions** for Azure-hosted deployments.

No continuously running scheduler is required.

Examples include:

- Appointment reminders.
- Confirmation requests.
- Routine follow-up reminders.
- Scheduled broadcasts.
- Deferred notifications.
- Retry jobs.
- Expired pending operations.
- Cleanup tasks.
- Expiration/deactivation of temporary operational knowledge.

A timer-triggered function can periodically query Azure SQL for due work.

```text
Timer Trigger
      ↓
Azure Function
      ↓
Query Azure SQL
for due jobs
      ↓
Create events
      ↓
Independent processing
```

The timer function should generally discover and dispatch work rather than performing every downstream operation itself.

---

# Appointment Reminder Example

An appointment can contain reminder state stored in Azure SQL.

```text
Appointment
├── appointment_time
├── reminder_time
├── confirmation_status
└── reminder_status
```

A timer-triggered function detects:

```text
reminder_time <= current_time
AND reminder_status = pending
```

and emits the appropriate notification event.

```text
Timer Function
      ↓
Find Due Reminders
      ↓
ReminderDue Event
      ↓
Messaging Function
      ↓
WhatsApp / Telegram
      ↓
Update Azure SQL
```

---

# Routine Follow-Up Example

Suppose a doctor recommends that a customer return after six months.

Only the operational follow-up requirement needs to be stored.

```text
FollowUp
├── customer_id
├── provider_id
├── due_at
├── status
└── operational category
```

When the date arrives:

```text
Timer Trigger
      ↓
Find Due Follow-Ups
      ↓
FollowUpDue Event
      ↓
Customer Notification
      ↓
Offer Provider Availability
      ↓
Customer Books if Desired
```

The system does not require medical records to perform this workflow.

---

# Background and Deferred Work

Operations that do not need to complete during the original request should become events.

Examples include:

- Appointment reminders.
- Provider broadcasts.
- Schedule-change notifications.
- Routine follow-up reminders.
- Provider-question escalation.
- Returning provider answers to customers.
- Generating embeddings.
- Updating vector knowledge.
- Retrying failed outbound messages.
- Scheduled maintenance.

For example:

```text
Customer Question
       │
       ▼
Knowledge Search
       │
       ▼
No Reliable Answer
       │
       ▼
ProviderQuestionCreated
       │
       ▼
Provider Receives Question

        ...later...

Provider Reply
       │
       ├────> Reply to Customer
       │
       └────> Store Operational Knowledge
                       │
                       ▼
                Generate Embedding
                       │
                       ▼
                   Azure SQL
```

---

# Infrastructure Direction

Primary stack:

- Next.js.
- DaisyUI.
- AI SDK.
- Prisma.
- Flask/Python services where appropriate.
- Docker.
- Terraform.
- floci-az.
- Azure.
- Azure Functions.
- Timer-triggered Azure Functions.
- Azure SQL.
- Azure SQL vector search.
- Event-driven/serverless services.

Infrastructure should be reproducible using Terraform wherever applicable.

The application must remain sufficiently decoupled from Azure that the same core product can operate in Azure or through a local/self-hosted environment.

---

# floci-az

**floci-az** is an important part of the infrastructure strategy.

It is not merely an Azure testing emulator.

It will be used for:

- Local development.
- Local validation.
- Validation of Azure-oriented application behavior.
- Running the application without Azure.
- Local hosting.
- Self-hosted deployments.
- Testing production-oriented workflows before deployment.

The architecture therefore treats floci-az as part of the deployment abstraction.

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

The application should be capable of operating against either environment with minimal environment-specific changes.

---

# Azure Deployment

The Azure-hosted architecture will include:

- Azure Functions for serverless execution.
- Timer-triggered Azure Functions for scheduled jobs.
- Azure SQL for relational data.
- Azure SQL vector capabilities for operational knowledge.
- Prisma for relational application access.
- Event-driven processing.
- AI SDK for model/provider abstraction.
- WhatsApp and Telegram integrations.

The project will prioritize Azure services with persistent free-tier availability where practical.

The architecture should remain:

- Serverless.
- Event-driven.
- Cost-conscious.
- Horizontally distributable.
- Independent of continuously running backend processes wherever practical.

Azure is a supported deployment target rather than a mandatory runtime dependency for the product.

---

# Azure-Independent Operation

A deployment should also be capable of running without Azure.

A service provider or organization may operate the system using:

- Docker.
- floci-az.
- Local/private infrastructure.
- A compatible relational database.
- A compatible vector-search implementation.
- A local/private AI endpoint.
- Their own messaging and networking configuration.

The abstraction should allow equivalent local implementations of:

- Serverless/function execution.
- Timed jobs.
- Relational storage.
- Vector retrieval.
- Event processing.

without rewriting the higher-level business logic.

---

# Deployment Model

Two primary distribution models are planned.

## Hosted SaaS

The project operates the infrastructure and organizations subscribe to the service.

The hosted version may provide:

- Azure-hosted infrastructure.
- Azure Functions.
- Azure SQL.
- Vector-backed operational knowledge.
- Scheduled functions.
- Messaging integrations.
- Platform-managed AI access.
- AI SDK integrations.
- Updates.
- Operational monitoring.

A service provider may alternatively supply their own AI provider configuration.

---

## Self-Hosted

Organizations may deploy the system within infrastructure they control.

The self-hosted version may use:

- Docker.
- Terraform where applicable.
- floci-az.
- Compatible relational storage.
- Compatible vector retrieval.
- Local scheduling equivalents.
- The organization's network.
- The organization's own AI API.
- A locally hosted AI model.

A self-hosted installation should not inherently require Azure.

---

# Deployment and AI Independence

Infrastructure and AI-provider choices should be independent.

Examples:

```text
Azure Deployment
+ Azure SQL
+ Azure Functions
+ Platform AI
```

```text
Azure Deployment
+ Azure SQL
+ Azure Functions
+ Provider's AI API
```

```text
Azure Deployment
+ Azure SQL
+ Private AI Endpoint
```

```text
floci-az / Self-Hosted
+ External AI API
```

```text
floci-az / Self-Hosted
+ Local AI Model
```

Conceptually:

```text
Infrastructure
├── Azure
│   ├── Azure Functions
│   ├── Timer Functions
│   ├── Azure SQL
│   └── Vector Search
│
└── Local / Self-Hosted / floci-az

AI
├── Platform Model
├── Provider API
└── Local / Private Model
```

Neither choice should unnecessarily constrain the other.

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
                 Context Layer     AI SDK
                       │             │
                       │      ┌──────┼────────┐
                       │      │      │        │
                       │      ▼      ▼        ▼
                       │   Platform Provider Local
                       │      AI     API      API
                       │
                       ▼
             Knowledge Retrieval
                       │
                       ▼
              Azure SQL Vectors
                       │
                       └──────┐
                              ▼
                         Tool Decision
                              │
                              ▼
                   Deterministic Backend
                              │
                  ┌───────────┼───────────┐
                  │           │           │
                  ▼           ▼           ▼
             Scheduling  Notifications  Knowledge
                  │           │           │
                  └───────────┼───────────┘
                              ▼
                           Prisma
                              │
                              ▼
                          Azure SQL
                              │
                ┌─────────────┴─────────────┐
                │                           │
                ▼                           ▼
             Events                  Scheduled Jobs
                │                           │
                ▼                           ▼
         Azure Functions             Timer Functions
```

---

# Request Lifecycle

A typical customer request follows approximately:

```text
1. Receive message
        ↓
2. Normalize channel payload
        ↓
3. Resolve organization
        ↓
4. Resolve user identity
        ↓
5. Check spam/block state
        ↓
6. Determine authorization scope
        ↓
7. Retrieve required relational context
        ↓
8. Generate query embedding if knowledge retrieval is needed
        ↓
9. Search permitted Azure SQL vector knowledge
        ↓
10. Resolve organization's AI configuration
        ↓
11. Send context through AI SDK
        ↓
12. AI determines response/tool request
        ↓
13. Backend validates requested operation
        ↓
14. Execute deterministic operation
        ↓
15. Persist state through Prisma/Azure SQL
        ↓
16. Emit events where necessary
        ↓
17. Generate response
        ↓
18. Send through original messaging channel
```

At no point should the AI become the authority for:

- Identity.
- Authorization.
- Tenant boundaries.
- Customer boundaries.
- Database integrity.
- Appointment state.
- Vector retrieval scope.

---

# Provider-Answer Lifecycle

When the system needs information from the provider:

```text
Customer asks question
        ↓
Authorized knowledge search
        ↓
No suitable answer found
        ↓
Create provider question
        ↓
Provider receives question
        ↓
Provider answers
        ↓
Classify answer scope
        │
        ├── Generic
        │      ↓
        │ Provider-level knowledge
        │
        └── Customer-specific
               ↓
          Customer-scoped knowledge
        ↓
Generate embedding
        ↓
Store text + vector + metadata
        ↓
Reply to waiting customer
```

This is one of the core capabilities differentiating the product from a traditional support chatbot.

---

# Scheduled Job Lifecycle

```text
1. Timer-triggered Azure Function executes
        ↓
2. Query Azure SQL for due jobs
        ↓
3. Claim/mark work for processing
        ↓
4. Emit one event per operation
        ↓
5. Independent Function handles event
        ↓
6. Perform deterministic action
        ↓
7. Send notification if required
        ↓
8. Persist status through Prisma/Azure SQL
```

Scheduled operations should be idempotent where possible so retries do not create duplicate notifications or duplicate state changes.

---

# MVP

The first meaningful version should focus on the core front-desk workflow.

## Provider

- Configure availability.
- Change availability.
- View/request schedule.
- Mark/unmark spam contacts.
- View spam contacts.
- Broadcast messages.
- Respond to escalated customer questions.
- Define follow-up reminder periods.
- Supply reusable operational information.

## Customer

- View available appointments.
- Request appointments conversationally.
- Book appointments.
- Reschedule their own appointments.
- Cancel where permitted.
- Confirm upcoming appointments.
- Ask operational questions.
- Receive provider responses.

## Operational Knowledge

- Provider question escalation.
- Generic provider knowledge.
- Customer-specific knowledge.
- Embedding generation.
- Azure SQL vector storage.
- Semantic retrieval.
- Deterministic tenant/provider/customer filtering.
- Optional knowledge expiration.

## Automation

- Appointment confirmation.
- One-day-before reminders.
- Appointment confirmation requests.
- Provider schedule-change notifications.
- Rescheduling workflows.
- Routine follow-ups.
- Timer-triggered scheduled jobs.
- Provider broadcasts.
- Retry processing.

## Channels

- Telegram.
- WhatsApp.

## AI

- AI SDK.
- Platform-managed model option.
- Bring-your-own-provider option.
- Custom/private endpoint option.
- Local model option.
- Tool calling.
- Natural-language intent understanding.
- Clarification questions.
- Authorized context retrieval.
- Provider escalation.

## Database

- Azure SQL for hosted deployments.
- Prisma for normal relational operations.
- Azure SQL native vectors for semantic operational knowledge.
- Dedicated vector repository abstraction.
- No direct AI-to-database access.
- Deterministic tenant/customer scoping.

## Infrastructure

- Azure Functions.
- Timer-triggered Azure Functions.
- Azure SQL.
- Azure SQL vector search.
- Prisma.
- AI SDK.
- Azure deployment support.
- floci-az validation.
- floci-az local/self-hosted operation.
- Docker.
- Terraform where applicable.
- No mandatory Azure dependency for self-hosting.

---

# What the AI Must Not Do

The AI must not:

- Access medical records.
- Diagnose a patient.
- Provide medical advice as part of the front-desk system.
- Read records belonging to unrelated customers.
- Execute arbitrary database queries.
- Connect directly to Azure SQL.
- Execute unrestricted vector searches.
- Use semantic similarity as an authorization mechanism.
- Retrieve another customer's private operational context.
- Modify provider availability on behalf of a customer.
- Modify another customer's appointment.
- Reschedule a customer's appointment because the provider directly requested it.
- Invent availability.
- Claim an operation succeeded before backend confirmation.
- Circumvent spam rules.
- Determine its own authorization scope.
- Select arbitrary identities to access.
- Receive database credentials.
- Receive another organization's AI credentials.
- Expose AI API credentials to customers.
- Bypass deterministic validation.
- Treat generated text as confirmation of a state-changing operation.

The application backend remains the authority for permissions and state.

---

# Architectural Principles

1. **Least privilege**  
   Every request receives only the data required for the current operation.

2. **Deterministic authorization**  
   Permissions are enforced by application code, not model reasoning.

3. **Tenant isolation**  
   Organization, provider, and customer boundaries are represented explicitly in relational data and enforced before data reaches the AI.

4. **Vector search is retrieval, not authorization**  
   Semantic similarity determines relevance only after deterministic scope filtering.

5. **Model independence**  
   AI SDK separates application logic from the underlying model/provider.

6. **Infrastructure independence**  
   Azure is a supported hosted target while floci-az enables local and self-hosted operation.

7. **Database abstraction**  
   Prisma provides the primary relational data-access layer.

8. **Vector abstraction**  
   Azure SQL vector-specific operations remain isolated behind a knowledge repository rather than leaking database-specific code throughout the application.

9. **Serverless execution**  
   Interactive and asynchronous work is decomposed into short, bounded executions.

10. **Event-driven workflows**  
    Long-running operations are represented as events rather than persistent processes.

11. **Scheduled serverless execution**  
    Time-based workflows use timer-triggered functions rather than continuously running schedulers.

12. **Channel independence**  
    Telegram and WhatsApp are adapters around a common messaging interface.

13. **Provider/customer separation**  
    Providers control availability; customers control their appointments within that availability.

14. **Minimal healthcare scope**  
    The healthcare implementation manages service operations and scheduling, not medical records or clinical decision-making.

15. **AI as an interface, not an authority**  
    The model understands language, organizes information, and proposes actions; deterministic application services control what actually happens.

16. **Knowledge should improve over time**  
    Missing information can be escalated to the provider, stored with appropriate scope, embedded, and reused when relevant later.

---

# Product Positioning

The product is not primarily:

> A chatbot for answering customer questions.

It is:

> An AI-operated front desk that coordinates customers and service providers through existing messaging platforms while executing tightly controlled operational workflows.

Its advantage is the combination of:

**Conversation + operational context + semantic operational memory + scheduling + controlled actions + provider escalation + proactive notifications + scheduled automation + deployment independence + model independence.**

Instead of ending a conversation when information is unavailable, the system can obtain the missing information from the service provider, answer the customer, and retain appropriately scoped operational knowledge for future requests.

Generic provider answers become reusable provider knowledge.

Customer-specific answers remain associated with that customer and cannot become visible to unrelated customers.

Azure SQL provides both relational operational state and vector-backed semantic retrieval for hosted deployments, while Prisma remains the primary application data-access abstraction.

AI SDK prevents the application's conversational layer from becoming tightly coupled to one model vendor and allows hosted, customer-provided, private, or local model configurations.

Timer-triggered serverless functions handle scheduled work without requiring continuously running workers.

floci-az allows the architecture to be validated, operated, and self-hosted independently of Azure.

This makes the system a persistent operational intermediary between the service provider and customer rather than another FAQ chatbot.
