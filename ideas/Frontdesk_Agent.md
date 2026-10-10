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
- Explore available products or services conversationally, including options relevant to their interests.

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

Allow the provider to instruct the system to send a message to a defined group of customers, including announcements about upcoming events, current offers, and operational changes. The provider supplies the details conversationally and may request a targeted notification; merely adding knowledge does not automatically broadcast it.

---

# Context Escalation and Asynchronous Follow-Up

The agent must not end a conversation or wait for the provider whenever information is unavailable. **Unresolved questions are independent, durable requests**, not blocked conversations.

## Missing or Unreliable Context

When the agent cannot answer a question using authorized information or sources:

1. Tell the customer that the information is not currently available and that the agent will ask the service provider and follow up when an answer is received.
2. Store a **pending question**, linked to the tenant, appropriate provider, original customer, conversation/channel, and original question.
3. Notify the authorized provider through the provider-facing interface as a separate, asynchronous action.
4. **Continue handling further messages and unrelated questions immediately.** Other questions, bookings, and reminders must not wait for the provider.

Multiple pending questions may accumulate for the same customer or across different customers. Each remains traceable and independently resolvable.

## Customer-Requested Clarification

A customer can also escalate an answer that was supplied but was unclear, incomplete, incorrect, or otherwise unsatisfactory.

For example:

> That doesn't answer my question. Can you check with the service provider?

This follows the **same pending-question workflow**, preserving the original question, the answer given, and the customer's clarification or objection. It does not interrupt the rest of the conversation.

## Provider Answer and Return

The provider may respond with text, a document, a ZIP archive, or a link. When the information is available and validated for the relevant scope:

1. Store the source information as appropriately scoped operational knowledge.
2. Check **open pending questions** that the new information may answer, including questions from more than one customer.
3. Re-evaluate each candidate question against its authorized tenant/provider/customer scope and the new source; relevance alone does not prove that a question is resolved.
4. Send the specific answer or clarification back through the **original customer's conversation/channel**, even if they have asked other questions since.
5. Mark only the successfully resolved questions as complete; unrelated or insufficiently answered questions remain pending.

Knowledge embedding and indexing may continue asynchronously without holding up a follow-up that can already be answered from validated source text.

Conceptually:

```text
Customer asks ──> Answer available? ──Yes──> Reply
                       │
                       No
                       ▼
               Create pending question
                       │
              ┌────────┴────────┐
              ▼                 ▼
      Notify provider    Acknowledge customer
              │                 │
        Provider later      Conversation keeps
        supplies context     working normally
              │
              ▼
        Match authorized
        pending questions
              │
              ▼
      Answer original users
      and resolve only those
      questions actually answered
```

The same process applies when a customer requests escalation after an unsatisfactory answer.

---
---

# Knowledge Sources and Ingestion

Service providers can add knowledge through **plain text, documents, uploaded ZIP archives, links, and public repositories** (including GitHub). General provider instructions and policies can be stored alongside these sources.

- Uploaded files and archives are extracted and processed as source-linked content; relevant text is chunked, embedded, and made searchable within the existing tenant/provider/customer access boundaries.
- Provided URLs can be saved as sources. Repository documentation and code can be indexed, while time-sensitive public information (such as issues, pull requests, workflow runs, and available action logs) can be fetched when a question requires current details.
- If a source does not answer a question, the agent creates a persistent pending request and continues the current conversation. The provider may later reply with text, files, or links; the agent checks which authorized pending questions the new context answers and follows up with those original requesters. An unsatisfactory answer can be escalated in the same way.
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
One shared containerized application runtime
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

an expiration date can prevent obsolete information from being presented as current.

## Time-Sensitive Knowledge

Events, offers, temporary closures, and other dated announcements should be stored with their actual effective dates, times, and timezone where available.

- Resolve relative phrases such as "one week from now" against the **original message timestamp** and the provider's timezone, not the date of a future customer query. Ask for clarification if the date or time is genuinely ambiguous.
- When a provider gives a start/end date or cancellation, retain that structured validity information alongside the source text and embedding.
- Before answering questions about current or upcoming events/offers, compare those dates against the current time. Do not present expired or cancelled information as still active.
- Historical information can remain stored for accurately answering questions about past events, but must be clearly described as past. Corrections and cancellations take precedence over stale announcements.
- If no reliable current information is available, ask the provider instead of assuming an old announcement still applies.

