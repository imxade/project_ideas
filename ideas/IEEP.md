# Butler

## Product Idea

Butler is a **local-first, provider-neutral AI work environment** built around one persistent primary agent interface.

The user selects one **primary AI agent/provider** that acts as the main conversational interface, understands the user's goal, keeps task context, decides what kind of work is required, delegates specialized work when appropriate, combines the results, and presents the final outcome back to the user.

The primary agent does not need to perform every capability itself.

The user can configure any number of additional AI agents, tools, or compatible endpoints for specific kinds of work.

For example:

- A general reasoning agent.
- A coding agent.
- An image-generation agent.
- An image-editing agent.
- A browser/research agent.
- A video-generation agent.
- A video-understanding agent.
- A speech or transcription agent.
- A document agent.
- A spreadsheet agent.
- A presentation agent.
- A local model.
- A private organizational model.
- A specialized domain agent.
- Any other compatible AI endpoint with a clearly described purpose and capabilities.

Each configured agent may conceptually have:

- A name.
- An endpoint.
- User-provided credentials.
- A model or model selection.
- A description of what it is good for.
- Context or instructions describing when it should be used.
- Declared capabilities.
- Optional constraints or preferences.

The primary agent uses this information to determine which available capability should handle each part of the user's request.

The user interacts mainly with **one interface**.

The specialized agents remain behind that interface unless their output needs to be surfaced directly.

Butler is **BYOK-only initially**.

The user provides the AI providers, endpoints, accounts, and keys they want to use.

There is no platform-managed AI access in the initial product.

---

# Core Principle

The core idea is:

> **One primary agent understands the user. Specialized agents and tools perform the work they are best suited for.**

The primary agent is the persistent interface.

It should be able to:

- Understand the user's complete request.
- Decide whether it can answer directly.
- Break a larger request into smaller tasks.
- Determine which capabilities are required.
- Select from the agents and tools the user has configured.
- Delegate independent work in parallel where useful.
- Continue work sequentially when one result depends on another.
- Ask the user when an important choice cannot be made safely.
- Combine specialist outputs into one coherent result.
- Keep track of the task across multiple interactions.
- Present progress without exposing hidden chain-of-thought.
- Surface generated files and other artifacts separately from conversational text.
- Report failures accurately instead of inventing successful outcomes.

For a simple request such as:

> Explain what this function does.

the primary agent may answer directly.

For a request such as:

> Research current landing-page patterns, redesign this page, generate three supporting illustrations, update the code, run the application, inspect the result, and prepare a short launch presentation.

the primary agent may coordinate several capabilities:

```text
User request
     ↓
Primary Agent
     ↓
Understand complete goal
     ↓
Break into required capabilities
     │
     ├── Research / Browser Agent
     ├── Coding / Workspace Agent
     ├── Image Generation Agent
     ├── Browser Validation
     └── Presentation Agent
     ↓
Collect results
     ↓
Validate / reconcile
     ↓
Primary Agent
     ↓
Final explanation + generated artifacts
```

The user should not have to manually switch between five separate AI products for one coherent task.

---

# Primary Agent

The user selects the primary provider/model.

This is the model the user normally talks to.

It is responsible for the overall interaction.

Its responsibilities include:

- Understanding intent.
- Maintaining task context.
- Clarifying ambiguous goals.
- Planning work.
- Selecting appropriate capabilities.
- Delegating work.
- Following dependencies between subtasks.
- Combining results.
- Deciding when another specialist is unnecessary.
- Recognizing when deterministic tools are more appropriate than another model.
- Tracking unresolved work.
- Explaining the final outcome.
- Showing relevant progress and state.

The primary agent should not be tied to one vendor.

The user may replace it with another compatible provider or local model.

Changing the primary agent should not require rebuilding the user's workspace or reconfiguring every specialist capability.

---

# Specialist Agents

A specialist agent is an AI endpoint configured for a particular purpose.

The user may configure as many as they need.

A specialist may be broad:

> Strong coding model.

or narrow:

> Use this endpoint only for image generation.

or:

> Use this browser-focused agent for web research and interaction.

or:

> Use this model for long-context repository analysis.

or:

> Use this private endpoint for company documents.

A specialist configuration should make it possible to communicate:

- What it can do.
- What it should be used for.
- What it should not be used for.
- What kind of input it accepts.
- What kind of output it produces.
- Any context useful to the primary agent when deciding whether to call it.

The primary agent should select specialists according to the task and their declared capabilities rather than according to a hard-coded vendor list.

---

# Capability-Based Delegation

Butler should reason about **capabilities**, not brand names.

Relevant capability classes may include:

## Text and Reasoning

- General conversation.
- Question answering.
- Planning.
- Analysis.
- Summarization.
- Long-context reasoning.
- Structured extraction.
- Classification.
- Translation.
- Writing.
- Editing.
- Review.

## Software Development

- Repository understanding.
- Code generation.
- Code editing.
- Refactoring.
- Debugging.
- Test generation.
- Test execution.
- Linting.
- Type checking.
- Builds.
- Dependency work.
- Git operations.
- Pull-request work.
- Issue implementation.
- Code review.
- Documentation updates.
- Terminal use.
- Running development servers.
- Inspecting application behavior.
- Working across multiple repositories.

The initial native product should target the complete practical coding loop users expect from modern coding-agent applications:

```text
Understand task
     ↓
Inspect repository
     ↓
Search/read files
     ↓
Edit project
     ↓
Run commands
     ↓
Observe real failures/results
     ↓
Iterate
     ↓
Review diff
     ↓
Validate
     ↓
Report verified result
```

## Web Research and Browser Work

- Search the web.
- Open and inspect pages.
- Gather current information.
- Compare sources.
- Follow links.
- Download relevant resources.
- Extract structured information.
- Interact with supported web applications.
- Perform browser-based research tasks.
- Validate a web application visually or functionally.
- Complete longer browser workflows where the user permits it.

A browser-focused specialist may perform this work while the primary agent remains the user's interface.

## Image Work

- Understand images.
- Generate images.
- Edit images.
- Remove or replace objects.
- Create illustrations.
- Create diagrams.
- Produce visual variants.
- Generate thumbnails.
- Create design assets.
- Analyze screenshots.
- Compare visual output.

