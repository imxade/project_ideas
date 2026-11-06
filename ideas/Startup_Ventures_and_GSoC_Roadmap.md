# Startup Venture and Open-Source Project Portfolio

> Living portfolio proposal. This document consolidates the venture ideas identified so far, separates commercial products from reusable open-source foundations, and maps potential work to GSoC-sized contributions. It is additive and does not replace existing proposal files.

## 1. Executive Summary

The goal is to build a portfolio of focused technology ventures supported by useful open-source infrastructure. The portfolio currently includes:

1. **Solly** — named venture concept, product definition pending.
2. **Application-based job board** — job discovery and application workflow platform.
3. **AI Front Desk Agent** — conversational operational layer for service providers and customers.
4. **Crashkaro** — named venture concept, product definition pending.
5. **Browser-accessible VM interface** — browser-first virtual machine experience, from native QEMU to browser-local WebAssembly experimentation.
6. **Automated Model Builder** — reproducible model training and evaluation from task requirements and examples.
7. **Merchant of Record Orchestrator** — provider-neutral integration layer for multiple MoR services.
8. **Static Product Demo and Tutorial Generator** — converts an authorized product into a backend-independent interactive demo.

These are not eight projects to build simultaneously. The portfolio should distinguish:

- Venture products with a clear user, problem, workflow, and business model.
- Open-source foundations that are independently useful.
- GSoC-sized contributions with bounded scope and measurable deliverables.
- Shared infrastructure that can reduce duplicated engineering.

GSoC is intended to support contributions to participating open-source organizations. It is not a funding mechanism for private product development. A GSoC proposal must fit the upstream project's needs, mentor capacity, contribution process, and official program rules.

## 2. Portfolio Strategy

### 2.1 Open-source and commercial boundary

| Layer | Suggested approach |
|---|---|
| Protocols, SDKs, adapters, interoperability layers | Open source |
| Core execution interfaces and reference implementations | Open source |
| Reproducible benchmarks and test suites | Open source |
| Documentation and developer tooling | Open source |
| Hosted operations and managed infrastructure | Potentially commercial |
| Enterprise support, compliance work, and managed integrations | Potentially commercial |
| Product-specific workflow and user experience | Decide transparently per product |

The open-source core should remain useful without the commercial service. Commercial value should come from hosting, operational guarantees, support, managed integrations, collaboration, or enterprise deployment rather than artificial restrictions.

### 2.2 Potential shared foundations

- Permissioned agent tool execution.
- Connector and integration SDKs.
- Durable jobs, retries, and event processing.
- Browser automation and isolated execution.
- VM lifecycle and remote display interfaces.
- Artifact storage and reproducible execution records.
- Evaluation, telemetry, and benchmarking.
- Identity, authorization, and tenant isolation.

Extract shared components only when multiple real use cases justify a stable interface. Do not build a universal platform before validating individual products.

## 3. Venture Ideas

## 3.1 Solly

**Status: definition required.**

The current notes identify the name but do not specify the product. Do not infer a product from the name. Before technical planning, define:

- One-sentence problem statement.
- Primary user and buyer.
- Existing workaround and competitors.
- Main user journey.
- First useful release.
- Data and integration requirements.
- Open-source core and commercial boundary.
- One measurable validation experiment.

Until this information is available, Solly remains a portfolio placeholder.

## 3.2 Application-Based Job Board

### Problem

Traditional job boards focus on listings. Applicants still manage applications across company portals, forms, email, referrals, spreadsheets, and messaging tools. Status, deadlines, document versions, contacts, and follow-ups become fragmented.

### Product direction

An application-centric job platform where each opportunity has a structured application workflow.

### Candidate interface

- Job discovery, filters, and saved searches.
- Candidate profile and reusable application information.
- Application workspace and checklist.
- Resume and document version tracking.
- Pipeline or kanban view.
- Deadlines, calendar, and follow-up reminders.
- Interview preparation, notes, and contact history.
- Application status and submission evidence.

### Employer interface

- Job creation and publishing.
- Application review.
- Candidate communication.
- Interview scheduling and hiring pipeline.

### Example workflow

1. Candidate saves a job.
2. The platform extracts requirements and creates a checklist.
3. Candidate selects a resume version and supporting documents.
4. Candidate reviews each field and submits through a permitted employer application route.
5. Platform records destination, timestamp, and confirmation evidence.
6. Candidate tracks status and schedules follow-ups.
7. Interview dates and preparation notes remain attached to that application.