The system therefore combines:

**semantic relevance + authorized scope + source context + validity period + current state**

when retrieving knowledge. Embeddings find potentially relevant text; deterministic time checks establish whether a dated claim is still applicable.

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
Discord ───┼──> Messaging Gateway
SMS ───────┤
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

The currently planned channel adapters are **WhatsApp, Telegram, and Discord**. Additional channels can later implement the same gateway interface.

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


# Container-First Runtime and Event-Driven Architecture

The initial hosted SaaS should use a **container-first, event-driven architecture** to keep runtime behavior as consistent as practical across local development, floci-az, and Azure. Azure Container Apps is the preferred hosted runtime for containerized components, with Docker providing a common packaging model.

The Next.js frontend, backend API, channel adapters, and background workers can be deployed as separate containers according to scaling and protocol needs. They share application contracts and infrastructure abstractions rather than requiring one monolithic process.

Persistent channel adapters are allowed where a protocol requires them, such as maintaining a Discord Gateway WebSocket connection. Webhook-based channels can deliver messages to API endpoints. Some adapters or workers may need active replicas, but not every application component needs to run continuously.

Incoming messages become events. Depending on the channel, they arrive through a webhook or a persistent adapter.

A simplified flow is:

```text
Incoming Message
      ↓
Channel Adapter (Webhook or Gateway)
      ↓
Containerized API / Message Handler
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
Persisted Event / Queue
      ↓
Containerized Worker
      ↓
Outbound Notification
```

Each incoming request and worker job should remain bounded.

Long-running workflows must use durable, resumable state and be decomposed into independent events, queued jobs, and bounded executions. Pending work must survive request completion, retries, and process restarts. The initial design should use Azure SQL for durable job/workflow state and Azure Queue Storage or Service Bus for queued delivery, selecting only the features validated in floci-az. A queue is a delivery mechanism, not a complete workflow orchestrator; workflow state, idempotency, retries, scheduling, and recovery must be designed explicitly.

Persistent adapters and workers are permitted where needed. The architecture does not require the entire application to run as one continuously active process.

---


# Event-Driven Workflows

For example, notifying 500 customers should not require one process execution to remain active until every message has been delivered.

Instead:

```text
Provider changes schedule
        ↓
ScheduleChanged Event
        ↓
Find affected appointments
        ↓
Enqueue one notification job
per affected customer
        ↓
Independent containerized workers
process bounded jobs
```

This supports horizontal scaling, retries, and recovery without tying an entire workflow to one long-running execution.

---


# Scheduled Jobs

Scheduled and time-dependent workflows should use persisted due-work records, a scheduler/dispatcher, and queue-driven workers. The initial implementation should favor a containerized scheduler/worker that can run the same code locally and in Azure, using only scheduling and messaging features validated through floci-az.

Azure SQL stores due times and job state. The scheduler/dispatcher periodically discovers due work and enqueues bounded jobs. It must tolerate restarts and repeated scans, and use safe claiming or idempotency controls to prevent duplicate effects.

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

A simplified flow is:

```text
Containerized Scheduler / Dispatcher
      ↓
Query Azure SQL for due jobs
      ↓
Enqueue due events/jobs
      ↓
Independent containerized workers
process the work
```

The scheduler/dispatcher should discover and dispatch work rather than perform every downstream operation itself. A future orchestration service may replace or complement this approach if it can be validated through floci-az.

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

A scheduler/dispatcher detects due reminders by querying persisted state:

```text
reminder_time <= current_time
AND reminder_status = pending
```

It enqueues the appropriate notification job.

```text
Scheduler / Dispatcher
      ↓
Find Due Reminders
      ↓
ReminderDue Job in Queue
      ↓
Containerized Messaging Worker
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
Scheduler / Dispatcher
      ↓
Find Due Follow-Ups
      ↓
FollowUpDue Job in Queue
      ↓
Containerized Worker
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
- Provider-question escalation and customer-requested clarification.
- Matching newly supplied context to unresolved pending questions.
- Returning provider answers to the original customer conversations.
- Generating embeddings and updating vector knowledge.
- Retrying failed outbound messages.
- Scheduled maintenance.

A pending question is persisted independently of conversation execution so other messages remain processable.

```text
Unanswered or disputed question
           │
           ▼