The primary agent should be able to decide that a request requiring visual generation should be delegated to an image-capable agent rather than trying to answer with text alone.

## Audio and Speech

- Transcription.
- Speech understanding.
- Text-to-speech.
- Voice generation where supported.
- Audio summarization.
- Audio cleanup workflows.
- Timestamped analysis.
- Speech/script alignment.
- Audio-content extraction.

## Video

- Video understanding.
- Video summarization.
- Scene analysis.
- Clip selection.
- Video generation.
- Video editing.
- Caption generation.
- Thumbnail coordination.
- Script generation.
- Voiceover coordination.
- Multi-step production workflows.

A video task may itself require several specialists rather than one endpoint.

For example:

```text
Input video
     ↓
Primary Agent
     ├── Transcription Agent
     ├── Video Understanding Agent
     ├── Image Generation Agent
     ├── Writing Agent
     └── Speech Agent
     ↓
Editing / deterministic processing
     ↓
Final video package
```

## Documents

- Read documents.
- Create documents.
- Rewrite documents.
- Summarize documents.
- Extract structured information.
- Compare documents.
- Generate reports.
- Work with PDFs.
- Prepare polished deliverables.

## Spreadsheets and Data

- Read spreadsheet data.
- Analyze tables.
- Clean data.
- Generate formulas.
- Transform datasets.
- Create or update spreadsheets.
- Produce charts.
- Explain calculations.
- Build structured reports.

## Presentations

- Create presentations.
- Rewrite presentations.
- Build slide narratives.
- Produce speaker notes.
- Convert research or documents into presentations.
- Coordinate supporting visuals.
- Update an existing deck.

## Files and Local Work

- Read files.
- Write files.
- Search files.
- Organize files.
- Work across folders.
- Create archives.
- Transform formats.
- Process local data.

## Terminal and System Tools

- Run commands.
- Use development tools.
- Use language runtimes.
- Execute scripts.
- Start and stop processes.
- Inspect command output.
- Run validation.
- Use project-specific tools.

## Git and Source Control

- Inspect repository status.
- Read history.
- Understand diffs.
- Create coherent changes.
- Work with branches.
- Resolve conflicts.
- Prepare commits.
- Work from issues.
- Review pull requests.
- Address review feedback.

## Continuous Visual Context and Computer Use

Where the user enables it, Butler should be able to maintain **continuous visual awareness of the host computer** and use the computer directly.

This is broader than taking an occasional screenshot.

The user may allow the primary agent to continuously observe the selected screen, display, window, or active desktop context while Butler is running.

This allows the agent to understand what is happening in applications that do not expose a dedicated integration.

When separately permitted, the agent should also be able to act through the ordinary computer interface:

- Move the pointer.
- Click.
- Scroll.
- Type.
- Use keyboard shortcuts.
- Open applications.
- Switch applications.
- Use menus and dialogs.
- Navigate a browser.
- Fill forms.
- Upload and download files.
- Copy and paste between applications.
- Read visible application state.
- Wait for an application to change.
- Continue a multi-step workflow across several applications.

The intended product behavior is:

> **If a normal user can accomplish a task through the host computer's visible interface, Butler should be able to attempt the same task within the permissions and limits the user has granted.**

Examples include:

- Open a browser, research companies, identify suitable roles, and help apply to matching positions.
- Use a webmail interface to draft or send user-authorized messages.
- Use a cloud-drive website to find, download, upload, organize, or edit files.
- Open a design or office application and work with the visible document.
- Move information between a browser, local files, terminal, and another application.
- Reproduce a desktop or web application problem and interact with the UI while investigating it.

Computer use should not require a dedicated plugin merely because the target service has a web or desktop interface that the user can already access.

A dedicated service integration may still be useful when the user wants faster structured access, stronger reliability, or a capability that cannot reasonably be achieved through the visible UI. It is an optional capability rather than a requirement for ordinary computer use.

### Persistent Observation

The user should be able to enable a persistent observation mode for an active Butler session or background task.

While enabled:

- Visual context can remain available continuously.
- The UI should clearly indicate that observation is active.
- The user can pause or stop it immediately.
- The user can restrict which displays/windows/applications are in scope.
- The agent can use changing visual state as context for a longer task.

Persistent observation must never become hidden surveillance.

### Host Authority

Computer use remains bounded by the authority available to Butler and by the user's explicit permissions.

The agent should not be able to use visual context to grant itself additional authority.

Authentication challenges, operating-system permission prompts, CAPTCHAs, MFA, legal attestations, purchases, destructive operations, or other sensitive checkpoints may require explicit user participation depending on the task and policy.

A task may be highly autonomous while still respecting those boundaries.


## External Services and Connectors

Where the user explicitly connects them, the product may eventually coordinate work involving:

- Email.
- Calendar.
- Messaging.
- Cloud files.
- Source repositories.
- Issue trackers.
- Databases.
- Business tools.
- Notifications.
- Other user-approved services.

The primary agent should be able to combine connected services with AI specialists and the local workspace in one task.

---

# Complete Task Delegation

A user request may require one capability or many.

The primary agent should determine the smallest useful execution plan.

Example:

> Find the latest information about this library, update our project to the recommended API, run the tests, and tell me what changed.

Conceptually:

```text
Primary Agent
     │
     ├── Browser/Research capability
     │       ↓
     │   Current information
     │
     └── Coding/Workspace capability
             ↓
         Project changes
             ↓
         Test execution
             ↓
         Actual result
     │
     ▼
Primary Agent
     ↓
Final explanation
```

Another example:

> Turn this research folder into a five-slide presentation with diagrams.

```text
Primary Agent
     │
     ├── Document/File analysis
     ├── Writing/Reasoning
     ├── Image/Diagram generation
     └── Presentation capability
     ↓
Presentation artifact
     +
Summary in main conversation
```

Another example:

> Analyze this product demo recording, identify the strongest moments, generate a launch clip, thumbnail, title, and post copy.

```text
Primary Agent
     │
     ├── Video understanding
     ├── Transcription
     ├── Writing
     ├── Image generation
     ├── Video generation/editing
     └── Validation
     ↓
Final media package
     +
Main conversational summary
```

---

# Parallel Work

Independent work should be able to happen in parallel.

For example:

> Research three competing products, inspect my current project, and create three homepage directions.

