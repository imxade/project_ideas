# AI Front Desk

## Product Idea

AI Front Desk is a general-purpose conversational coordination layer between a **service provider** and their **customers or participants**, covering operational questions, knowledge access, booking, meetings, and service coordination. The service provider may be one individual or an authorized person/team managing a business, organization, chain, or multiple professionals and resources.

The product has two primary interaction sides: a customer-facing side and a provider-facing side. The same provider-facing workflow manages operational information, schedules, reservations, announcements, and escalated questions whether the service provider is an individual or an organization. Individual professionals or resources are distinguished internally when needed for availability, reservation assignment, authorization, or a customer's explicitly stated preference; they do not each require a separate front-desk conversation or interface.

Instead of behaving like a traditional customer-support chatbot that only answers FAQs, the system maintains operational context, has controlled access to service records and schedules, asks the provider-facing side for missing information when necessary, and can execute predefined actions such as booking, rescheduling, notifications, reminders, and broadcasts.

A common customer frustration with bot-only support is getting repetitive answers without a practical way to reach a person, especially for complaints, suspected fraud, urgent incidents, technical support, or other exceptional requests. AI Front Desk should reduce routine support workload, not eliminate human support: it handles well-defined requests automatically while preserving a clear path to a small authorized human team for real responses, judgment, investigation, and action.

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

A service provider may be an individual professional or an authorized person/team acting for a business, organization, chain, or group of professionals. The provider-facing side is one operational interface for the service provider and its authorized staff, not a separate front desk for every doctor or employee.

A provider can:

- Define working days and hours for the services, professionals, or resources they are authorized to manage.
- Add or remove availability and mark relevant professionals or resources unavailable.
- Ask about and request schedules or appointments across their authorized scope, including a specific professional or resource when needed.
- Broadcast messages to selected or eligible customers.
- View numbers marked as spam.
- Mark a customer/contact as spam.
- Remove a number from the spam list.
- Configure reminder timing for appointments, meetings, and follow-ups.
- Supply documentation, archives, links, and other approved knowledge sources.
- Answer customer questions escalated by the AI.
- Provide operational information that can be reused for future customer questions.

The provider-facing side can update availability for the service, professional, or resource it is authorized to manage, but it does not silently move an existing customer reservation. If an availability change affects appointments, the system contacts affected customers and lets them select another valid slot.

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

Provider-side availability management and customer bookings are deliberately separated. Bookings may represent appointments, meetings, consultations, or other scheduled services.

The provider-facing side controls:

**Availability for the services, professionals, and resources within its authorized scope**

The customer controls:

**Their own appointments within that availability**

The front desk coordinates between both sides. A specific professional or resource is selected when the booking requires it, not by assuming that every customer conversation belongs to one individual professional.

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

## Human Handoff for Urgent, Sensitive, or Unresolved Issues

Customers may request a human at any point. The agent must not trap them in repetitive automated answers when they need a person. Complaints, suspected fraud, urgent incidents, technical support requests, and other issues requiring judgment or investigation should be routed to authorized staff through the existing provider-facing interface, with appropriate priority and a concise summary of the conversation so the customer does not need to repeat everything.

For emergencies or other time-critical risks, follow configured escalation rules and provide the appropriate verified emergency or official contact route; the bot must not imply that it has resolved the situation. If a human is not immediately available, explain the next step honestly and do not promise an unconfirmed response time. This uses the existing provider-facing side and does not require a separate support role or interface.

## Provider Answer and Return

The provider may respond with text, a document, a ZIP archive, or a link. When the information is available and validated for the relevant scope:

1. Store the source information as appropriately scoped operational knowledge.
2. Check **open pending questions** that the new information may answer, including questions from more than one customer.
3. Re-evaluate each candidate question against its authorized tenant/provider/customer scope and the new source; relevance alone does not prove that a question is resolved.
4. Send the specific answer or clarification back through the **original customer's conversation/channel**, even if they have asked other questions since.
5. Mark only the successfully resolved questions as complete; unrelated or insufficiently answered questions remain pending.

Source extraction or preparation of additional bounded text context may continue asynchronously without holding up a follow-up that can already be answered from validated source text.

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

# Internal Account and Data-Isolation Model

The hosted SaaS is **multi-tenant by default**. A tenant/account is an internal security and data-isolation boundary, not an additional conversational role that customers or providers need to manage explicitly.

The service-provider side may represent a solo professional or an authorized person/team managing an organization, chain, or multiple professionals. For example:

```text
Service-provider account
├── Shared operational information and booking rules
├── Authorized staff using one provider-facing interface
├── Professionals or resources (when relevant to schedules/bookings)
└── Customers and their reservations
```