Persist PendingQuestion
           │
      ┌────┴──────────────┐
      ▼                   ▼
Provider notified    Customer told a
asynchronously       follow-up is pending
      │                   │
Provider replies     Customer continues
later                using the agent
      │
      ▼
Store/validate new context
      │
      ▼
Find related OPEN questions
within authorized scope
      │
      ▼
Send answers asynchronously
to original customer channels
      │
      ▼
Mark answered questions resolved
      │
      ▼
Generate embeddings as needed
```

---


# Infrastructure Direction

Primary stack:

- Next.js.
- DaisyUI.
- AI SDK.
- Prisma.
- Flask/Python services where appropriate.
- Docker and containerized application services.
- Azure Container Apps for hosted container runtime.
- Azure Queue Storage or Service Bus for event/job delivery, selected according to verified floci-az support.
- Azure SQL and Azure SQL vector search.
- Terraform where applicable.
- floci-az for local development, validation, and self-hosted operation.
- Event-driven processing with durable workflow/job state.

Infrastructure should be reproducible using Terraform wherever applicable.

The same container images should run locally and in Azure wherever practical. The application must remain sufficiently decoupled from Azure-specific services that the same core product can operate in Azure or through a local/self-hosted environment.

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

The Azure-hosted architecture will use:

- Azure Container Apps for containerized frontend, API, channel-adapter, and background-worker components as appropriate.
- Azure SQL for relational data and durable workflow/job state.
- Azure SQL vector capabilities for operational knowledge.
- Azure Queue Storage or Azure Service Bus for queued event/job delivery, based on the features validated through floci-az.
- Prisma for relational application access.
- Event-driven processing and bounded worker jobs.
- AI SDK for model/provider abstraction.
- WhatsApp, Telegram, and Discord integrations.

The project will prioritize cost-conscious Azure services where practical while validating the exact service behavior required by the application.

The architecture should remain:

- Container-first.
- Event-driven.
- Cost-conscious.
- Horizontally scalable.
- Able to keep persistent replicas only for components that need them, such as a Discord Gateway adapter or a background worker.

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
- A compatible queue/event-delivery mechanism.
- A local/private AI endpoint.
- Their own messaging and networking configuration.

The abstraction should allow equivalent local implementations of:

- Containerized API, channel-adapter, and worker execution.
- Queue/event delivery.
- Timed jobs and scheduled dispatch.
- Durable asynchronous workflow execution, using equivalent supported mechanisms where necessary.
- Relational storage.
- Vector retrieval.

without rewriting the higher-level business logic.

---

# Future Infrastructure Exploration

## Azure Static Web Apps and Durable Functions

Azure Static Web Apps (SWA) and Azure Durable Functions are **future exploration candidates, not dependencies of the initial MVP**. Revisit them once floci-az supports the required behavior well enough for local development and end-to-end validation.

- SWA may be evaluated as an optional frontend-hosting approach if it reduces operational complexity without undermining local/Azure parity.
- Durable Functions may be evaluated as an orchestration option for durable multi-step workflows if the required trigger, orchestration, retry, and replay behavior can be validated through floci-az.

Until then, use containerized application components, Azure SQL-backed workflow/job state, and queue-driven workers. Keep workflow logic behind application-level abstractions so a future scheduling or orchestration implementation can be introduced without rewriting business logic.

---

# Deployment Model

Two primary distribution models are planned.

## Hosted SaaS

The project operates the infrastructure and organizations subscribe to the service.

The default hosted architecture is shared and multi-tenant:

```text
Shared Next.js frontend
        +
Shared containerized API and worker services
        +
Shared Azure SQL database
        +
Tenant/provider/customer scoped data
```

Ordinary account creation does not provision a new application stack, Azure SQL database, or vector database.

The hosted version may provide:

- Azure-hosted infrastructure.
- Azure Container Apps for containerized application services.
- Azure SQL and vector-backed operational knowledge.
- Queue-backed event delivery and containerized background workers.
- Durable scheduled-job state and dispatch.
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
+ Azure Container Apps
+ Azure SQL
+ Queue / Worker Services
+ Platform AI
```