Possible execution:

```text
                         Primary Agent
                              │
            ┌─────────────────┼─────────────────┐
            │                 │                 │
            ▼                 ▼                 ▼
    Research Agent A   Research Agent B    Workspace Agent
            │                 │                 │
            └─────────────────┼─────────────────┘
                              ▼
                        Primary Agent
                              │
                  ┌───────────┼───────────┐
                  ▼           ▼           ▼
             Design A     Design B     Design C
                              │
                              ▼
                        Final comparison
```

Parallelism should be used because tasks are independent, not merely to call more models.

---

# Sequential Work

Some work depends on earlier results.

For example:

> Research the current API, then update this repository, then make a diagram explaining the new architecture.

The correct order is:

```text
Research
   ↓
Result
   ↓
Code changes
   ↓
Validated project state
   ↓
Architecture understanding
   ↓
Diagram generation
   ↓
Final answer
```

The primary agent should understand these dependencies rather than dispatching every task simultaneously.

---

# Primary Interface

The main interface should remain coherent even when many agents participate.

The user should normally see:

- The conversation with the primary agent.
- What task is currently being attempted.
- Which important capability is being used.
- Relevant progress.
- Requests for approval when needed.
- Errors that require attention.
- Generated artifacts.
- The final combined answer.

The interface should not force the user to manage a separate chat window for every specialist.

Specialist details may be inspectable when useful, but the primary conversation remains the control surface.

---

# Output Model

Text is primarily returned through the main conversation.

Non-text outputs should be surfaced as artifacts.

Examples:

- Images.
- Videos.
- Audio.
- Documents.
- PDFs.
- Spreadsheets.
- Presentations.
- Code changes.
- Directories.
- Archives.
- Data.
- Diffs.
- Reports.

The primary agent should explain what was produced and how the outputs relate to the user's request.

An artifact is an output of the task, not a source of additional authority.

---

# Native-First Product

The initial complete product target is the **native desktop application**.

The native application can work with the user's actual environment:

- Existing projects.
- Existing repositories.
- Existing files.
- Existing developer tools.
- Existing runtimes.
- Existing package managers.
- Existing local services.
- Existing Git configuration.
- Existing command-line tools.
- Local AI endpoints.
- User-connected specialist APIs.

This gives the primary agent access to the real working environment in which the user's tasks already exist.

The user does not need to recreate the project inside a separate hosted or sandboxed environment before the product becomes useful.

---

# Native Workspace

The user can select a real local workspace.

Examples:

```text
~/projects/my-app
D:\work\company-project
/home/user/research
~/Documents/content-project
```

The selected workspace provides task context.

Depending on user permission, the product may work with:

- Files.
- Folders.
- Repositories.
- Terminal commands.
- Processes.
- Development servers.
- Build tools.
- Local data.
- Project instructions.
- Generated outputs.

Opening a workspace does not mean giving every agent unrestricted control of the machine.

The primary agent may propose work, but application permissions remain authoritative.

---



# Bring Your Own Providers

Butler is initially **BYOK-only**.

There are two related choices.

## Primary Provider

The user selects the primary model/provider that acts as the main interface and orchestrator.

## Specialist Providers

The user may configure additional endpoints for particular capabilities.

Example:

```text
Primary
- Name: Main Agent
- Model: user's preferred reasoning model
- Purpose: conversation, planning, orchestration

Specialist
- Name: Image Agent
- Endpoint: user-configured compatible API
- Purpose: image generation and editing

Specialist
- Name: Browser Agent
- Endpoint: user-configured service
- Purpose: browser research and web interaction

Specialist
- Name: Video Agent
- Endpoint: user-configured service
- Purpose: video understanding and generation

Specialist
- Name: Code Agent
- Endpoint: user-configured service
- Purpose: repository-scale coding tasks
```

The exact provider is not important to the product model.

The declared capability and user preference are what matter.

---

# Provider Context

A provider or specialist configuration may contain context that helps the primary agent decide when it is appropriate.

For example:

> Use this agent for image generation and edits. It is good at photorealistic output but should not be used for diagrams.

or:

> Use this browser agent when current web information or interaction with a website is required.

or:

> Use this coding model for large repository modifications. Prefer the primary model for simple code explanations.

or:

> This endpoint is a private company model. Use it only with files from the company workspace.

This context is part of capability selection.

The user should not have to repeat it in every prompt.

---

# User Preferences

The user should be able to influence routing.

Examples:

- Always use a particular model for coding.
- Prefer a local model for private documents.
- Use the primary model for simple questions.
- Use the image agent only when an image is explicitly requested.
- Prefer one browser agent over another.
- Ask before using an expensive or slow endpoint.
- Never send a particular workspace to a particular external provider.
- Use local capabilities when possible.

Explicit user preferences should override generic automatic selection where safe.

---

# Context and Memory

The primary agent needs enough context to coordinate work across time.

Relevant context may include:

- User goal.
- User constraints.
- Current task.
- Previous decisions.
- Workspace state.
- Relevant files.
- Git state.
- Recent commands.
- Validation results.
- Specialist outputs.
- Generated artifacts.
- Connected capabilities.
- User routing preferences.
- Open questions.
- Current progress.

Not every specialist needs the entire context.

Each delegated task should receive only the context needed for that task.

For example, an image-generation endpoint may need:

- Visual brief.
- Desired dimensions.
- Reference images.

It does not necessarily need:

- Complete repository history.
- Provider credentials for other agents.
- Unrelated conversations.
- Terminal output.

The primary agent coordinates context rather than blindly forwarding everything everywhere.

---

# Voice Conversation

Butler should support **voice as another way to use the same primary conversation**.

Voice mode is not a separate conversation type.

The conversation should remain readable as ordinary text:

```text
User speaks
     ↓
Speech becomes a normal user message
     ↓
Message appears in the conversation
     ↓
Primary Agent processes it normally
     ↓
Assistant response appears as normal text
     ↓
Response may also be spoken aloud
```

The user should therefore be able to move naturally between:

- Typing a message.
- Speaking a message.
- Reading the response.
- Hearing the response.
- Continuing by voice.
- Continuing by text.

A conversation may freely mix typed and spoken turns.

For example:

```text
User speaks:
"Check why the tests are failing."

Conversation shows:
You: Check why the tests are failing.

Primary Agent delegates workspace work.

Conversation shows:
Agent: Three tests are failing because...

The same response may be spoken aloud.
```

## Speak In

The user can provide a prompt by voice.

The recognized speech should appear as an ordinary editable/inspectable user message rather than disappearing into a separate voice-only session.

Before or while sending, the user should be able to understand what the system recognized.

Voice input should work for the same kinds of requests as text input, including:

- Questions.
- Coding tasks.
- Research requests.
- Creative work.
- Follow-up instructions.
- Corrections.
- Interruptions.
- Provider or capability preferences.
- Approval or rejection of actions.

## Speak Out

Assistant responses should always remain available as normal text.

The user may additionally enable spoken playback.

Speak-out should be useful for:

- Hands-free use.
- Listening while working in another application.
- Long explanations.
- Accessibility.
- Conversational back-and-forth.
- Monitoring longer-running work without constantly reading the screen.

Text remains the canonical visible conversation even when audio playback is enabled.

## Independent Controls

Speech input and speech output should be independently controllable.

A user may choose:

```text
Text in  + Text out
Voice in + Text out
Text in  + Text out + spoken playback
Voice in + Text out + spoken playback
```

The user should not be forced into full duplex voice merely because they used the microphone once.

## Same Primary Agent

Voice does not bypass the primary-agent model.

A spoken request enters the same orchestration flow as a typed request:

```text
Voice / Text
     ↓
Primary Agent
     ↓
Capability decision
     ├── Answer directly
     ├── Delegate to specialist
     ├── Use workspace
     └── Use deterministic tool
     ↓
Result
     ↓
Text response
     +
Optional speech playback
```

If the spoken request needs image generation, browser research, coding, video work, document creation, or another specialist capability, the primary agent delegates it exactly as it would for a typed request.

## Interruptions

Voice interaction should eventually allow natural interruption.

Examples:

> Stop.

> Don't change that file.

> Use the other image agent.

> Pause the current task.

> Explain that part again.

An interruption should become part of the visible conversation or task state where appropriate.

Stopping spoken playback is different from stopping the underlying task. The interface should make that distinction clear.

## Voice Outputs From Specialists

Specialist agents may produce audio artifacts such as generated speech, music, or processed audio.

Those artifacts are different from ordinary **speak-out** of the primary agent's textual response.

The user should be able to distinguish:

- Conversation speech playback.
- Generated audio artifact.
- Source audio supplied to a task.

## Conversation Continuity

Voice and text share:

- The same conversation history.
- The same task state.
- The same primary agent.
- The same configured specialists.
- The same workspace.
- The same permissions.
- The same artifacts.
- The same context.

Switching between voice and text should not start a new session or lose context.

---

# Prompt Queue and Steering

New prompts received while the primary agent is already working should have two possible meanings.

## Queue

**Queue is the default behavior.**

A new prompt is added behind the currently active work.

The current task continues without having its objective unexpectedly changed.

Example:

```text
Active:
Fix the authentication bug.

User sends:
After that, update the README.

Result:
Authentication task continues.
README request waits in the queue.
```

Queued prompts should be visible and manageable.

The user should be able to:

- Reorder them.
- Remove them.
- Edit them.
- Send one immediately.
- Convert one into a steering instruction.
- Assign one to another task when appropriate.

## Steer

A steering prompt modifies the work that is currently in progress.

Example:

```text
Active:
Redesign the settings page.

User steers:
Keep the layout, but do not change the color palette.
```

The new instruction becomes a constraint on the active task.

## Default Behavior

The global default should be configurable in settings.

Initial default:

> **Queue new prompts while work is running.**

Users who prefer a more conversational continuously-directed agent may change the default to:

> **Steer the active task.**

The user should also be able to override the default for an individual message.

Queueing and steering apply equally to typed and spoken prompts.

---

# Persistent Tasks

Butler should behave like a continuing work environment rather than a disposable chatbot.

A task may remain active over many interactions.

The product should be able to represent:

- Objective.
- Plan.
- Subtasks.
- Delegated work.
- Completed work.
- Failed work.
- Pending approvals.
- Running processes.
- Generated artifacts.
- Validation status.
- Open questions.

The user can interrupt or redirect the task.

For example:

> Stop using the image agent. Use only the assets already in the project.

or:

> Keep the code change, but regenerate the presentation using the other presentation provider.

or:

> Do not continue with deployment. Just prepare the files.

The primary agent should incorporate the new constraint without requiring the user to start over.

---

# Multiple Tasks

A workspace may contain several independent tasks.

Example:

```text
Workspace: product-launch

Task 1
Fix website issue

Task 2
Research competitors

Task 3
Generate launch images

Task 4
Prepare announcement video

Task 5
Create investor presentation
```

Each task should preserve its own state while still being able to use the same configured providers and capabilities.

---

# Coding and Current Coding-Agent Use Cases

Butler should cover the practical capabilities users expect from modern coding-agent products, while not being limited to coding.

Relevant coding workflows include:

- Understand an unfamiliar repository.
- Explain architecture.
- Search code.
- Read many files.
- Modify many files.
- Implement features.
- Fix bugs.
- Reproduce failures.
- Run commands.
- Run tests.
- Run lint/type checks.
- Build projects.
- Start development servers.
- Inspect logs.
- Work with Git.
- Review diffs.
- Prepare commits.
- Work from issues.
- Review pull requests.
- Address review comments.
- Research documentation.
- Upgrade libraries.
- Create migrations.
- Write tests.
- Generate documentation.
- Refactor.
- Diagnose performance problems.
- Compare implementation approaches.
- Work across repositories.
- Work on long-running multi-step engineering tasks.

The native workspace is initially the preferred environment for these tasks because it can use the user's real project and real tools.

---

# Modern Coding-Agent Baseline

Butler should meet or exceed the practical workflow expectations users now have from modern coding-agent applications.

That baseline includes more than editing files.

## Parallel Agent Work

The user should be able to delegate several independent engineering tasks at the same time.

Examples:

- Fix one bug while another agent researches a dependency upgrade.
- Ask several agents to explore different implementation approaches.
- Run one task on frontend work and another on backend work.
- Have a reviewer inspect changes while another task continues.
- Compare multiple candidate solutions before choosing one.