The customer-facing and provider-facing sides remain the primary product interaction model. A clinic does not need a separate front-desk conversation or interface for every doctor. Organization-wide information and schedules are managed across the authorized scope; individual professional/resource IDs are used when needed for availability, reservation assignment, permissions, or a customer's explicit preference.

A solo professional may use the same model without a meaningful distinction between organization and individual. The internal data model can still represent an account/tenant boundary for isolation.

The normal hosted deployment should use:

```text
One shared frontend
        +
One shared containerized application runtime
        +
One shared relational database
        +
Many isolated tenants
```

The application should **not** create a separate relational database database or complete application instance for every provider or tenant by default.

Isolation is enforced through deterministic relational scope. `tenant_id` identifies the service-provider account and its data-isolation boundary. In knowledge and scheduling records, `provider_id` identifies a specific professional/resource only when relevant; it does not identify the provider-facing account or staff using its interface. `customer_id` identifies the customer when customer-specific scope applies.

The authenticated application context supplies the allowed identifiers. The AI does not choose its account, professional/resource, or customer scope.

A fully isolated application environment belongs to the **self-hosted deployment model**, not to ordinary SaaS signup.

---

# Operational Knowledge and Text-Based Context Storage

The current preferred design stores reusable operational knowledge as **ordinary text records in relational database**, together with relational metadata that defines where each record applies. The text is the source of truth. This is the initial approach for hosted and self-hosted deployments.

The application does not require a separate vector database, embeddings, or semantic search for the MVP. Knowledge is not split into independent physical databases or files for each customer; account-wide, professional/resource-specific, and customer-specific information uses the same logical record model with different enforced scopes.

A conceptual record may contain:

```text
OperationalKnowledge
├── id
├── tenant_id
├── scope                 // TENANT, PROVIDER, CUSTOMER
├── provider_id           // Professional/resource ID when relevant
├── customer_id           // Customer ID for private information
├── content               // Plain-text context
├── category              // Optional operational category
├── source_reference      // Source message, file, URL, or repository location
├── created_at
├── updated_at
├── valid_from            // Optional
├── valid_until           // Optional
└── active
```

Original documents, ZIP archives, URLs, and repository files may be supplied as **source material**. The application extracts or fetches useful text and stores source-linked text records or sections for the agent to use. An original file may be retained when needed for traceability, but the operational knowledge model is text-based rather than dependent on retrieving entire document files at answer time.

## Knowledge Scope

Knowledge has three logical scopes:

- **TENANT:** Information that applies across the service-provider account, such as general opening hours, shared policies, payment procedures, or organization-wide announcements. Both `provider_id` and `customer_id` are null.
- **PROVIDER:** Information specific to a professional, service, location, or resource. It includes the tenant and the relevant `provider_id`; `customer_id` is null.
- **CUSTOMER:** Private information or an exception associated with a verified customer. It includes the tenant and `customer_id`, with `provider_id` only when the information is also tied to a particular professional/resource.

These are logical scopes in the shared data model, not separate databases or a requirement to create one text file per customer.

### Tenant-Wide Knowledge

Examples include:

- Organization-wide opening hours and holiday closures.
- Parking information.
- General payment procedures.
- Common booking rules.
- Shared office instructions.

A tenant-wide record has `scope = TENANT`, with no professional/resource or customer association. It may be used across the relevant professionals and resources within that tenant, but never outside that tenant.

### Professional- or Resource-Specific Knowledge

Knowledge that applies only to one professional, service, location, or resource should be scoped to that target. Information that applies across the whole organization belongs in tenant-wide knowledge instead and should not be duplicated across individual professionals.

For example:

Customer:

> Does Dr. A offer consultations on Saturdays?

Provider-facing response:

> Dr. A offers consultations on Saturdays from 9 AM until noon.

The record has `scope = PROVIDER` and the relevant professional/resource ID. It can be used when that professional/resource is relevant to the current request or booking. A conversation must not be assigned to a professional merely because the customer spoke to that professional previously.

### Customer-Specific Knowledge

Some provider responses apply only to one customer and must never become general provider knowledge.

For example:

Customer:

> Can I come 30 minutes earlier than usual tomorrow?

Provider:

> Yes, Rahul can come at 9:30 AM tomorrow.

The record has `scope = CUSTOMER` and Rahul's verified customer ID. It may also include a professional/resource ID when the exception is tied to that target.

## Scope Classification and Safe Storage

The AI may interpret a provider's response and **propose** whether the information is tenant-wide, professional/resource-specific, or customer-specific. The authenticated application context supplies the tenant, provider/resource, and customer identifiers; the AI must not invent or select those identifiers.

