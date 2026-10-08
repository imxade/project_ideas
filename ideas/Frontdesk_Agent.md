# AI Front Desk

## Product Idea

AI Front Desk is a general-purpose conversational coordination layer between **any service provider** and their **customers or participants**, covering operational questions, knowledge access, booking, meetings, and service coordination.

Instead of behaving like a traditional customer-support chatbot that only answers FAQs, the system maintains operational context, has controlled access to service records and schedules, asks the service provider for missing information when necessary, and can execute predefined actions such as booking, rescheduling, notifications, reminders, and broadcasts.

The same core product is intended to serve different industries and organizations without separate industry-specific implementations. Where used in healthcare, its scope remains **service and scheduling records, not medical records**.

As a final deliverable, the project will also include a **hosted demonstration instance** operated by us. Access to this instance will be limited to explicitly authorized demo accounts so the complete workflow and practical usage of the product can be shown during pitches, reviews, or demonstrations without exposing the hosted environment as an unrestricted public service.

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

A service provider may be an individual professional, team, business, or organization.

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
- Configure reminder timing for appointments, meetings, and follow-ups.
- Supply documentation, archives, links, and other approved knowledge sources.
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

Provider availability and customer bookings are deliberately separated. Bookings may represent appointments, meetings, consultations, or other scheduled services.

The provider controls:

**Availability**

The customer controls:

**Appointments within that availability**

Example:

A service provider originally works:

Monday  
Wednesday  
Friday

The provider changes the schedule to:

Monday–Friday, 10:00 AM–5:00 PM.

The system updates provider availability.

If instead the provider removes Wednesday from their schedule, the system detects bookings affected by the change.

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

Send confirmations and reminders at provider-configured times (for example, one day, 20 minutes, or 10 minutes before an appointment or meeting). Notify the relevant participants and optionally request attendance confirmation.

## Customer Declines

Offer available alternatives and allow rescheduling.

## Routine Follow-Up Due

If a provider recommends another visit after three months, six months, or another interval, schedule an appropriate reminder according to the provider's recommendation.

## Provider Broadcast

Allow the provider to instruct the system to send a message to a defined group of customers.

---

# Context Escalation

One major distinction from conventional support bots is that the bot should not simply fail when information is missing.

If a customer asks something that the system cannot answer from existing authorized information, but the service provider can answer it, the system creates a provider query. The provider may respond with text or supply additional files or links.

For example:

Customer:

> Will your office be open during the holiday next month?

If no answer exists in the available operational knowledge or approved sources, the system asks the provider.

The provider answers:

> Yes, but only until 1 PM.

The system can then:

1. Reply to the original customer.
2. Store the provider's response as operational knowledge.
3. Generate an embedding for semantic retrieval.
4. Use that information when appropriate future questions are asked.

This creates an evolving operational knowledge base rather than a static FAQ chatbot.

---

# Knowledge Sources and Ingestion

Service providers can add knowledge through **plain text, documents, uploaded ZIP archives, links, and public repositories** (including GitHub). General provider instructions and policies can be stored alongside these sources.

- Uploaded files and archives are extracted and processed as source-linked content; relevant text is chunked, embedded, and made searchable within the existing tenant/provider/customer access boundaries.
- Provided URLs can be saved as sources. Repository documentation and code can be indexed, while time-sensitive public information (such as issues, pull requests, workflow runs, and available action logs) can be fetched when a question requires current details.
- If a source does not answer a question, the agent escalates to the provider, who can reply with text or provide another document or link. The agent then responds to the original requester and stores reusable knowledge with the appropriate scope.
- Importing and refreshing sources uses bounded background jobs rather than scanning large archives or repositories during a single chat request. Only approved or publicly accessible material should be retrieved; files are treated as data, not executable instructions.

The system should retain source references so responses can be grounded in the information actually retrieved.

---

# Tenant and Hosted Isolation Model

The hosted SaaS is **multi-tenant by default**.

The primary isolation boundary is the service organization that signs up for the product.

For example:

```text
Tenant: Service Organization A

Providers:
├── Dr. A
├── Dr. B
└── Dr. C

Customers:
├── Customer 1
├── Customer 2
└── Customer 3
```

A provider is normally a member of a tenant rather than a separate SaaS deployment.

The normal hosted deployment should use:

```text
One shared frontend
        +
One shared serverless application
        +
One shared Azure SQL database
        +
Many isolated tenants
```

The application should **not** create a separate Azure SQL database, vector database, or complete application instance for every provider or tenant by default.

Isolation is enforced through deterministic relational scope:

```text
tenant_id
    ↓
provider_id
    ↓
customer_id
```

The authenticated application context supplies these identifiers. The AI does not choose its tenant or customer scope.

A fully isolated application environment belongs to the **self-hosted deployment model**, not to ordinary SaaS signup.

---

# Operational Knowledge and Vector Search

Azure-hosted deployments will use **Azure SQL's native vector capabilities** to store and retrieve semantic operational knowledge.

The hosted SaaS uses a shared operational knowledge table rather than creating a separate physical vector database for every provider.

Provider responses that may be useful in future conversations can be stored together with an embedding and ordinary relational scope metadata such as `tenant_id`, `provider_id`, and `customer_id`.

The vector determines semantic relevance. The relational columns determine which records are allowed to participate in the search.

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

# Tenant-Wide Operational Knowledge

Some information applies to the whole service organization rather than one provider.

Examples include:

- Organization-wide opening hours.
- Holiday closures.
- Parking information.
- General payment procedures.
- Common booking rules.
- Shared office instructions.

A tenant-wide record may conceptually contain:

```text
OperationalKnowledge
├── id
├── tenant_id
├── provider_id = NULL
├── customer_id = NULL
├── scope = TENANT
├── content
├── embedding
├── created_at
├── valid_until (optional)
└── active
```

This information may be used across providers within that tenant when relevant.

It must never be visible to another tenant.

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
├── tenant_id
├── provider_id
├── customer_id = NULL
├── scope = PROVIDER
├── content
├── embedding
├── created_at
├── updated_at
├── valid_until (optional)
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
FILTER tenant_id
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
├── tenant_id
├── provider_id
├── customer_id
├── scope = CUSTOMER
├── content
├── embedding
├── created_at
├── valid_until (optional)
├── active
└── source/reference metadata
```

The customer association is stored as ordinary relational metadata.

The embedding represents the semantic meaning of the content.

The system **must not rely on the customer's name being embedded into the vector to enforce customer isolation**.

Instead, retrieval must deterministically constrain the query:

```text
tenant_id = current_organization
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

Operational knowledge retrieval is always scoped before semantic ranking.

## Customer Request

A customer interacting with a provider may retrieve:

```text
1. Tenant-wide knowledge for the current tenant
2. Provider-specific knowledge for the current provider
3. Customer-specific knowledge for the current customer
```

It must exclude:

```text
1. Another customer's private knowledge
2. Another provider's provider-specific knowledge
3. Another tenant's knowledge
```

A conceptual authorization filter is:

```text
tenant_id = current_tenant
AND (
    scope = TENANT
    OR (
        scope = PROVIDER
        AND provider_id = current_provider
    )
    OR (
        scope = CUSTOMER
        AND provider_id = current_provider
        AND customer_id = current_customer
    )
)
```

Only the records allowed by this scope participate in vector similarity ranking.

## Specificity Precedence

When multiple equally relevant active records apply, prefer the more specific scope:

```text
Customer-specific
        ↓
Provider-specific
        ↓
Tenant-wide
```

A specific temporary exception therefore does not overwrite the general rule.

If two active records at the same scope genuinely conflict and the system cannot deterministically determine which is current, it should ask the provider instead of guessing.

## Provider Request

A provider may retrieve:

- Tenant-wide knowledge available within their tenant.
- Their own provider-specific operational knowledge.
- Customer-specific knowledge only when the requested operation and authorization allow access to that customer.
- Pending customer questions addressed to them.

Provider access remains subject to deterministic application authorization.

## Safe Storage Default for Escalated Answers

A response produced because of one customer's escalated question should default to **customer-specific** knowledge unless it is clearly marked as generally applicable.

If the provider indicates that the answer applies to everyone, it may be stored at provider or tenant scope.

For ambiguous cases, the bot may ask whether the information should be remembered for all customers.

This prevents a private answer from accidentally becoming generic knowledge.

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

# Embedding Consistency and Failure Handling

For the shared hosted Azure SQL vector store, embeddings stored in the same vector column must use a consistent platform-selected embedding configuration.

A service provider may choose their own conversational model through AI SDK without implicitly changing the embedding representation used by the shared knowledge store.

Embedding generation is asynchronous from the provider/customer conversation where practical.

If storing the source text succeeds but embedding generation fails:

```text
Store source text
      ↓
embedding_status = PENDING
      ↓
Reply to the waiting customer
      ↓
Retry embedding generation later
```

The original provider answer remains the source of truth. The embedding is only a retrieval index.

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

`get_my_appointments()`

`get_provider_availability()`

`book_my_appointment(slot_id)`