Parallel work should preserve separation between tasks so one agent does not accidentally overwrite another agent's work.

The primary agent remains the place where the user understands and supervises the overall state.

## Isolated Variants of the Same Repository

Several agents may need to work on the same project independently.

The product should support the concept of isolated working copies or branches so agents can:

- Explore alternative implementations.
- Attempt risky changes without disturbing the main working state.
- Compare competing fixes.
- Work on several issues simultaneously.
- Allow the user to inspect one result before adopting it.

The product idea is the isolation behavior, not any particular source-control mechanism.

## Change Review

The user should be able to review agent work as part of the same experience.

Relevant use cases include:

- Inspect changed files.
- Inspect diffs.
- Comment on a proposed change.
- Ask the agent to revise a specific portion.
- Reject part of a change.
- Keep part of a change.
- Compare before/after behavior.
- Open the affected project in another preferred editor when desired.
- Ask a separate reviewer agent to inspect the work.

Review is part of the task lifecycle rather than a separate product.

## Multiple Terminals and Processes

Real projects frequently need more than one running command.

Examples:

- Development server.
- Test watcher.
- Database process.
- Build process.
- Logs.
- One-off commands.

The workspace should be able to represent several active processes and let the user understand which task owns them.

## Remote Development Environments

The user's useful workspace may not always be on the same physical machine.

Later, Butler should be able to treat a user-approved remote development environment as another workspace type.

Examples:

- Remote server.
- Development box.
- Private machine.
- Cloud development environment.
- Organization-controlled host.

The primary product model remains the same:

```text
User
  ↓
Primary Agent
  ↓
Selected Workspace
  ↓
Files + Tools + Processes
```

The workspace location may be local, remote, or browser-local.

## Reusable Skills and Capability Profiles

Users should be able to define reusable ways of working.

A reusable capability profile may describe:

- When to use a specialist agent.
- Preferred workflow for a particular task.
- Project or organization conventions.
- Supporting reference material.
- Useful tools.
- Expected output.
- Validation criteria.

Examples:

- "Frontend UI review."
- "Release readiness review."
- "Security audit."
- "Create product screenshots."
- "Prepare weekly engineering report."
- "Investigate CI failure."
- "Generate launch media package."

The primary agent can select these reusable capabilities automatically when appropriate, while the user may also request one explicitly.

## Background and Repeatable Work

Some work should not require the user to manually start the same process every time.

Later use cases include:

- Daily issue triage.
- Periodic CI review.
- Dependency monitoring.
- Scheduled reports.
- Release summaries.
- Recurring repository checks.
- Repeated research.
- Watching for a condition and reporting when it changes.

Completed work should return to a review/inbox surface rather than disappearing into background execution.

## In-App Browser Work

The agent workspace should be able to include browser-based work when needed.

Examples:

- Research documentation.
- Inspect a deployed site.
- Test a frontend.
- Compare visual output.
- Work through browser-based developer tools.
- Validate a user flow.
- Interact with a web service the user has authorized.

Browser work is one capability available to the primary agent, not a separate product.

## Computer and Application Use

Where explicitly enabled, Butler should eventually be able to coordinate tasks involving applications outside the coding workspace.

Examples:

- Inspect an application visually.
- Use a design tool.
- Work with a document application.
- Transfer information between authorized applications.
- Use a browser and local files together.
- Complete a multi-application workflow.

This is broader than screen understanding.

Observation, input, and higher-impact actions remain separately permissioned.

## Rich File Review

The workspace should make non-code files understandable without forcing the user to leave the task.

Relevant file classes include:

- PDFs.
- Documents.
- Spreadsheets.
- Presentations.
- Images.
- Audio.
- Video.
- Data files.

The primary agent should be able to inspect these through the relevant configured capability and include them naturally in a larger task.

## Progress, Sources, and Artifacts

For longer work, the user should be able to inspect:

- Current plan.
- Completed steps.
- Active delegated tasks.
- Sources used for research.
- Changed files.
- Generated artifacts.
- Validation results.
- Failures.
- Pending approvals.
- Remaining work.

This is a supervision surface, not hidden reasoning.

## Persistent Preferences

The product should remember user-defined working preferences where appropriate.

Examples:

- Preferred primary model.
- Preferred coding specialist.
- Preferred review style.
- Approval preferences.
- Common project instructions.
- Preferred output format.
- Routing preferences for sensitive/private material.

Preferences should reduce repetition without silently expanding authority.

## Continue Work Across Time

A task may continue for a long period.

The user should be able to:

- Leave a task running where the selected execution environment supports it.
- Return later.
- Review what happened.
- Redirect it.
- Continue from existing context.
- Resume a recurring workflow.
- Move between devices or surfaces where the product later supports that safely.

The product should preserve meaningful task state rather than forcing every long task into a single uninterrupted chat session.

---

# Research Use Cases

Butler should support end-to-end research workflows.

Examples:

- Investigate a technical question.
- Compare products.
- Research travel or purchasing options.
- Gather current information from multiple sources.
- Analyze documentation.
- Build a structured report.
- Download relevant material.
- Cross-check claims.
- Produce a final document or presentation.

The primary agent may combine:

- General reasoning.
- Web research.
- Document analysis.
- Spreadsheet analysis.
- Image understanding.
- Writing.
- Presentation generation.

---

# Creative Use Cases

Butler should support creative workflows that require several AI capabilities.

Examples:

- Generate an image campaign.
- Edit a set of product images.
- Turn an idea into a storyboard.
- Generate video assets.
- Build a thumbnail.
- Produce narration.
- Create captions.
- Generate a final media package.
- Turn research into a visual presentation.
- Create diagrams alongside written documentation.

The user gives one goal.

The primary agent coordinates the appropriate specialists.

---

# Office and Knowledge-Work Use Cases

Butler should be able to coordinate ordinary knowledge work.

Examples:

- Analyze a PDF and create a spreadsheet.
- Turn meeting material into a presentation.
- Compare contracts or documents.
- Generate a report from several files.
- Clean and analyze a dataset.
- Create charts.
- Rewrite a document.
- Produce a proposal.
- Organize project material.
- Convert information between document formats.
- Prepare a complete deliverable package.

The product should not force all work into plain chat text.