The backend validates the proposed scope and the provider's authority before saving the record. Information must not be made broader merely because the model guesses that it sounds general:

- An explicit provider instruction that information applies to all relevant customers can be stored as tenant-wide knowledge.
- A response tied to one named customer or one customer's pending question defaults to customer-specific knowledge.
- Professional/resource-specific information is stored against that professional/resource only when the target is clear.
- If the scope is ambiguous, keep the narrower scope or ask the provider to clarify before making it generally available.

A response produced because of one customer's escalated question should default to **customer-specific** knowledge. It may become tenant-wide only when the provider explicitly confirms that it is generally applicable. This prevents a private exception from accidentally becoming clinic-wide policy.

## Context Assembly and Retrieval

The preferred initial retrieval method is to **provide the AI with the complete, authorized, reasonably sized text context** needed for the current request. The model can interpret that context and select the relevant facts without requiring a separate search engine.

For each customer request, the backend should:

1. Resolve the tenant and verified customer from authenticated channel identity.
2. Determine whether a particular professional/resource is relevant from the current request, an applicable booking, or an explicit customer choice.
3. Retrieve active tenant-wide text, applicable professional/resource text, and only that customer's private text.
4. Retrieve live structured records such as appointments and availability through their dedicated backend operations rather than relying on copied text.
5. Supply the bounded context to the AI through the AI integration layer.

The backend must apply authorization and scope filters **before** any text reaches the model. A customer request may receive:

```text
tenant_id = current_tenant
AND (
    scope = TENANT
    OR (
        scope = PROVIDER
        AND provider_id = current_professional_or_resource_when_relevant
        AND customer_id IS NULL
    )
    OR (
        scope = CUSTOMER
        AND customer_id = current_customer
        AND (
            provider_id IS NULL
            OR provider_id = current_professional_or_resource_when_relevant
        )
    )
)
```

It must exclude another tenant's knowledge, another customer's private information, and professional/resource-specific knowledge for an unrelated professional/resource.

When the authorized text collection is small enough, provide all applicable active text directly. Do not send every tenant record, every customer's information, or unbounded conversation history. Large requests must be bounded or paginated.

If the source collection becomes too large to provide as a whole, the backend can select additional context using explicit scope, category, source IDs, dates, or section references. The AI may request an additional approved source or section through a constrained backend operation; the backend still applies authorization and size limits. This does not require free-form database access or a semantic search engine.

For large uploaded sources, text may be divided into source-linked sections so only bounded portions need to be supplied. This is a content-management and context-size measure, not a requirement to generate embeddings or perform vector search.

### Specificity and Conflicts

When multiple applicable records address the same subject, prefer the most specific valid scope:

```text
Customer-specific
        ↓
Professional/resource-specific
        ↓
Tenant-wide
```

A specific temporary exception does not overwrite the general rule. Validity dates, status, source, and update history must be considered. If two active records at the same scope conflict and the system cannot determine which one is current, ask the provider rather than guessing. Similar wording or model confidence does not establish which factual statement is true.

### Provider Context Access

A provider may access tenant-wide knowledge, professional/resource-specific knowledge within their authorized scope, and customer-specific information only when the requested operation and authorization allow access to that customer. Provider access remains subject to deterministic application authorization.

## Knowledge Lifecycle and Time-Sensitive Context

Provider responses should not automatically become permanent truth forever. Knowledge records should support:

- Creation and last-modification timestamps.
- Optional effective and expiration dates.
- Active/inactive status.
- Source conversation or source reference.
- Provider identity and applicable scope.

Events, offers, temporary closures, and other dated announcements should retain their actual effective dates, times, and timezone where available.

- Resolve relative phrases such as "one week from now" against the **original message timestamp** and the provider's timezone, not the date of a future customer query. Ask for clarification if the date or time is genuinely ambiguous.
- Retain start/end dates and cancellations as structured validity information alongside the source text.
- Before answering questions about current or upcoming events/offers, compare those dates against the current time. Do not present expired or cancelled information as current.
- Historical information may remain available for questions about past events, but must be clearly described as past. Corrections and cancellations take precedence over stale announcements.
- If no reliable current information is available, ask the provider instead of assuming an old announcement still applies.

The system uses **authorized scope + source context + validity period + current state** when preparing context for the AI.

## Source Ingestion

Service providers can add knowledge through plain text, documents, uploaded ZIP archives, links, and public repositories (including GitHub).