```text
Azure Deployment
+ Azure Container Apps
+ Azure SQL
+ Queue / Worker Services
+ Provider's AI API
```

```text
Azure Deployment
+ Azure Container Apps
+ Azure SQL
+ Private AI Endpoint
```

```text
floci-az / Self-Hosted
+ Containerized Application
+ External AI API
```

```text
floci-az / Self-Hosted
+ Containerized Application
+ Local AI Model
```

Conceptually:

```text
Infrastructure
├── Azure
│   ├── Azure Container Apps
│   ├── Queue Storage / Service Bus
│   ├── Azure SQL
│   └── Vector Search
│
└── Local / Self-Hosted / floci-az
    ├── Docker Containers
    ├── Compatible Queue / Event Delivery
    └── Compatible Storage

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
       Queue / Event Bus          Scheduler / Dispatcher
                │                           │
                ▼                           ▼
       Containerized Workers      Azure SQL Job State
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

Both missing information and customer dissatisfaction produce independent pending questions.

```text
Customer question / clarification
             ↓
Persist pending request
             ↓
Acknowledge customer and continue conversation
             ↓
Provider receives request asynchronously
             ↓
Provider supplies text/files/links later
             ↓
Store and scope new information
             ↓
Find relevant unresolved requests
             ↓
Check each against authorization and answer quality
             ↓
Reply to each original requester via their channel
             ↓
Resolve answered requests; leave others pending
             ↓