---

# Services Without Mandatory Plugins

Butler should not require a dedicated connector for every service the user wants to use.

If the user can already access a service through an application or website, the native computer-use capability may operate that interface directly.

Examples:

- Email through an existing webmail or desktop-mail session.
- Cloud files through a browser or installed drive application.
- Job sites through the browser.
- Source-hosting websites through the browser or local Git tools.
- Calendars through their existing web or desktop interface.
- CRM or internal business tools through their normal application UI.
- Collaboration tools through their web or desktop applications.

This enables cross-application workflows without waiting for a dedicated integration to exist.

For example:

```text
User goal
   ↓
Primary Agent
   ↓
Research companies in browser
   ↓
Open job listings
   ↓
Compare requirements with user's resume
   ↓
Prepare tailored application material
   ↓
Complete allowed application steps
   ↓
Record results
```

or:

```text
Research relevant company
   ↓
Find appropriate public recruiting contact
   ↓
Draft personalized outreach
   ↓
Use user's existing mail interface
   ↓
Send where authorized and appropriate
```

Dedicated connectors remain optional.

They may be preferable when:

- Structured data access is significantly more reliable than UI automation.
- The task needs high-volume machine-readable data.
- The user wants an API-level workflow.
- Background execution cannot depend on an interactive desktop session.
- A service's UI or terms do not permit automation.

The product should therefore support both:

**computer use through the normal interface**

and

**explicit integrations where they add real value**.


# Whole-Computer File and Media Workflows

Butler should be able to treat the user's local files, cloud files, browser sessions, and visible applications as parts of one task when the user has granted access.

A representative prompt is:

> Scan my Google Drive and my local device for images. Group photos containing the same person into separate folders. The folder names do not need to identify the person; random IDs or numbers are fine. Preserve the originals. Give me the resulting folders, or render the grouped results in the conversation so I can review them.

The user should not need to tell Butler exactly how to obtain the files.

The primary agent may decide among available approaches such as:

- Open Google Drive in the browser and inspect files there.
- Use an already authenticated Drive application.
- Use a configured structured integration if one exists.
- Download selected files temporarily for local analysis.
- Download a relevant folder or catalog when that is more practical.
- Process local files directly.
- Combine cloud and local material into one task.
- Work in batches when the collection is too large to process safely at once.

The product goal is the requested outcome rather than forcing the user to choose the access mechanism.

## Multiple Accounts

There may be multiple signed-in accounts or storage locations.

The user's instruction determines scope.

Examples:

> Use my work Google account.

> Look in the personal account currently open in the second browser profile.

> Check all Google Drive accounts that are already available.

> Search both my local Pictures folder and all currently accessible Drive accounts.

If the requested account is unambiguous, Butler may proceed with that account.

If several accounts match and the user has not specified which one to use, Butler should ask rather than guessing.

A request to use one account does not authorize searching every other available account.

## Person-Based Photo Grouping

For a photo-grouping task, the result may conceptually look like:

```text
Grouped People
├── person-001
│   ├── IMG_0012.jpg
│   ├── IMG_1844.jpg
│   └── vacation_22.png
├── person-002
│   ├── DSC_4421.jpg
│   └── IMG_7782.jpg
└── uncertain
    ├── IMG_9912.jpg
    └── crop_14.png
```

The system does not need to infer or expose a person's real identity.

Random or opaque group names are sufficient unless the user explicitly supplies names or asks to label groups.

Similarity is not certainty.

Where confidence is low, photos should be placed in an uncertain/review group or presented for confirmation rather than forced into an incorrect identity cluster.

The user may receive the result as:

- Real output folders.
- A generated archive.
- A reviewable gallery in the conversation.
- A list of proposed groups before files are copied.
- Another appropriate artifact.

## Videos and Media

The same concept can extend to videos.

For example:

> Search my local videos and Drive videos and group clips containing the same people.

A video workflow may require:

- Sampling frames.
- Detecting recurring people.
- Comparing appearances across frames.
- Handling several people in one clip.
- Associating one video with multiple relevant groups where appropriate.
- Producing reviewable previews before creating large copies.

Video processing may require significantly more compute, time, storage, and model usage than photos.

The primary agent should adapt the plan accordingly rather than pretending that all media can be processed instantly.

## Preserve Originals by Default

The system should perform the **minimum side effect necessary** to accomplish the user's request.

If the user says:

> Group these images into folders.

that does **not** imply:

> Move the original files out of their current locations.

The normal interpretation should preserve originals.

Depending on the task, the system may:

- Copy files into result folders.
- Create links/references where appropriate.
- Create a generated index/gallery.
- Produce a manifest describing the groups.

Moving, deleting, overwriting, renaming originals, changing cloud organization, or modifying source metadata requires either explicit instruction or a clearly necessary confirmation.

Examples:

> Move the originals into the grouped folders.

is different from:

> Group the photos for me.

and:

> Delete duplicates after grouping.

is a separate higher-impact request.

## Storage and Resource Pressure

The system should monitor practical constraints.

If fulfilling the request would require more local storage than is safely available, it should not blindly download or duplicate the entire catalog.

It may instead:

- Process files in batches.
- Analyze them where they already are.
- Use temporary working files.
- Produce references instead of copies.
- Ask the user to choose another destination.
- Ask whether older temporary files may be removed.
- Ask whether the user prefers a lower-storage approach.

If a meaningful user choice is required, the task should pause and ask.

Example:

> Copying these groups would require approximately 180 GB, but only 72 GB is currently available. I can instead create a gallery/index without duplicating the originals, process the files in batches to another drive, or continue after you free space. Which should I use?

The agent should not silently delete unrelated files to make space.

## Cross-Application Tasks

The same product behavior should apply beyond photos.

Examples:

> Find every invoice in my Downloads folder, email attachments, and Drive, then organize copies by financial year.

> Search my local files and cloud storage for every version of my resume and show me the latest three.

> Find all screenshots relating to this project across my computer and Drive and create a review folder.

> Locate every presentation about Project Alpha across my local machine and accessible cloud accounts, compare them, and create one consolidated deck.

> Find duplicate large videos across local storage and cloud storage, show me the duplicates, and wait for me before deleting anything.

> Collect all media from this trip from my computer and cloud accounts, group it by likely event/person/date, and give me a reviewable collection.