- Plain text supplied directly by a provider can be stored as scoped operational knowledge.
- Uploaded files and archives are processed in bounded background jobs. Useful text is extracted and stored as source-linked records or sections with the appropriate tenant/provider/customer scope. Original files may be retained as source material where needed.
- Repository URLs can be saved as sources. Prefer fetching relevant files or sections when needed rather than requiring an upfront semantic index of the entire repository. Current public issues, pull requests, workflow runs, and available action logs can be fetched when a question requires fresh information.
- Source content is treated as data, not executable instructions. Only approved or publicly accessible material should be retrieved.
- Retain source references so responses can be grounded in the information actually supplied or retrieved.

If supplied context does not answer a question, the agent creates a persistent pending request and continues the current conversation. When the provider later supplies text, a file, or a link, the system checks authorized open pending questions against the new information and follows up with the original requesters. This matching can use bounded candidate questions and AI interpretation of the supplied text; vector search is not required for the workflow.

---

# Open Design Question: Semantic and Vector Search

Semantic/vector search is **not part of the current preferred implementation or MVP requirement**. The current design uses text as the canonical knowledge representation, direct provision of bounded authorized context, ordinary relational queries, and source/section retrieval when additional content is needed.

It remains an open design question whether a future deployment would benefit enough from semantic or hybrid search to justify the additional indexing, embedding, evaluation, and operational complexity. Reconsider it only if practical testing shows that the text-first approach regularly misses relevant information in large or varied source collections, or that the authorized text cannot be selected and supplied efficiently through metadata, source references, and bounded context assembly.

If that need is demonstrated, evaluate options such as conventional full-text retrieval, hybrid keyword/semantic retrieval, or relational database's native vector capabilities. Any such retrieval would remain optional and must operate only after deterministic tenant, professional/resource, and customer scope filtering. The original scoped text remains the source of truth; an embedding, if ever introduced, is only a derived retrieval aid. A separate vector database is not currently planned or required.

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

`get_authorized_operational_context()`

`get_source_section(source_reference, section_reference)`

`propose_operational_knowledge(content, proposed_scope)`

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
- Knowledge and source-context retrieval boundaries.

---

# Relational Data Storage

The product uses a **relational database** for operational state and scoped text-based knowledge. Domain models, business rules, and application services should remain independent of vendor-specific database features. The preferred implementation engine and hosting options are specified separately in the **Technology Stack** section.

The relational database stores organizations and service-provider accounts, customers and messaging identities, provider/resource information, provider availability and appointments, appointment status and reminder schedules, follow-up schedules and pending questions, spam records and conversation metadata, scoped operational knowledge text and source references, broadcast definitions, AI configuration references, audit records where required, and durable workflow and scheduled-job state.

The AI does not connect directly to the database.

---

# ORM and Data Access

The application uses an ORM as its primary relational data-access layer. The chosen ORM and its database connector are technology-stack decisions rather than product requirements.

The ORM handles normal relational operations, including scoped operational knowledge records and their metadata. Application code should use the data-access layer rather than spreading database-specific SQL throughout the codebase.

---

# Knowledge Service and Repository

A knowledge service/repository centralizes operations for storing, updating, and assembling authorized text context. It enforces scope rules and returns bounded source-linked text to application services.

Conceptually:

```text
Conversation / Provider Services
              ↓
        Knowledge Service
              ↓
        Relational ORM
              ↓
       Relational Database
```

The application should use operations such as:

```text
storeKnowledge(...)
getAuthorizedContext(...)
getSourceSection(...)
updateKnowledge(...)
deactivateKnowledge(...)
```

This keeps scope validation and context assembly in one place. It does not require a specific database engine, vector-specific SQL, embedding services, or a dedicated vector repository. A future retrieval strategy can be added behind the knowledge service if measured requirements justify it.

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
Knowledge Service / ORM
 │
 ▼
Relational Database
```

The application determines the organization, provider/resource, customer, role, permitted action, and permitted data scope before any database query is executed.

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
                         AI integration layer
          │
          └────────┬─────────┘
                   ▼
           Deterministic Tools
                   │
                   ▼
          Application Services
                   │
                   ▼
                 ORM
                   │
                   ▼
               relational database
```

The currently planned channel adapters are **WhatsApp, Telegram, and Discord**. Additional channels can later implement the same gateway interface.

The internal application works with a normalized message format instead of depending directly on WhatsApp, Telegram, or another messaging platform.


---

# AI Layer

The application should not depend directly on one model vendor.

**AI integration layer** will provide the application-level AI abstraction.

Conceptually:

```text
Conversation Engine
        │
        ▼
      AI integration layer
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

The application uses the AI integration layer for capabilities such as:

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
- A custom AI integration layer provider integration.

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
     └── Provider Options
              │
              ▼
            AI integration layer
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

The initial hosted SaaS should use a **container-first, event-driven architecture** to keep runtime behavior as consistent as practical across local development, floci-az, and Azure. managed container runtime is the preferred hosted runtime for containerized components, with Docker providing a common packaging model.

Start with the fewest containers that meet the product's needs: a shared application/API and a background worker/scheduler. Split channel adapters or other components into separate containers only when a platform protocol, isolation requirement, or measured scaling need justifies it. Share application contracts and infrastructure abstractions across those components.

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
Assemble Authorized Text Context
      ↓
AI integration layer
      ↓
AI Decision / Tool Call
      ↓
Deterministic Application Service
      ↓
ORM / relational database
      ↓
Persisted Event / Queue
      ↓
Containerized Worker
      ↓
Outbound Notification
```

Each incoming request and worker job should remain bounded.

Long-running workflows must use durable, resumable state and be decomposed into independent events, queued jobs, and bounded executions. Pending work must survive request completion, retries, and process restarts. The initial design should use relational database for durable job/workflow state and one queue implementation validated in floci-az for queued delivery; select one queue implementation by default and do not add a second queue provider without a concrete need. A queue is a delivery mechanism, not a complete workflow orchestrator; workflow state, idempotency, retries, scheduling, and recovery must be designed explicitly.

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

relational database stores due times and job state. The scheduler/dispatcher periodically discovers due work and enqueues bounded jobs. It must tolerate restarts and repeated scans, and use safe claiming or idempotency controls to prevent duplicate effects.

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
Query relational database for due jobs
      ↓
Enqueue due events/jobs
      ↓
Independent containerized workers
process the work
```

The scheduler/dispatcher should discover and dispatch work rather than perform every downstream operation itself. A future orchestration service may replace or complement this approach if it can be validated through floci-az.

---


# Appointment Reminder Example

An appointment can contain reminder state stored in relational database.

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
Update relational database
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
- Extracting and normalizing supplied files or archives into source-linked text.
- Matching newly supplied context to bounded candidate pending questions within authorized scope.
- Returning provider answers to the original customer conversations.
- Refreshing source material when requested or scheduled.
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
Store/validate scoped text context
      │
      ▼
Identify candidate open questions
within authorized scope
      │
      ▼
Check whether the new context answers each question
      │
      ▼
Send answers asynchronously
to original customer channels
      │
      ▼
Mark answered questions resolved
```



# Technology Stack

The product requirements and core architecture remain independent of specific vendors. The following are current implementation preferences and deployment options, not customer-facing product requirements.

## Application and AI

- **Application framework:** Next.js.
- **UI styling:** DaisyUI.
- **AI integration:** AI SDK, behind an application-level model/provider abstraction.
- **Relational data access:** Prisma ORM.
- **Additional runtimes:** Only when a concrete requirement justifies them.

## Relational Database

**PostgreSQL is the first-choice and first-class database target.** It is the preferred starting point for development, testing, hosted deployment, and self-hosted operation.

PostgreSQL is preferred because the same established relational engine can be self-hosted, packaged for supported CPU architectures including ARM, and obtained from multiple managed database providers. The PostgreSQL project documents ARM among the architectures on which PostgreSQL can be expected to work. Platform-specific builds, container images, extensions, and drivers must still be validated.

Possible PostgreSQL deployment options include:

1. **Azure:** Azure Database for PostgreSQL Flexible Server for managed Azure hosting.
2. **Other managed providers:** Amazon RDS for PostgreSQL, Google Cloud SQL for PostgreSQL, and other managed PostgreSQL services.
3. **Self-hosted:** Community PostgreSQL running in the organization's own infrastructure or containers, including supported ARM-based environments.

These providers offer PostgreSQL-compatible database services, but they do not guarantee identical extensions, versions, connection behavior, compute architectures, or operational features. Validate the features required by the application against the selected provider and region instead of assuming every service is interchangeable.

Other relational database engines may be evaluated as future alternatives if a concrete deployment requirement justifies them. The MVP does not promise equal support for multiple database engines; PostgreSQL is the initial supported target.

## Knowledge Storage

- Store operational knowledge as ordinary scoped text records in the relational database.
- Keep source references, validity metadata, and account/provider/customer scope with each record.
- No separate vector database or embedding service is required for the preferred implementation.
- Semantic or vector retrieval remains an open future evaluation, not an MVP dependency.

## Runtime and Infrastructure

- **Local and self-hosted packaging:** Docker and containerized application services.
- **Preferred Azure container runtime:** Azure Container Apps.
- **Queue/event delivery:** Azure Queue Storage or Azure Service Bus; select one after validating the required features through floci-az.
- **Infrastructure as code:** Terraform where applicable.
- **Local Azure-oriented validation:** floci-az.
- **Execution model:** Event-driven processing with durable workflow/job state and bounded worker jobs.