`reschedule_my_appointment(appointment_id, slot_id)`

`cancel_my_appointment(appointment_id)`

`update_my_provider_availability(schedule)`

`mark_spam(contact_reference)`

`remove_spam(contact_reference)`

`list_spam()`

`create_followup_reminder(customer_reference, date, type)`

`broadcast_message(audience, message)`

`search_operational_knowledge(query)`

`store_operational_knowledge(scope, content)`

The authenticated tenant, provider, and customer identities are injected by application code rather than selected freely by the model.

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

# Identity Edge Cases

A phone number, Telegram account, or other external identifier is **not a globally unique customer record across the entire SaaS**.

The same person may interact independently with multiple service organizations.

For example:

```text
Service A → Customer A17 → +91XXXXXXXXXX
Service B → Customer B42 → +91XXXXXXXXXX
```

Those customer relationships remain isolated.

Incoming messages should resolve the tenant from the receiving messaging connection first, and then resolve the sender within that tenant.

The same customer may also use multiple messaging channels. Those identities must not be merged solely because names or usernames look similar. Linking identities requires a trusted verification flow.

A changed phone number must not automatically inherit another number's records based on AI reasoning.

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

The supported channel adapters include **WhatsApp, Telegram, Discord, and embeddable website chat**. Additional channels can later implement the same gateway interface.

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

# Hosted and Local AI Endpoints

A local URL such as:

```text
http://localhost:8000/v1
```

is meaningful only from the machine or container making the request.

This works naturally for a self-hosted deployment where the AI endpoint is locally reachable.

The hosted SaaS cannot directly use a service provider's `localhost` endpoint. A provider-supplied endpoint used by the hosted service must be securely network-accessible from the hosted application.

The platform must not silently fall back from a provider-selected AI service to a platform model unless that fallback behavior has been explicitly configured.

This avoids changing privacy, billing, or data-processing expectations without consent.

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
Configured Messaging Channel
      ↓
Update Azure SQL
```

---

# Routine Follow-Up Example

Suppose a service provider asks a customer to return for a follow-up after six months.

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

The default hosted architecture is shared and multi-tenant:

```text
Shared Next.js frontend
        +
Shared serverless backend
        +
Shared Azure SQL database
        +
Tenant/provider/customer scoped data
```

Ordinary account creation does not provision a new application stack, Azure SQL database, or vector database.

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

# Practical Correctness and Reliability Edge Cases

The initial implementation should handle the following cases without introducing unnecessary infrastructure.

## Concurrent Booking

Two customers may attempt to book the same slot at nearly the same time.

Availability displayed earlier is not sufficient authorization to book.

The state-changing database operation must validate and claim the slot atomically.

Only one booking succeeds. The other request receives updated availability.

## Provider Availability Changes During Booking

If a provider changes availability after a customer was shown a slot but before the booking commits, the database state wins.

The invalid slot is rejected and alternatives are offered.

## Availability Changes Affect Existing Appointments

If a provider removes availability that contains existing appointments:

```text
Existing appointment
      ↓
Mark as affected / needs rescheduling
      ↓
Notify customer
      ↓
Customer selects replacement
```

The system must not silently move or cancel the appointment.

## Duplicate and Out-of-Order Messages

Messaging platforms and webhook infrastructure may retry or deliver messages out of order.

Where available, incoming messages should be deduplicated using a key such as:

```text
tenant_id
channel_connection_id
external_message_id
```

Every state-changing tool must re-check current database state before committing.

## Idempotent Mutations

A model/tool retry must not execute the same state change twice.

For example, if booking succeeds but response generation fails, retrying the request must return the existing result rather than creating a second appointment.

State-changing operations should therefore carry an idempotency/operation identifier where appropriate.

## Timed Functions Are Scanners

Timer-triggered functions should discover durable due work rather than represent the work themselves.

A scheduled record may contain:

```text
id
tenant_id
type
due_at
status
attempt_count
claimed_at
completed_at
idempotency_key
```

If a timer invocation is missed or processing fails, a later scan can rediscover unfinished due work.

A reminder job must re-read the current appointment state before sending because the appointment may have been cancelled, rescheduled, or completed after the reminder was created.

## Outbound Delivery Is Separate From State Changes

Successful appointment or schedule updates must not depend on immediate WhatsApp or Telegram delivery.

Conceptually:

```text
Database state change
        +
Outbound message/event record
        ↓
Commit
        ↓