The primary agent may combine:

- Computer vision.
- Video understanding.
- File search.
- Browser use.
- Cloud interfaces.
- Local filesystem access.
- Deterministic hashing/metadata checks.
- Specialist models.
- Artifact generation.

The user describes the outcome.

Butler decides how to combine available capabilities to reach it.

---

# Conservative Action Semantics

When a request admits several valid interpretations, Butler should prefer the interpretation that satisfies the request with the **least irreversible change**.

General defaults:

- Read before writing.
- Copy/reference before moving.
- Preserve originals before replacing them.
- Propose before deleting.
- Avoid renaming source material unless requested.
- Avoid changing account-wide organization unless requested.
- Do not broaden from one account, folder, application, or workspace to others without authorization.
- Do not send externally merely because content was prepared.
- Do not publish merely because content was generated.
- Do not purchase merely because an item was selected.
- Do not submit a form merely because it was filled when the submission has meaningful consequences.
- Ask when storage, permissions, identity, destination, cost, or other material constraints require a user choice.

This does not mean asking unnecessary questions for every reversible action.

The system should make progress autonomously within the user's stated scope while reserving confirmation for ambiguous, destructive, sensitive, costly, or scope-expanding decisions.

The broader product objective is:

> **Anything a user can reasonably accomplish on their computer should be expressible as an Butler task, and the platform should attempt to complete it by combining the available AI capabilities, host tools, browser/application control, files, and services within the permissions and limits the user has granted.**

This is an aspirational product capability, not permission to exceed user intent, operating-system authority, service rules, or physical constraints.

---

# Scheduled, Periodic, and Background Work

Scheduling is a first-class part of Butler rather than an unrelated add-on.

The user should be able to ask for work to happen:

- Once at a specific date and time.
- After a delay.
- Every few minutes or hours.
- Daily.
- Weekly.
- On selected days.
- At a recurring calendar schedule.
- When a watched condition becomes true.
- Repeatedly until a goal is reached or a stop condition occurs.

Examples:

> Tomorrow at 9 AM, check this repository and summarize any failing CI runs.

> Every hour, research companies that match my profile and prepare relevant recruiter outreach.

> Every weekday morning, review new roles that fit my resume and prepare applications for the strongest matches.

> Every Friday, inspect this project and create a weekly engineering summary.

> Watch this page and tell me when registration opens.

> At 6 PM, open the project, run the release checks, and prepare the results for review.

A scheduled task may use the same capabilities as an interactive task, subject to its permissions:

- Primary agent reasoning.
- Specialist agents.
- Native workspace.
- Terminal.
- Browser.
- Computer control.
- Files.
- Voice or notifications.
- User-approved service access.

The same delegation model applies:

```text
Schedule / trigger
      ↓
Task wakes
      ↓
Primary Agent
      ↓
Research / workspace / specialists / computer use
      ↓
Result
      ↓
Inbox / notification / artifact / pending approval
```

Scheduled tasks should have visible history and state.

The user should be able to:

- Inspect upcoming jobs.
- Pause a job.
- Resume a job.
- Edit the schedule.
- Run it immediately.
- Skip the next run.
- Cancel it.
- See previous executions.
- See failures.
- See actions awaiting approval.

### Repeated External Communication

Recurring tasks may include sending external communications where the user has authorized that behavior.

For example, a job-search workflow may periodically research suitable companies, identify relevant public recruiting contacts, prepare personalized outreach, and send messages within the user's configured limits.

Such automation should remain bounded by:

- User-defined targeting criteria.
- User-defined sending limits.
- Applicable service rules.
- Applicable law and consent requirements.
- Anti-spam safeguards.
- Sensitive-action checkpoints where appropriate.

The goal is useful autonomous work, not indiscriminate bulk messaging.

### Background Computer Work

A native scheduled task may continue to use the host computer when the required desktop session, applications, permissions, and execution conditions are available.

If the computer is locked, offline, logged out, blocked by an authentication challenge, or otherwise unable to continue, the task should report that state rather than invent progress.

Scheduled execution does not create new permissions.

It reuses only the authority the user has explicitly granted.


# Deterministic Tools

Not every task should be delegated to another AI model.

The primary agent should also be able to choose deterministic tools.

Examples:

- File operations.
- Git.
- Terminal commands.
- Search.
- Calculations.
- Media transforms.
- Format conversion.
- Test execution.
- Build tools.
- Data processing.

The primary agent chooses between:

- Answer directly.
- Delegate to a specialist AI.
- Use a deterministic tool.
- Combine several of them.

---

# Permission and Authority

The AI system should be useful without becoming the authority over the user's machine or connected services.

Relevant capability boundaries may include:

- Read workspace files.
- Modify workspace files.
- Execute commands.
- Start background processes.
- Use the network.
- Interact with browsers.
- Use screen context.
- Use external services.
- Send messages.
- Publish or deploy.
- Delete data.
- Perform other high-impact actions.

The user should be able to choose the degree of autonomy.

Possible modes include:

### Review-Oriented

Important actions wait for user approval.

### Balanced

Routine work proceeds while sensitive actions require approval.

### Autonomous Within Scope

The system may proceed within explicitly granted boundaries but still cannot expand those boundaries by itself.

Autonomy changes approval frequency.

It does not give the model permission to grant itself new capabilities.

---

# Failure Handling

Failures must remain failures.

If a specialist fails, the primary agent should know that it failed.

If a tool fails, the task should not silently claim success.

Examples:

- Image generation failed.
- Browser task was blocked.
- Video generation timed out.
- File write failed.
- Command failed.
- Tests failed.
- Provider rejected the request.
- Required capability is not configured.
- Workspace is unavailable.

The primary agent may:

- Retry when appropriate.
- Choose another compatible configured capability where user policy permits.
- Ask the user.
- Continue unaffected subtasks.
- Return a partial result.

It must not fabricate the missing result.

---

# No Hidden Provider Switching

The user should know which providers are configured and be able to control routing.

Automatic delegation is useful.

Invisible replacement of user intent is not.

If the user explicitly says:

> Use Agent A for this task.

the primary agent should not silently substitute Agent B.

If a provider is unavailable, the interface may offer or perform a policy-permitted alternative, but the actual provider used should remain inspectable.