---

# Infrastructure Direction

The runtime should remain container-first and event-driven. Keep application services, durable workflow state, queue delivery, and scheduling behind application-level contracts so the same workflows can run in local development, hosted infrastructure, and self-hosted deployments. Use the fewest separate services that satisfy protocol, isolation, reliability, and measured scaling requirements.

Infrastructure should be reproducible using infrastructure-as-code tools wherever applicable. The same container images should run locally and in hosted environments wherever practical. Keep application logic sufficiently decoupled from specific cloud services to support local and self-hosted operation.

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

Azure is a supported hosted deployment target. The Azure deployment should use:

- A managed container runtime for the frontend, API, channel-adapter, and background-worker components as appropriate.
- A managed relational database service for operational data, scoped text-based knowledge, and durable workflow/job state.
- One queue implementation for event/job delivery, selected according to the behavior validated through floci-az.
- The selected ORM/data-access layer and knowledge service.
- Event-driven processing and bounded worker jobs.
- The configured AI integration and messaging providers.

The specific database engine and managed database offering are technology-stack choices, not requirements of the product's domain model. The project will prioritize cost-conscious services while validating the exact service behavior required by the application.

The architecture should remain container-first, event-driven, cost-conscious, horizontally scalable, and able to keep persistent replicas only for components that need them, such as a Discord Gateway adapter or a background worker. Azure is a supported deployment target rather than a mandatory runtime dependency.

# Azure-Independent Operation

A deployment should also be capable of running without Azure.

A service provider or organization may operate the system using:

- Docker.
- floci-az.
- Local/private infrastructure.
- A compatible relational database.
- Compatible source/file storage where supplied files need to be retained.
- A compatible queue/event-delivery mechanism.
- A local/private AI endpoint.
- Their own messaging and networking configuration.

For the MVP, Azure-independent operation means validating the same containerized application and required supporting services through floci-az, using the preferred relational database implementation. Supporting arbitrary alternative database engines or queues is not required unless a concrete need justifies it.

The abstraction should allow equivalent local implementations of:

- Containerized application and worker execution.
- Queue/event delivery.
- Timed jobs and scheduled dispatch.
- Durable asynchronous workflow execution, using equivalent supported mechanisms where necessary.
- Relational storage and scoped text-based knowledge.
- Source/file storage where uploaded source material needs to be retained.

without rewriting the higher-level business logic.

# Future Infrastructure Exploration

## Azure Static Web Apps and Durable Functions

Azure Static Web Apps (SWA) and Azure Durable Functions are **future exploration candidates, not dependencies of the initial MVP**. Revisit them once floci-az supports the required behavior well enough for local development and end-to-end validation.

- SWA may be evaluated as an optional frontend-hosting approach if it reduces operational complexity without undermining local/Azure parity.
- Durable Functions may be evaluated as an orchestration option for durable multi-step workflows if the required trigger, orchestration, retry, and replay behavior can be validated through floci-az.

Until then, use containerized application components, relational database-backed workflow/job state, and queue-driven workers. Keep workflow logic behind application-level abstractions so a future scheduling or orchestration implementation can be introduced without rewriting business logic.

---

# Deployment Model

Two primary distribution models are planned.

## Hosted SaaS

The project operates the infrastructure and organizations subscribe to the service.

The default hosted architecture is shared and multi-tenant. The initial product interaction is messaging-first; if a web frontend is included, it belongs to the provider-facing setup/administration side and does not introduce a third product-facing role. Public customer website chat remains future scope, not an MVP requirement.

```text
Shared web frontend
        +
Shared containerized API and worker services
        +
Shared relational database database
        +
Tenant/provider/customer scoped data
```

Ordinary account creation does not provision a new application stack or relational database database.

The hosted version may provide:

- Azure-hosted infrastructure.
- managed container runtime for containerized application services.
- relational database for relational data and scoped text-based operational knowledge.
- Queue-backed event delivery and containerized background workers.
- Durable scheduled-job state and dispatch.
- Messaging integrations.
- Platform-managed AI access.
- AI integration layer integrations.
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
+ managed container runtime
+ relational database
+ Queue / Worker Services
+ Platform AI
```

```text
Azure Deployment
+ managed container runtime
+ relational database
+ Queue / Worker Services
+ Provider's AI API
```

```text
Azure Deployment
+ managed container runtime
+ relational database
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
│   ├── managed container runtime
│   ├── Queue Storage / Service Bus
│   └── relational database
│
└── Local / Self-Hosted / floci-az
    ├── Docker Containers
    ├── Compatible Queue / Event Delivery
    └── Compatible Relational and Source Storage

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
                 Context Layer     AI integration layer
                       │             │
                       │      ┌──────┼────────┐
                       │      │      │        │
                       │      ▼      ▼        ▼
                       │   Platform Provider Local
                       │      AI     API      API
                       │
                       ▼
              Authorized Text Context
                       │
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
                           ORM
                              │
                              ▼
                          relational database
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
       Containerized Workers      relational database Job State
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
7. Retrieve required structured records
        ↓