Embed reusable knowledge asynchronously
```

Each pending record keeps enough context to identify the original question, customer, provider, and destination conversation. A single provider response may resolve several related questions, but unrelated questions must remain open.

The agent does not have to wait for provider response before accepting further input from any user. This non-blocking follow-up is a core capability, not an industry-specific behavior.

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

Large broadcasts should similarly be split into bounded per-recipient worker jobs rather than one long-running execution.

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
- Explore published products/services from provider websites or connected catalogs.
- Receive provider responses.

## Operational Knowledge

- Text, document, ZIP, URL, and public-repository sources.
- Source ingestion and retrieval of current external information when needed.
- Non-blocking provider escalation for missing or unsatisfactory answers.
- Durable pending-question tracking and asynchronous follow-up to original channels.
- Matching new provider context to relevant unresolved questions.
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
- Provider escalation without blocking ongoing conversations.
- Customer-requested clarification of unsatisfactory answers.

## Database

- Azure SQL for hosted deployments.
- Prisma for normal relational operations.
- Azure SQL native vectors for semantic operational knowledge.
- Dedicated vector repository abstraction.
- No direct AI-to-database access.
- Deterministic tenant/customer scoping.

## Infrastructure

- Azure Container Apps for hosted containerized services.
- Containerized frontend, API, channel adapters, and background workers.
- Azure Queue Storage or Service Bus for queued event/job delivery, subject to verified floci-az support.
- Azure SQL for operational and durable workflow/job state.
- Azure SQL vector search.
- Prisma.
- AI SDK.
- Azure deployment support.
- floci-az validation and local/self-hosted operation.
- Docker.
- Terraform where applicable.
- No mandatory Azure Functions, SWA, or Durable Functions dependency for the initial implementation.
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
- Azure Static Web Apps or Azure Durable Functions as mandatory dependencies.

The initial runtime is container-first rather than Functions-first. The frontend, API, adapters, and workers may be deployed separately as containers. Persistent channel adapters and some workers may need active replicas, but ordinary requests and background tasks should remain independently deployable and bounded. Durable work must be tracked in persistent state and dispatched reliably; queues alone do not replace explicit workflow state, idempotency, retry, and recovery logic.

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
   Azure is a supported hosted target while floci-az and compatible local services enable local and self-hosted operation.

7. **Database abstraction**  
   Prisma provides the primary relational data-access layer.

8. **Vector abstraction**  
   Azure SQL vector-specific operations remain isolated behind a knowledge repository rather than leaking database-specific code throughout the application.

9. **Container-first runtime and bounded execution**  
   Containerized services maximize local/Azure runtime parity. Requests and worker jobs remain bounded even when adapters or workers need persistent processes.

10. **Event-driven workflows**  
    Long-running operations use durable workflow state, events, and bounded executions rather than relying on a single process to remain active.

11. **Durable scheduled work**  
    Due work is persisted and dispatched to workers. The initial implementation uses scheduling and queue mechanisms validated through floci-az rather than depending on Azure Durable Functions.

12. **Channel independence**  
    Telegram, WhatsApp, and Discord are adapters around a common messaging interface. Adapters may use webhooks or persistent connections according to each platform's protocol.

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

A containerized scheduler/dispatcher and queue-driven workers handle scheduled work and retries, with durable job state stored in Azure SQL.

floci-az allows the architecture to be validated, operated, and self-hosted independently of Azure.

This makes the system a persistent operational intermediary between the service provider and customer rather than another FAQ chatbot.

---

# Example Use Cases

These are **two applications of the same general-purpose front desk**, not separate products or separate architectures.

## 1. Clinic Front Desk

**Interfaces**

- **Patient-facing:** Patients message the clinic through WhatsApp or Telegram to ask questions and manage appointments. **Future option:** embed customer chat on the clinic's landing or service pages.
- **Provider-facing:** Authorized clinic staff use their chat interface to update availability, supply operational information, answer escalated questions, and request announcements.

**Example workflow**

1. **Add operational context.** Staff tell the agent: "We're hosting a patient orientation event next Saturday at 3 PM in Hall B," or provide details of a time-limited service offer. The agent stores the announcement with its resolved date, location, conditions, and validity period, along with any supporting documents or links.
2. **Answer customer questions.** A patient asks through the clinic's messaging channel about the venue, event timing, available services, offer eligibility, opening hours, or appointment availability. The agent answers using current authorized context. Once an event has passed or an offer has expired, it explains that it is no longer current rather than promoting the old announcement.
3. **Escalate without blocking.** If the information is missing, or the patient finds an answer unsatisfactory, the agent records a pending question, says it will check with staff, and continues answering other patient requests. Staff later clarify through their provider-facing chat, optionally attaching a file or link. The agent returns the clarification to the original patient and checks whether it also resolves other related pending questions, while preserving customer-specific access boundaries.
4. **Notify when instructed.** Staff can ask the agent to announce the event, offer, or changed schedule to an appropriate set of patients. The announcement is sent through the configured messaging channels rather than being broadcast automatically whenever context changes.
5. **Coordinate appointments.** Patients book, cancel, or reschedule within staff-defined availability. If availability changes, affected patients are notified and offered valid replacement slots.
6. **Send timed reminders.** The agent sends appointment confirmations, configurable reminders, attendance requests, and provider-specified follow-up reminders.

**Boundary:** This workflow handles front-desk operations and service information, **not medical records, diagnosis, or clinical advice**.

## 2. Organization Front Desk (Discord)

**Interfaces**

- **Contributor-facing:** Contributors interact with the agent through a **Discord bot** in supported channels or conversations. **Future option:** embed the agent on the organization's website or project pages.
- **Maintainer-facing:** Authorized maintainers use the **provider-facing interface** (such as a protected bot conversation or admin chat) to provide context, manage their availability, answer escalations, and request announcements. They are the service providers in this example.

**Example workflow**

1. **Connect sources.** Maintainers give the agent general organization instructions, documentation, uploaded ZIP archives, and links to public GitHub repositories. The system indexes useful source content and can fetch current public issues, pull requests, workflow runs, and available action logs when a question needs fresh information.
2. **Answer contributor questions and guide discovery.** A contributor asks through Discord how to set up a project, why a PR is failing, where to find a policy, what events are coming up, or which published projects match their interests. The agent retrieves relevant authorized documentation or live repository information and responds through Discord. A future website integration could offer the same project-discovery experience.
3. **Escalate and keep chatting.** If the answer is unavailable or the contributor considers it unsatisfactory, the agent acknowledges the pending follow-up, records the question, and asks an authorized maintainer via the **maintainer-facing interface**. The contributor can continue asking other questions or scheduling meetings without waiting.
4. **Learn and follow up asynchronously.** The maintainer later supplies an explanation, file, ZIP archive, or link. The agent stores appropriately scoped knowledge, identifies which outstanding questions can now be answered, and replies to each original contributor in Discord. Questions not addressed remain pending.
5. **Manage events and announcements.** A maintainer says: "We have a contributor onboarding session one week from now at 6 PM," adds a venue or meeting link, and optionally asks the agent to notify the relevant audience. The date is resolved when the context is added. Subsequent questions receive the correct upcoming, current, cancelled, or past-event status.
6. **Coordinate meetings.** Contributors request time with a maintainer or mentor through Discord. The agent shows valid availability, handles booking/rescheduling, and notifies both parties of changes.
7. **Send reminders.** Scheduled notifications can remind the contributor and maintainer **10 or 20 minutes before their meeting**, according to the configured reminder rules.

**Shared behavior:** The Discord bot is the consumer-facing front desk; maintainer conversations are the provider-facing side of the **same agent and underlying scheduling, knowledge, escalation, and notification system**.

---

# Scheduling & Reservation Management

AI Front Desk provides a general-purpose scheduling and reservation management capability for coordinating people, time slots, sessions, and resources across different service domains.

The system supports provider availability management, appointment and meeting scheduling, rescheduling, and notifications. The same underlying capability can be extended to modify existing reservations involving a defined time period or resource, subject to availability, provider rules, and the capabilities of the connected system.

## Example Use Cases

| Domain | Example operations |
|---|---|
| Appointments and consultations | Schedule appointments and move them to another available time slot |
| Meetings and mentorship | Schedule meetings, reschedule them, and notify participants |
| Classes and workshops | Manage session schedules and move participants to sessions with available capacity |
| Hotels and accommodation | Change reservation dates when suitable rooms are available |
| Vehicle and equipment rentals | Modify rental periods based on resource availability |
| Venues and meeting rooms | Reschedule reservations or move them to another available space |
| Tours and experiences | Change reserved dates, times, or sessions where permitted |
| Other scheduled services | Adapt scheduling and reservation changes to provider-defined rules |

Initially, the generalized extension focuses on **rescheduling or modifying existing reservations**, while retaining the appointment and meeting workflows already defined. New reservation creation, cancellations, and more complex modifications can be introduced incrementally.

All changes must respect authorization, availability, applicable conditions, and the connected system's capabilities. The original reservation should remain unchanged if a requested modification cannot be completed successfully.

---

# Future Extension: Social Marketing Agent

A separate, optional **Social Marketing Agent** may be built **on top of AI Front Desk**. It would help service providers promote their products and projects through relevant public conversations, while reusing Frontdesk's existing product context, knowledge retrieval, and provider escalation instead of maintaining an independent knowledge base.

## Proposed Workflow

1. **Connect projects and accounts.** The service provider chooses the products/projects to represent, supplies or reuses their Frontdesk context, and authorizes supported social accounts (for example, X/Twitter and Reddit).
2. **Discover relevant conversations.** The marketing agent periodically searches permitted public posts and comments for questions or discussions genuinely related to those products or the problems they solve.
3. **Filter before engaging.** Check relevance, platform/community rules, prior interactions, and duplicate post/comment identifiers. Do not repeatedly respond to the same content or insert unrelated promotions.
4. **Prepare grounded responses.** Request authorized **public product/project information** from Frontdesk and draft a reply based on that information. Initially, publishing discovered-post replies should require provider review/approval and comply with the social platform's API and automation rules.
5. **Handle follow-ups.** Track replies to permitted published comments. When a new response remains within the product-related scope, reuse a previously verified Frontdesk answer directly where appropriate, without an unnecessary additional generation call. If new reasoning or wording is needed, use the AI layer only as required.
6. **Escalate missing context.** If Frontdesk cannot answer, use its existing non-blocking provider-question workflow. Once the provider supplies the clarification, the marketing agent can prepare or send the follow-up where authorized and permitted.

## Boundaries

- Social discovery, interaction tracking, approval, and posting belong to the **Social Marketing Agent**; Frontdesk remains responsible for knowledge, authorized answers, and provider escalation.
- Only information explicitly suitable for public disclosure may be used. Customer-specific records and private conversations must never enter social replies.
- Automated posting and replies must respect each platform's permissions, anti-spam policies, rate limits, and community rules. Discovering a relevant post does **not** automatically authorize a promotional reply.
- Keep a minimal record of platform, account, post/comment, conversation, and reply status to prevent duplicate engagements.
- This is a **future extension**, not part of the Frontdesk MVP or a requirement for its initial hosted demonstration.