---

# Artifacts

Artifacts are first-class task outputs.

Possible artifacts include:

- Code changes.
- Diffs.
- Images.
- Audio.
- Video.
- Documents.
- PDFs.
- Spreadsheets.
- Presentations.
- Data files.
- Web pages.
- Archives.
- Reports.
- Generated project folders.

The primary conversation explains the result.

The artifact contains the actual deliverable.

The product should preserve the relationship between:

- User request.
- Delegated work.
- Source material.
- Generated output.
- Validation.
- Final artifacts.

---

# Initial Product Scope

The initial product should prove the complete orchestration idea without requiring every possible specialist from day one.

The first version should support the full product model:

```text
User
   ↓
Primary Agent
   ↓
Capability decision
   ├── Answer directly
   ├── Use native workspace
   ├── Use configured specialist
   └── Use deterministic tool
   ↓
Collect results
   ↓
Primary Agent
   ↓
Final text + artifacts
```

## Primary Agent

- User-selected BYOK provider.
- Persistent conversation.
- Task context.
- Planning.
- Capability selection.
- Delegation.
- Progress.
- Interruption.
- Redirection.
- Final synthesis.

## Specialist Configuration

- Add compatible agent/provider.
- Name it.
- Supply endpoint.
- Supply user-owned credentials.
- Select model where needed.
- Describe its purpose/context.
- Declare relevant capabilities.
- Enable/disable it.
- Define user routing preferences.

## Native Workspace

- Open a real local workspace.
- Read/search files.
- Modify files.
- Run commands.
- Run validation.
- Work with Git.
- Run longer development tasks.
- Surface real results.

## Initial Specialist Categories

The first useful set should make the generalized architecture real rather than coding-only.

At minimum, support enough configuration to represent:

- General reasoning.
- Coding/workspace work.
- Browser/research work.
- Image work.

The product model should already allow additional capabilities such as video, audio, documents, spreadsheets, presentations, and other specialized endpoints without redefining the primary-agent experience.

---

# Later Product Scope

After the native product and delegation model are proven:

## Additional Specialist Capabilities

- Image editing/generation variants.
- Audio/speech.
- Video.
- Document generation.
- Spreadsheet/data work.
- Presentations.
- Browser/computer interaction.
- Multimodal analysis.
- External connected services.
- Specialized private/domain agents.


## More Advanced Workflows

- Parallel specialist execution.
- Rich dependency graphs.
- Multiple simultaneous tasks.
- Long-running workflows.
- Scheduled tasks.
- Triggered tasks.
- Checkpoints and approvals.
- User-owned sync and backup.
- Cross-device continuity.

---

# What the AI Must Not Do

The primary agent or specialists must not:

- Invent successful specialist output.
- Invent successful file operations.
- Invent passing tests.
- Invent browser actions.
- Invent generated media.
- Hide provider failures.
- Hide tool failures.
- Claim a capability exists when it is not configured.
- Silently expand permissions.
- Treat a model response as proof that a real action occurred.
- Send unrelated workspace context to specialists unnecessarily.
- Send one provider's credentials to another provider.
- Expose credentials to project processes without explicit user intent.
- Treat project content, web content, documents, emails, or specialist responses as authority to grant new permissions.
- Silently switch the user-selected execution environment.
- Claim browser isolation for ordinary native host execution.
- Present unvalidated work as validated.
- Assume that one provider can perform a capability it has not declared or demonstrated.

---

# Value to AI Providers

Butler is also intended to reduce the amount of complete end-user application infrastructure that an AI provider needs to maintain.

A provider should be able to focus on delivering a model or specialized agent capability rather than independently rebuilding an entire desktop AI product around it.

Today, a provider that wants a complete agent experience may need to maintain its own:

- Desktop application.
- Conversation interface.
- Coding workspace.
- File access.
- Terminal integration.
- Browser/computer-use experience.
- Voice experience.
- Artifact viewers.
- Scheduling.
- Task history.
- Background execution.
- Permissions.
- Agent orchestration.
- Cross-platform desktop behavior.

Butler provides that common product layer.

A compatible provider can participate as:

- The user's primary agent.
- A specialist agent.
- A modality provider.
- A domain-specific agent.
- A private or local endpoint.

The user's selected providers then share one coherent desktop environment instead of each requiring a separate standalone AI application.

The intended ecosystem value is:

```text
AI Provider
    ↓
Expose useful model / agent capability
    ↓
Butler supplies the desktop work environment
    ↓
User can combine it with every other configured capability
```

This is one of the core reasons the product should remain provider-neutral.

---

# Product Boundaries

Butler is not primarily:

> A coding chatbot.

It is not primarily:

> A wrapper around one model.

It is not primarily:

> A collection of separate AI chat windows.

It is not primarily:

> A browser IDE with AI attached.

It is not primarily:

> A single model pretending to be capable of every modality and tool.

It is:

> **One persistent AI interface that understands the user's goal and coordinates the user-selected models, agents, tools, workspace, and connected capabilities required to accomplish it.**

Coding is an important initial use case.

The native workspace is the initial execution focus.

But the product itself is broader:

**conversation + orchestration + capability selection + specialist delegation + real tools + artifacts + persistent context + user control.**

---

# Product Positioning

The product can be summarized as:

> **One AI desktop interface for the models, agents, tools, applications, and computer you choose.**

The user selects a primary agent.

The user connects specialist capabilities.

The user supplies their own credentials.

The user gives Butler a goal.

The primary agent decides what is needed.

Simple work may be handled directly.

Specialized work is delegated.

Independent work can happen in parallel.

Dependent work happens in the correct order.

Deterministic tools perform deterministic operations.

The native workspace provides access to real projects and tools.

Continuous visual context and computer use allow the agent to work through normal desktop and browser interfaces when the user permits it.

Scheduled jobs allow the same capabilities to operate later or repeatedly.

The user can speak or type.

New prompts are queued by default or can steer active work.

The final experience remains one coherent interaction rather than a collection of disconnected AI products.

For users, the long-term idea is:

**User goal + primary agent + specialist agents + host computer + tools + persistent context + scheduled work + artifacts + verified outcomes.**

For AI providers, the value is:

**Provide the model or specialist capability without needing to independently maintain the entire desktop agent application around it.**