8. Assemble active, authorized text context
        ↓
9. Resolve organization's AI configuration
        ↓
10. Send context through AI integration layer
        ↓
11. AI determines response/tool request
        ↓
12. Backend validates requested operation
        ↓
13. Execute deterministic operation
        ↓
14. Persist state through ORM/relational database
        ↓
15. Emit events where necessary
        ↓
16. Generate response
        ↓
17. Send through original messaging channel
```

At no point should the AI become the authority for:

- Identity.
- Authorization.
- Tenant boundaries.
- Customer boundaries.
- Database integrity.
- Appointment state.
- Which private or professional/resource scope it may access.

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
Validate and store scoped text context
             ↓
Identify candidate open questions within authorized scope
             ↓
Check each against the new context and answer quality
             ↓
Reply to each original requester via their channel
             ↓
Resolve answered requests; leave others pending
```

Each pending record keeps enough context to identify the original question, customer, provider, and destination conversation. A single provider response may resolve several related questions, but unrelated questions must remain open.

The agent does not have to wait for provider response before accepting further input from any user. This non-blocking follow-up is a core capability, not an industry-specific behavior.

# Scheduled Job Lifecycle

```text
1. Containerized scheduler/dispatcher runs
        ↓
2. Query relational database for due jobs
        ↓
3. Claim/mark work for processing
        ↓
4. Enqueue one job per operation
        ↓
5. Independent containerized worker handles each job
        ↓
6. Perform deterministic action
        ↓
7. Send notification if required
        ↓
8. Persist status through ORM/relational database
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

## Scheduled Work Discovery

The scheduler/dispatcher should discover durable due work and enqueue it rather than perform the downstream work itself.

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

## Supplied Context May Be Insufficient

The presence of text context does not automatically make it a valid answer. If authorized context does not clearly answer the customer's question, or its source or validity is uncertain, the system should treat the answer as unknown and escalate to the provider rather than guess.



## Knowledge Conflicts

If two active records at the same scope conflict and metadata cannot determine which one is current, the system should ask the provider.

Language similarity or model confidence must not be used to decide which factual statement is true. Apply source, scope, status, and validity rules; ask the provider when an active same-scope conflict cannot be resolved deterministically.

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
- Respond to escalated customer questions, including prioritized human handoffs for complaints, suspected fraud, technical issues, urgent incidents, and other cases requiring human judgment.
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
- Request a human response and receive a clear handoff or next step for unresolved, sensitive, or urgent issues.

## Operational Knowledge

- Plain-text operational context stored as scoped relational database records.
- Account-wide, professional/resource-specific, and customer-specific knowledge.
- AI-assisted scope classification, with backend validation and provider confirmation before information is made broadly available.
- Direct provision of complete, authorized, reasonably sized context to the AI.
- Source-linked text extraction from supported documents, ZIP archives, URLs, and public repositories.
- Bounded retrieval of approved source sections or current external information when needed.
- Non-blocking provider escalation for missing or unsatisfactory answers.
- Durable pending-question tracking and asynchronous follow-up to original channels.
- Matching newly supplied context to candidate unresolved questions within authorized scope.
- Deterministic account/customer filtering, with professional/resource filtering only when relevant.
- Source references, validity metadata, and optional knowledge expiration.
- No required embedding generation, vector storage, or semantic search in the MVP.

## Automation

- Appointment confirmation.
- Configurable reminders before appointments or meetings (for example, one day, 20 minutes, or 10 minutes in advance).
- Appointment confirmation requests.
- Provider schedule-change notifications.
- Rescheduling workflows.
- Routine follow-ups.
- Persisted scheduled-job dispatch.
- Provider broadcasts.
- Retry processing.

## Channels

- Telegram.
- WhatsApp.
- Discord.

## AI

- AI integration layer.
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
- Human handoff for urgent, sensitive, or unresolved requests, using the existing provider-facing interface.

## Database

- relational database for hosted deployments.
- ORM for normal relational operations.
- Text-based operational knowledge stored and retrieved through the scoped knowledge service.
- No direct AI-to-database access.
- Deterministic account/customer scoping, with professional/resource scope when relevant.

## Infrastructure

- managed container runtime for hosted containerized services.
- Start with a shared application/API container and a background worker/scheduler; separate channel adapters only where platform protocols require them.
- One queue implementation selected according to verified floci-az support.
- relational database for operational and durable workflow/job state.
- ORM.
- AI integration layer.
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
- Connect directly to relational database.
- Request or receive context outside the backend-authorized scope.
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

- A mandatory vector database or vector-search service.
- A separate relational database database per tenant.
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

3. **Internal data isolation**  
   Account/tenant, provider/resource, and customer boundaries are enforced in relational data before records reach the AI. These implementation boundaries support the two primary product sides and do not require a separate user-facing front desk for each professional.

4. **Context retrieval follows authorization**  
   The backend applies tenant, professional/resource, and customer scope before any text reaches the AI. Any future search mechanism must preserve this rule.

5. **Model independence**  
   AI integration layer separates application logic from the underlying model/provider.

6. **Infrastructure independence**  
   Azure is a supported hosted target while floci-az and compatible local services enable local and self-hosted operation.

7. **Database abstraction**  
   ORM provides the primary relational data-access layer.

8. **Knowledge service abstraction**  
   Scoped text storage, source references, and context assembly remain behind a knowledge service rather than being scattered throughout the application.

9. **Container-first runtime and bounded execution**  
   Containerized services maximize local/Azure runtime parity. Requests and worker jobs remain bounded even when adapters or workers need persistent processes.

10. **Event-driven workflows**  
    Long-running operations use durable workflow state, events, and bounded executions rather than relying on a single process to remain active.

11. **Durable scheduled work**  
    Due work is persisted and dispatched to workers. The initial implementation uses scheduling and queue mechanisms validated through floci-az rather than depending on Azure Durable Functions.

12. **Channel independence**  
    Telegram, WhatsApp, and Discord are adapters around a common messaging interface. Adapters may use webhooks or persistent connections according to each platform's protocol.

13. **Provider-side/customer-side coordination**  
    The provider-facing side manages availability and operational rules across its authorized scope; customers manage their own bookings within that availability. Individual professionals or resources are assigned only where the booking or authorization requires it.

14. **Minimal healthcare scope**  
    The healthcare implementation manages service operations and scheduling, not medical records or clinical decision-making.

15. **AI as an interface, not an authority**  
    The model understands language, organizes information, and proposes actions; deterministic application services control what actually happens.

16. **Knowledge should improve over time**  
    Missing information can be escalated to the provider, stored as text with appropriate scope and source metadata, and reused in later authorized context.

---

# Product Positioning

The product is not primarily:

> A chatbot for answering customer questions.

It is:

> An AI-operated front desk that coordinates customers and service providers through existing messaging platforms while executing tightly controlled operational workflows.

Its advantage is the combination of:

**Conversation + scoped text-based operational context + scheduling + controlled actions + provider escalation + proactive notifications + scheduled automation + deployment independence + model independence.**

Instead of ending a conversation when information is unavailable, the system can obtain the missing information from the service provider, answer the customer, and retain appropriately scoped operational knowledge for future requests.

Account-wide answers become reusable account-wide knowledge; professional/resource-specific answers remain scoped to the relevant professional or resource. Customer-specific answers remain associated with that customer and cannot become visible to unrelated customers.

The hosted SaaS shares application and database infrastructure by default while enforcing deterministic internal account, provider/resource, and customer boundaries before authorized text context reaches the AI. These implementation details support the simple customer-facing/provider-facing product model rather than defining additional conversational roles.

relational database provides relational operational state and scoped text-based operational knowledge for hosted deployments, while ORM remains the primary application data-access abstraction.

AI integration layer prevents the application's conversational layer from becoming tightly coupled to one model vendor and allows hosted, customer-provided, private, or local model configurations.

A containerized scheduler/dispatcher and queue-driven workers handle scheduled work and retries, with durable job state stored in relational database.

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

1. **Connect sources.** Maintainers give the agent general organization instructions, documentation, uploaded ZIP archives, and links to public GitHub repositories. The system extracts useful text into source-linked sections and can fetch selected files or current public issues, pull requests, workflow runs, and available action logs when a question needs fresh information.
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

Initially, the generalized extension focuses on **rescheduling or modifying existing reservations in additional reservation domains or connected systems**, while retaining the appointment and meeting workflows already defined. Appointment/meeting creation and cancellation remain part of the core front-desk workflow; creation, cancellation, and complex modifications for additional reservation types can be introduced incrementally.

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