Independent delivery attempt
```

A messaging outage therefore does not corrupt appointment state.

Large broadcasts should similarly be split into bounded per-recipient work rather than one long-running function execution.

## High-Impact Provider Actions

Clear customer actions such as booking a named slot do not require unnecessary repeated confirmation.

However, provider actions with broad impact should show their effect before execution.

Examples include:

- Broadcasting to a large audience.
- Changing availability that affects existing appointments.
- Blocking a large set of contacts.

## Spam Is Not Cancellation

Marking a contact as spam stops normal interaction and outbound communication.

It does not:

- cancel appointments,
- delete the customer,
- delete historical service records.

These are separate operations.

## AI Failure Does Not Corrupt Operational State

AI availability is separate from database correctness.

If the configured model is unavailable:

- existing appointments remain valid,
- deterministic scheduled processing may continue where AI is not required,
- the application must not fabricate a successful action.

If a database mutation cannot be confirmed, the assistant must not claim that it succeeded.

## Vector Retrieval Can Return No Answer

The closest vector is not automatically a valid answer.

If no authorized knowledge record is sufficiently relevant, the system should treat the result as unknown and escalate to the provider.

## Knowledge Conflicts

If two active records at the same scope conflict and metadata cannot determine which one is current, the system should ask the provider.

Semantic similarity must not be used to decide which factual statement is true.

## Bounded Context

Large schedule or knowledge requests must be bounded or paginated before being passed to the model.

The AI may format authorized records into structured text, but it should not receive arbitrarily large database result sets.

## Time Handling

Concrete appointment timestamps should be stored in UTC.

Tenant/provider timezone information is stored separately for schedule interpretation and display.

Authoritative timezone conversion and scheduling calculations belong to deterministic application code rather than the model.

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

- Text, document, ZIP, URL, and public-repository sources.
- Source ingestion and retrieval of current external information when needed.
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
- Configurable reminders before appointments or meetings (for example, one day, 20 minutes, or 10 minutes in advance).
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
- Discord.
- Embeddable website chat.

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

# Architecture Decisions Explicitly Kept Simple

The initial hosted system does **not** require:

- A separate vector database per provider.
- A separate Azure SQL database per tenant.
- A separate application deployment per tenant.
- Automatic Azure infrastructure provisioning on every signup.
- Cross-tenant customer identity merging.
- Semantic deduplication of unresolved provider questions.
- Arbitrary AI-generated SQL.
- AI-based authorization.
- Long-running persistent bot workers.

The default rule is:

> **Share infrastructure; isolate data deterministically.**

Self-hosting provides complete deployment isolation when an organization wants to run its own application environment.

---

# Architectural Principles

1. **Least privilege**  
   Every request receives only the data required for the current operation.

2. **Deterministic authorization**  
   Permissions are enforced by application code, not model reasoning.

3. **Tenant isolation**  
   Tenant, provider, and customer boundaries are represented explicitly in relational data and enforced before data reaches the AI.

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
    Telegram, WhatsApp, Discord, and embeddable website chat are adapters around a common messaging interface.

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

The hosted SaaS shares application and database infrastructure by default while enforcing deterministic tenant, provider, and customer boundaries for both relational and vector retrieval.

Azure SQL provides both relational operational state and vector-backed semantic retrieval for hosted deployments, while Prisma remains the primary application data-access abstraction.

AI SDK prevents the application's conversational layer from becoming tightly coupled to one model vendor and allows hosted, customer-provided, private, or local model configurations.

Timer-triggered serverless functions handle scheduled work without requiring continuously running workers.

floci-az allows the architecture to be validated, operated, and self-hosted independently of Azure.

This makes the system a persistent operational intermediary between the service provider and customer rather than another FAQ chatbot.

---

# Example Use Cases

These are illustrations of the same general-purpose front-desk system, not separate products.

## Clinic Front Desk

A clinic configures its staff availability and lets patients book or reschedule appointments through messaging. When availability changes, affected patients are notified and offered valid alternatives. The system sends configurable appointment and routine follow-up reminders, answers administrative questions from authorized operational knowledge, and escalates unanswered questions to clinic staff. It does not access medical records.

## Open-Source Organization Front Desk

An open-source organization connects a **Discord bot** and optionally embeds the same chat experience on its website. Maintainers upload documentation or ZIP archives and add links to public repositories. Contributors can ask about project files, issues, pull requests, and available CI/action logs. The agent retrieves relevant indexed or current public information, and escalates unanswered questions to maintainers, who can respond with text, files, or links.

Contributors can also book meetings with maintainers based on their configured availability. Both sides receive applicable notifications, such as reminders **10 or 20 minutes before a meeting**. Answers supplied by maintainers become appropriately scoped knowledge for future requests.