An assistant may help search, prepare, organize, and track applications. It must not submit applications or contact employers without explicit authorization and review.

### Business hypotheses

- Free candidate tier with paid productivity features.
- Employer hiring workflow subscriptions.
- Integrations for universities, bootcamps, and placement cells.
- Optional transparent career services.

### Open-source opportunities

- Job/application interchange schema.
- Resume and job-description parsing evaluation set.
- Browser form automation test harness with user consent.
- Calendar and application tracker integrations.

### Validation

Interview job seekers and placement teams. Measure application completion time, missed follow-ups, duplicate data entry, and repeat usage. Validate whether the first product should focus on discovery, application tracking, or both.

## 3.3 AI Front Desk Agent

**Existing proposal:** [Frontdesk_Agent.md](https://github.com/imxade/project_ideas/blob/master/ideas/Frontdesk_Agent.md)

### Problem and product

Service providers receive repeated questions and operational requests through messaging channels. FAQ bots fail when information is missing, bookings need to change, or a human must intervene.

Build an AI-operated front desk that coordinates customers and providers through familiar messaging channels. It combines scoped knowledge retrieval, scheduling, controlled actions, notifications, and asynchronous human escalation.

### Users and interfaces

- **Customer interface:** WhatsApp, Telegram, Discord, or web chat.
- **Provider interface:** protected dashboard or authorized messaging interface for schedules, pending questions, knowledge sources, and announcements.
- **Internal boundary:** tenant/account, professional/resource, and customer scopes for deterministic authorization.

### Main workflows

- Ask operational questions.
- Search services and availability.
- Book, reschedule, and cancel supported appointments.
- Receive confirmations and reminders.
- Receive provider announcements.
- Escalate missing or unsatisfactory answers.
- Let authorized staff answer pending questions.
- Reuse validated answers as appropriately scoped knowledge.
- Route urgent or sensitive issues to a human.

### Architecture and safety

The model must not have unrestricted database access. The backend retrieves only the authorized tenant, provider/resource, and customer context. State changes happen through deterministic backend operations with validation and confirmation.

A missing answer creates a durable pending question. The customer is informed that the provider will be asked, while unrelated conversation continues. When the provider responds, the system checks which pending questions are actually answered and follows up through the original channel.

Healthcare use remains limited to front-desk operations and scheduling unless a separately governed clinical product is designed.

### Open-source opportunities

- Channel adapter SDK.
- Permissioned tool-execution framework.
- Durable human-escalation workflow.
- Tenant-scoped retrieval reference implementation.
- Messaging webhook test harness.
- Scheduling interoperability API.

Commercial offerings may include hosting, organization management, managed channel integrations, support, analytics, and enterprise deployment.

## 3.4 Crashkaro

**Status: definition required.**

The available notes do not establish what Crashkaro does. The name alone cannot establish whether it is a crash-reporting product, vehicle-related service, developer tool, or another category.

Define the exact problem, user, buyer, event or workflow handled, first end-to-end use case, required integrations, sensitive data, safety/legal constraints, and open-source/commercial boundary. Do not force it into another venture's architecture without evidence.

## 3.5 Browser-Accessible VM Interface

### Working tagline

**A computer in your browser. No local VM setup required.**

### Problem

Virtual machines commonly require local installation, compatible hardware, downloads, configuration, and platform-specific setup. Sharing a reproducible environment is cumbersome. Browser VM projects already reduce some of this friction, but differ in supported architectures, operating systems, devices, persistence, performance, and isolation.

### Product vision

A browser-first interface to create, launch, access, and manage virtual machines. It should support two execution modes:

1. **Native or remote execution**, where QEMU runs on a host or server and the browser provides control and display.
2. **Browser-local execution**, where a compatible emulator or runtime compiled to WebAssembly executes within the browser sandbox.

Both modes should implement a stable VM session abstraction. The UI and higher-level workflows should not depend on a specific backend.

Important technical distinction: WebAssembly does not provide direct access to host hardware virtualization or arbitrary host devices. A QEMU WebAssembly port may face constraints involving system calls, threading, networking, storage, device emulation, memory, and browser APIs. Browser-local execution must not be presented as equivalent to unrestricted native QEMU.

### Why build it

- Reduce setup friction for education and demonstrations.
- Make reproducible development and testing environments easier to share.
- Provide temporary isolated environments for technical tasks.
- Support browser-based workshops and labs.
- Enable agent-controlled execution environments with explicit permissions.
- Explore local/offline execution where the browser and runtime permit it.

### Use cases

- Launch a Linux development environment from a link.
- Demonstrate an OS or application without local installation.
- Provide a reproducible classroom lab.
- Run a temporary sandbox for a coding exercise.
- Demonstrate Android apps where the emulator and graphics/device stack permit it.
- Access a Windows guest through native or remote QEMU first, subject to licensing and image requirements.
- Give an authorized agent a controlled environment for code execution or file inspection.

### User interface and interaction

The web application should provide:

- Environment catalogue and templates.
- Create/launch environment.
- OS and architecture selection where supported.
- CPU, memory, and storage configuration where supported.
- Boot and lifecycle status.
- Console or graphical display.
- Keyboard and pointer input.
- File upload/download and shared folders where supported.
- Snapshot and reset controls where supported.
- Session sharing with explicit permissions.
- Usage indicators, logs, and diagnostics.
- Clear indication of native/remote versus browser-local execution.

A user selects a template and supported resources. The frontend requests a session from an execution adapter. The adapter returns a session identifier, capabilities, connection metadata, and lifecycle events. The UI connects to display/console transport and sends input through a defined protocol.

### Architecture abstraction

    Web UI
      |
      v
    Session API / Control Plane
      +-- Identity and Authorization
      +-- Template and Image Registry
      +-- Session Lifecycle
      +-- Capability Negotiation
      +-- Usage and Audit Events
      |
      v
    Execution Adapter Interface
      +-- Native QEMU Adapter
      |     +-- QMP lifecycle/control
      |     +-- Display transport
      |     +-- Storage and image management
      |
      +-- Remote QEMU Adapter
      |     +-- Isolated worker
      |     +-- Network/display gateway
      |
      +-- Browser WASM Adapter
            +-- WebAssembly emulator/runtime
            +-- Browser storage
            +-- Browser display/input APIs
            +-- Worker and memory constraints

Candidate adapter operations:

- createSession
- startSession
- pauseSession
- resumeSession
- stopSession
- resetSession
- getSessionState
- getCapabilities
- sendInput
- connectDisplay
- snapshotSession
- restoreSession
- transferFile

Not every backend must support every operation. Unsupported capabilities must be explicit.

### Existing projects and prior art

These are starting points for research, not proof that the proposed product is novel.

| Project | What it demonstrates | What to investigate |
|---|---|---|
| [QEMU](https://github.com/qemu/qemu) | General-purpose machine emulator and virtualizer | Native baseline, device models, QMP, portability |
| [WebVM](https://github.com/leaningtech/webvm) | Virtual machine experience in a browser | Browser execution architecture and constraints |
| [v86](https://github.com/copy/v86) | x86 PC emulator and x86-to-Wasm JIT in browser | Browser-local emulation, devices, performance, snapshots |
| [qemu-wasm](https://github.com/ktock/qemu-wasm) | Existing QEMU-on-browser experiment/fork | Build status, patches, targets, limitations, maintenance |
| [Firecracker](https://github.com/firecracker-microvm/firecracker) | Server-side microVM technology | Server isolation and lifecycle concepts, not browser execution |
| [Blink](https://github.com/jart/blink) | Lightweight x86-64 Linux emulator | Alternative emulation approach and scope |

Also investigate noVNC, SPICE clients, WebSocket gateways, WebRTC, Web Workers, SharedArrayBuffer, OPFS, IndexedDB, and browser security restrictions. Evaluate before adopting.

### Native QEMU to WebAssembly roadmap

#### Stage 0 — Prior-art and feasibility audit

Deliverables:
- Compare QEMU, WebVM, v86, qemu-wasm, and related projects.
- Record architectures, OS support, devices, display, storage, networking, persistence, and browser requirements.
- Inspect whether qemu-wasm builds and what its demo actually proves.
- Record licenses and upstream contribution paths.

Validation:
- Reproduce one existing browser VM demo.
- Build native QEMU on a documented host.
- Document browser and hardware requirements.
- Do not claim Windows or Android browser-local support without a working proof.

#### Stage 1 — Native QEMU control proof

Build a minimal Linux guest on a Linux host. Configure CPU, memory, disk, and network. Control lifecycle through QMP and capture state/logs.

Validation:
- Automated guest boot test.
- QMP commands produce expected state transitions.
- Shutdown and failure cases are observable.
- No unmanaged processes remain after cleanup.

#### Stage 2 — Native display and input

Connect the browser to native QEMU through a selected display protocol, initially evaluating VNC/SPICE and established browser clients. Forward keyboard and pointer input, support reconnects, and separate control from display traffic.

Validation:
- Boot and interact with a Linux desktop in browser.
- Measure latency and rendering performance.
- Test reconnect behavior.
- Verify authentication and session isolation.

#### Stage 3 — Backend-neutral session abstraction

Define lifecycle API, capability model, session metadata, resource configuration, image references, display/input contract, file transfer, persistence, errors, and audit events.

Validation:
- Native QEMU adapter passes contract tests.
- Mock adapter runs frontend tests.
- Unsupported features are reported accurately.
- UI has no QEMU-specific lifecycle assumptions.

#### Stage 4 — Remote native QEMU

Move QEMU to isolated server-side workers. Add scheduling, resource limits, authenticated control/display gateways, image management, cleanup, persistent state, and logs.

Validation:
- Users cannot access each other's sessions.
- Quotas and cleanup are enforced.
- Worker restart and failed boot are handled.
- Public network exposure is minimized.
- Browser never receives host credentials.

#### Stage 5 — Browser-local emulator baseline

Before porting QEMU, integrate an existing browser emulator such as v86. Load a supported image, render display, accept input, and measure memory, startup time, responsiveness, browser compatibility, and persistence.

Validation:
- Works without a remote VM worker after assets load.
- Capabilities and limitations are documented.
- Refresh/storage behavior is tested.
- Performance results are reproducible.

#### Stage 6 — QEMU WebAssembly feasibility prototype

Choose a minimal QEMU target and device configuration. Identify unsupported system calls and dependencies. Evaluate WASI versus browser-specific Emscripten/WebAssembly integration. Adapt platform interfaces behind a narrow compatibility layer. Defer networking and complex devices until a minimal guest boots.

Validation:
- Reproducible build.
- Minimal guest boots in browser, or a blocker report explains why not.
- Unsupported features listed.
- CPU and memory measured against native QEMU and v86.

A successful port is not assumed. If QEMU's system interfaces make browser execution impractical, preserve the session abstraction and use a suitable browser-native emulator for local execution.

#### Stage 7 — Storage, networking, persistence

Experiment with OPFS/IndexedDB-backed disks, image import/export, snapshots, browser-compatible networking, quotas, cleanup, and worker recovery.

Validation:
- Persistent disk survives reload when enabled.
- Export/import works with a compatible native environment.
- Quota exhaustion and interrupted writes are handled.
- Network and browser security constraints are documented.

#### Stage 8 — OS and workload expansion

Expand to lightweight Linux, CLI development images, educational OS images, Android environments where compatible, and Windows guests through native/remote QEMU first.

Validation:
- Per-OS support matrix.
- Automated boot, input, storage, shutdown tests.
- Licensing and image redistribution respected.
- Performance measured on defined browsers and hardware.

#### Stage 9 — Product hardening

Add identity, revocable sharing, quotas, abuse prevention, observability, image integrity, accessibility, self-hosting, and security review.

### Suggested technology choices

| Area | Candidate | Caveat |
|---|---|---|
| VM engine | QEMU | Mature and broad device support |
| VM control | QMP | Structured control protocol |
| Browser display | noVNC or tested SPICE/WebRTC path | Select after latency/feature comparison |
| Frontend | TypeScript and React | Common web ecosystem |
| Control API | TypeScript or Rust service | Keep independent from VM engine |
| Native worker | Linux host with QEMU | Containers alone are not a complete VM security boundary |
| Browser runtime | WebAssembly, Web Workers, browser storage APIs | Browser constraints apply |
| Session state | PostgreSQL or transactional relational store | Durable lifecycle and ownership |
| Interactive transport | WebSocket initially | Separate real-time traffic from durable jobs |
| Packaging | OCI images and reproducible build scripts | Portable worker deployment |
| Testing | QEMU integration, browser automation, protocol contract tests | Validate layers independently |

### Security model

Treat guest images and guest network traffic as untrusted. Run native VMs least-privileged with explicit limits. Never expose QMP publicly. Authenticate all session/display/control connections. Isolate storage. Validate uploaded images. Restrict egress where needed. Browser sandboxing is not a substitute for security review.

### Business hypotheses

- Hosted VM sessions billed by usage.
- Managed classroom and workshop environments.
- Private team workspaces.
- Enterprise/self-hosted deployment and support.
- Preconfigured development/test environments.
- Permissioned agent execution environments.

### GSoC-sized candidates

- QMP session adapter and conformance tests.
- Browser display/input integration and reconnect handling.
- Browser-local v86 integration behind a stable runtime API.
- Reproducible QEMU build and compatibility report.
- Narrow-target QEMU WebAssembly feasibility prototype.
- Browser-backed virtual disk persistence.
- VM capability discovery and UI integration.

Each needs an interested upstream organization and mentor. A student deliverable must remain useful even if the full product is not completed.

## 3.6 Automated Model Builder

**Existing proposal:** [model_builder.md](https://github.com/imxade/project_ideas/blob/master/ideas/model_builder.md)

Users define input/output schemas, provide examples, set quality/resource constraints, and receive an evaluated, reproducible model artifact.

Workflow:
1. Define task and schemas.
2. Upload or create examples.
3. Validate and version dataset.
4. Analyze task and constraints.
5. Generate bounded candidate configurations.
6. Run inexpensive trials and eliminate weak candidates.
7. Optimize promising candidates.
8. Evaluate against benchmark.
9. Select using explicit constraints.
10. Package model, schemas, preprocessing, metadata, and evaluation.
11. Collect feedback and support retraining.

Interface:
- Task setup.
- Dataset/schema editor.
- Training dashboard.
- Candidate comparison.
- Evaluation report.
- Model registry.
- Model Pack export/inference endpoint.
- Feedback and retraining view.

Open-source opportunities include dataset validation, evaluation harness, model packaging specification, provenance format, and benchmarking. Do not promise every model family or guaranteed quality. Publish a supported task matrix.

## 3.7 Merchant of Record Orchestrator

**Existing proposal:** [MoR_orchestrator.md](https://github.com/imxade/project_ideas/blob/master/ideas/MoR_orchestrator.md)

A stateless layer between an application and multiple MoR providers, initially considering Dodo and Paddle.

Requirements:
- Provider selection by country, currency, product, tax, subscription, and capability.
- Multiple providers and explicit fallback.
- Customer, payment, subscription, invoice, and history continuity.
- Idempotent events and duplicate/late/out-of-order webhook handling.
- Provider API version compatibility.
- Gradual migration without forcing all subscriptions to move.
- Historical transaction interpretation after integration changes.

Open-source opportunities:
- Provider adapter SDK.
- Canonical payment/subscription event schema.
- Webhook replay and idempotency tests.
- Provider capability registry.
- Integration conformance tests.

Commercial opportunities may include managed routing, monitoring, operations, support, and enterprise integrations.

## 3.8 Static Product Demo and Tutorial Generator

**Existing proposal:** [link_to_tutorial.md](https://github.com/imxade/project_ideas/blob/master/ideas/link_to_tutorial.md)

Take an authorized website or project, explore meaningful journeys, generate a backend-independent interactive demo, and optionally record a guided tutorial.

Workflow:
1. Accept authorized URL or project.
2. Explore pages and controls within a defined budget.
3. Build semantic graph of product states/transitions.
4. Identify backend-dependent actions.
5. Replace them with deterministic mock behavior.
6. Generate safe synthetic data.
7. Generate guided workflows.
8. Produce self-contained demo.
9. Record with subtitles and hints.
10. Export semantic metadata for narration, editing, and localization.

Open-source opportunities include workflow format, deterministic mock runtime, browser exploration recorder, flow replay engine, and tutorial metadata. Only analyze authorized sites. Never copy private customer data or publish captured content without permission.

## 4. Cross-Portfolio Opportunities

| Shared component | Potential consumers |
|---|---|
| Permissioned agent tools | Front Desk, job assistant, VM agent |
| Browser automation/workflow recorder | Job board, demo generator, VM testing |
| Durable workflow engine | Front Desk, job reminders, MoR events |
| Connector/adapter SDK | Front Desk, MoR, job board imports |
| Artifact/provenance store | Model Builder, VM images, product demos |
| Isolated execution environment | VM product, model evaluation, browser automation |
| Evaluation/benchmark framework | Model Builder, agents, VM performance |
| Identity and tenant framework | Front Desk, job board, hosted VM |

Do not create a monolithic internal platform first. Build the smallest useful library for a real repeated need.

## 5. GSoC Strategy

### 5.1 Appropriate work

GSoC can support public work in an existing open-source organization, such as:
- Improving an established codebase.
- Building a missing adapter or integration.
- Implementing a protocol/API.
- Tests, benchmarks, and documentation.
- Portability and compatibility.
- A bounded prototype useful to upstream users.

It is not appropriate to frame private SaaS features, customer acquisition, or proprietary product work as GSoC deliverables.

### 5.2 Organization-first process

1. Identify established organizations whose codebase and community match the idea.
2. Read contributor guides, issues, project ideas, and current GSoC status.
3. Join discussions and make small contributions first.
4. Ask maintainers whether the work is wanted and who could mentor it.
5. Write the proposal in the organization's format.
6. Scope milestones to the official program period.
7. Upstream work under the accepted license and contribution process.

Do not assume QEMU, v86, WebVM, or another project participates in a particular cycle. Confirm the current official organization list and rules.

### 5.3 Candidate GSoC catalogue

| Candidate | Potential upstream research | Deliverable | Difficulty |
|---|---|---|---|
| QMP session adapter | QEMU/VM tooling ecosystem | Lifecycle adapter and contract tests | Medium |
| Browser display/input | WebVM/v86 or browser VM community | Robust display/input integration | Medium |
| Browser-local storage | WebVM/v86 ecosystem | Persistent disk prototype | Medium-high |
| QEMU WebAssembly feasibility | QEMU/porting community if accepted | Narrow reproducible prototype and blocker report | High |
| VM capability schema | VM tooling ecosystem | Backend-neutral schema and tests | Medium |
| Agent permissioned tool API | Active agent framework | Permission policy and audit layer | Medium |
| Front Desk channel adapter | Messaging/bot ecosystem | Reusable adapter and test harness | Medium |
| Durable escalation workflow | Agent/workflow ecosystem | Persisted questions and replay tests | Medium |
| MoR adapter conformance kit | Payment/open-source ecosystem | Canonical events and provider contract tests | Medium |
| Model Pack validator | ML packaging ecosystem | Versioned package spec and validator | Medium |
| Deterministic demo runtime | Browser automation/testing ecosystem | Mock runtime and replay | Medium-high |
| Job application interchange schema | Career/education ecosystem | Public schema and import/export | Medium |

These are research directions, not confirmed GSoC projects or endorsements.

### 5.4 Proposal requirements

Every proposal should include:
- Problem and motivation.
- Existing project analysis and references.
- Why the work belongs upstream.
- Scope and non-goals.
- Design and architecture.
- Deliverables and milestones.
- Weekly/phase timeline.
- Validation and tests.
- Risks and fallback plan.
- Relevant contributions and skills.
- Communication and availability.
- Mentor/maintainer feedback.

A broad startup vision is not a substitute for a testable contribution plan.

## 6. Prior-Art Research Checklist

For each project, create a dated comparison table containing source links, verified capabilities, maintenance status, license, limitations, gaps, and design implications.

### VM/browser execution
- [QEMU](https://github.com/qemu/qemu)
- [WebVM](https://github.com/leaningtech/webvm)
- [v86](https://github.com/copy/v86)
- [qemu-wasm](https://github.com/ktock/qemu-wasm)
- [Firecracker](https://github.com/firecracker-microvm/firecracker)
- [Blink](https://github.com/jart/blink)

Investigate actual supported targets and demonstrations rather than inferring capability from repository names.

### Other categories
- **Front Desk:** agent frameworks, messaging integrations, scheduling, support platforms, workflow engines. Compare authorization, persistence, human handoff, tenant isolation, and deterministic actions.
- **Job board:** ATS platforms, aggregators, candidate CRMs, career platforms, application assistants. Identify whether differentiation is discovery, application completion, or tracking.
- **Model Builder:** AutoML, training orchestration, experiment tracking, model registries, evaluation, packaging. Compare task coverage and reproducibility.
- **MoR:** payment orchestration, billing abstractions, provider SDKs, subscription migration, webhook normalization. Distinguish gateway orchestration from MoR responsibilities.
- **Demo generator:** browser recording, synthetic backends, test replay, product tours, documentation. Compare semantic workflow understanding with screen recording.

## 7. Prioritization

This is a starting hypothesis, not a final ranking.

| Project | Validation route | Open-source potential | Technical risk | Next step |
|---|---|---|---|---|
| Front Desk Agent | Pilot with one provider | High | Medium | Validate one vertical/channel |
| Application job board | Candidate interviews and prototype | Medium | Medium | Test tracking before marketplace breadth |
| Browser VM | Reproduce prior art and native QEMU | High | High | Feasibility audit and native baseline |
| Model Builder | Narrow task prototype | High | High | Select one task family and benchmark |
| MoR orchestrator | Interview SaaS teams using multiple MoRs | High | Medium-high | Validate migration/webhook needs |
| Demo generator | Test on authorized apps | High | High | Build deterministic mock runtime |
| Solly | Clarify user/problem | Unknown | Unknown | One-page concept |
| Crashkaro | Clarify user/problem | Unknown | Unknown | One-page concept |

Choose one primary venture for near-term customer validation, one open-source technical project for community contribution, and at most one adjacent experiment. Avoid committing to all at once.

## 8. Validation Framework

### Gate 1 — Problem validation
- Interview 8–12 target users where practical.
- Record workaround, frequency, and cost of the problem.
- Identify user and buyer.
- Separate evidence from assumptions.

### Gate 2 — Workflow validation
- Map the end-to-end journey.
- Prototype the smallest useful workflow.
- Test before building broad feature sets.
- Measure completion, time, failure points, and return usage.

### Gate 3 — Technical feasibility
- Reproduce prior art.
- Build the riskiest technical slice first.
- Define performance and correctness benchmarks.
- Document limitations and fallbacks.

### Gate 4 — MVP
- Deliver one complete vertical workflow.
- Add basic authentication, permissions, logging, and recovery.
- Test with a controlled group.

### Gate 5 — Adoption and commercial validation
- Measure activation, repeat usage, retention, and task success.
- Test pricing with actual buyers.
- Track operating cost and support burden.
- Confirm the open-source/commercial boundary is sustainable.

## 9. Recommended Working Order

1. Clarify Solly and Crashkaro.
2. Select the near-term business priority based on user access and validation speed.
3. Create a prior-art matrix for the VM interface and one other selected project.
4. Reproduce v86/WebVM and establish a native QEMU baseline before attempting a WebAssembly port.
5. Identify actual GSoC organizations and mentors before writing GSoC proposals.
6. Make a small upstream contribution to establish credibility.
7. Prepare organization-specific proposals with bounded milestones.
8. Update this portfolio as evidence changes.

## 10. Decisions for the Next Iteration

- What exactly are Solly and Crashkaro?
- Which venture is the near-term business priority?
- Is the VM product browser-local, remote-VM UI, or hybrid?
- Which guest OS should be supported first?
- Should the job board focus on candidates, employers, or placement organizations?
- Which open-source communities are we prepared to contribute to?
- Which GSoC cycle is being targeted?

## 11. Closing Position

The portfolio should create a reinforcing loop. Ventures expose real user problems. Open-source projects solve reusable technical problems. GSoC, where accepted by an appropriate organization, helps contributors build public infrastructure. The startup integrates that infrastructure into focused products and provides hosting, support, and operational value. Real usage reveals further problems worth solving in the open.

> **Build useful open-source foundations. Turn validated workflows into focused products. Let adoption, not assumptions, decide what becomes a business.**

## Appendix A — Existing portfolio documents

- [AI Front Desk Agent](https://github.com/imxade/project_ideas/blob/master/ideas/Frontdesk_Agent.md)
- [Automated Model Builder](https://github.com/imxade/project_ideas/blob/master/ideas/model_builder.md)
- [Merchant of Record Orchestrator](https://github.com/imxade/project_ideas/blob/master/ideas/MoR_orchestrator.md)
- [Static Product Demo and Tutorial Generator](https://github.com/imxade/project_ideas/blob/master/ideas/link_to_tutorial.md)

This proposal is additive. Existing Markdown files are not modified.
