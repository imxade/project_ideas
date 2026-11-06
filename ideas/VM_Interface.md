# VM Interface — A General-Purpose Computer Platform for Humans and AI Agents

> **The possibilities are endless when an agent has unrestricted access to a full-fledged computer/VM.**

A general-purpose computer provisioning, control and sharing interface. A human or external AI agent describes the computer it needs, receives a running VM, and uses the real guest operating system through its display, input, files and available tools. A human and an agent can work in the same live session.

**Implementation order:** native QEMU/KVM first; QEMU-WASM in the browser second. Both use the same Computer model and client-facing API, with each runtime reporting the capabilities it actually supports.

---

# 1. Existing Projects and Prior Art

Before building another VM control plane, evaluate projects that already provide lifecycle management, remote APIs, display access, storage operations and session sharing. Reuse proven components or architecture where they reduce implementation and maintenance effort without narrowing the required QEMU capabilities.

The following review reflects the project investigation performed on 6 November 2026. Recheck repository activity, implementation details, security and licensing before adopting code.

| Project | Relevant capabilities | Use in this project |
|---|---|---|
| [vmcontrol](https://github.com/toey/vmcontrol) | Direct QEMU/KVM management, REST API, web UI, QEMU WebSocket VNC/noVNC, ISO and disk operations, networking, TPM, import/export and file transfer options | First implementation to audit and run. Check API/authentication, process isolation, QMP, file transfer, device coverage and security defaults. Verify its licensing before code reuse; a conventional root-level LICENSE file was not found in the reviewed branch |
| [qcontroller](https://github.com/q-controller/qcontroller) | Declarative QEMU management, REST/gRPC and OpenAPI, remote nodes, events, image distribution and separation of privileged execution from control services | Architecture reference for remote-agent access, host workers, durable jobs and privilege separation. The reviewed LICENSE identifies MIT; assess whether its multi-service design is justified for the initial single-host release |
| [Vexa-VM](https://github.com/ItzGlace/vexa-vm) | Rust service using libvirt/QEMU, REST API, noVNC gateway, scoped console links, jobs, metrics and audit | Reference for libvirt management and secure browser console design. The reviewed repository identifies AGPL-3.0; treat it as a reference unless a licensing review approves reuse |
| [Proxmox VE](https://www.proxmox.com/) and [API documentation](https://pve.proxmox.com/pve-docs/api-viewer/index.html) | Complete virtualization platform with lifecycle, storage, network, authentication, task and browser-console APIs | Benchmark for capabilities already solved by an integrated VM platform. Compare its deployment and host model against a smaller QEMU/libvirt service |
| [libvirt](https://libvirt.org/) and its [QEMU driver](https://www.libvirt.org/drvqemu.html) | VM lifecycle, host capabilities, storage pools, virtual networks, remote connections and QEMU process management | Active candidate for the native management layer. Prototype against direct QEMU management and retain it if it reduces total work while preserving required QEMU devices and QMP operations |
| [QEMU](https://www.qemu.org/) and [QMP](https://www.qemu.org/docs/master/interop/qemu-qmp-ref.html) | Full-system emulation, virtual hardware, machine configuration and runtime control | Selected foundation for native execution and host-side virtual input |
| [noVNC](https://github.com/novnc/noVNC) | Browser-based VNC client with keyboard/mouse interaction and display handling | Selected browser viewer for native QEMU. It is a viewer, not the VM manager, authorization layer or session gateway |
| [QEMU-WASM](https://github.com/ktock/qemu-wasm) | Experimental QEMU adapted for WebAssembly/browser execution | Selected for a later browser-runtime feasibility phase. Validate boot, display, input, storage, networking, browser compatibility and performance before promising support |
| [v86](https://github.com/copy/v86) | Browser-based x86 PC emulator | Browser-emulation reference and comparison point for constrained browser-local use cases; not a replacement for the required QEMU device and architecture coverage |

Additional components to evaluate or reuse:

| Component | Intended role |
|---|---|
| [qemu-img](https://www.qemu.org/docs/master/tools/qemu-img.html) | Create, inspect, convert, resize, clone and export virtual disk images |
| [QEMU Guest Agent](https://www.qemu.org/docs/master/interop/qemu-ga.html) | Optional guest information, freeze and file/command operations when installed |
| [libguestfs](https://libguestfs.org/) | Optional offline disk and filesystem inspection or preparation |
| [Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/) | Optional ingress for the API and console gateway; not part of the VM abstraction |

The initial investigation should run vmcontrol and qcontroller, inspect Vexa-VM, compare Proxmox as a complete platform, and prototype direct QEMU versus libvirt-managed QEMU. Measure actual required workflows rather than deciding from feature lists alone.

The comparison must establish whether each candidate can create the required machine profiles and advanced devices; be operated by a remote agent through an API; provide a secure browser view of the same VM; manage installation media and disk artifacts; transfer files; inject keyboard, pointer and supported touch events; isolate privileged operations; and satisfy licensing and security requirements.

---

# 2. Product Idea and Motivation

## 2.1 The problem

Agents that need to work with real software or operating systems commonly depend on a collection of separate capabilities:

- Application-specific MCP servers and AI plugins.
- Browser-use and desktop-control tools.
- Website- or SaaS-specific API wrappers.
- File-transfer services and custom test runners.
- Separate environments for different operating systems.
- Remote desktops, cloud PCs and agent execution workspaces.
- Specialist chatbot connectors and sub-agent adapters.

Each integration has its own interface, configuration, permissions and failure modes. A new application or workflow often requires another connector or execution environment. The same fragmentation appears in dedicated browser environments, such as [BrowserOS neo](https://github.com/browseros-ai/BrowserOS), which address one part of the larger computer-access problem.

A full computer provides a common execution surface. An agent can use a browser, desktop application, terminal, local script, development tool, game launcher or chatbot through the environment in which that application already runs. It can move files between them and let a human inspect or take over the same live session.

## 2.2 The product

VM Interface lets a human, external agent or automation client:

1. Describe a computer declaratively: OS/image, architecture, CPU, memory, disks, firmware, devices, boot media and networking.
2. Validate the request against the selected host and runtime capabilities.
3. Create and boot the computer.
4. Observe its actual display and send keyboard, pointer and supported touch input.
5. Install and operate guest software using whatever tools are appropriate for that OS.
6. Upload, download and manage files, installation media and virtual disks.
7. Share the same running computer with a human through a secure browser session.
8. Preserve, snapshot, clone, export or destroy the computer according to supported lifecycle operations.

The platform provides the computer and a small control surface. The external agent or human chooses the OS setup method, browser, applications, programming language, test framework and task workflow.

## 2.3 Why it is useful

The key benefit is **breadth through a small set of reusable computer primitives**. Instead of implementing a new platform connector for every application, the agent can operate applications that are already usable inside the guest OS.

This can reduce or replace the execution and application-access layer of many browser/desktop harnesses, UI-oriented MCPs and super-agent products. In a super-agent workflow, the user can sign in to the agentic tools and chatbots they choose inside the VM. The external host agent can then use those authenticated services as task-specific workers: send instructions and reference files, move outputs between services, inspect and compare results, validate files locally and share the finished work. The host agent remains responsible for reasoning and orchestration; VM Interface supplies the common computer, files and shared session.

Direct APIs remain useful where they are more reliable or provide structured operations. VM Interface is a general computer substrate, not a reason to replace every good API with visual automation.

---

# 3. Core User Workflows

The same Computer API supports these workflows without introducing a separate platform feature for each application.

## 3.1 Provision and configure an operating system

A user asks for a Windows computer with a specified architecture, CPU/RAM/disk, UEFI firmware and TPM 2.0. The agent obtains official installation media, creates the machine, attaches the ISO, boots it, and completes setup using the GUI or an OS-native unattended answer file where supported. It installs the requested application and leaves the computer at a chosen checkpoint. The user opens the browser link to sign in or continue using the same desktop.

For example, the agent can install Windows 11 and Epic Games Launcher, then stop at the sign-in screen for the user.

## 3.2 Develop and test an Android application

The agent selects a compatible AArch64 Android/AOSP image for the chosen QEMU machine profile, creates the guest, sets up the development tools, builds an application, installs it and runs tests. It observes the UI, collects screenshots and logs, fixes failures and repeats until the acceptance criteria pass. A separate Linux build VM may be used where useful.

Android images must be selected for the virtual machine model; an arbitrary OEM phone image is not assumed to boot on generic QEMU hardware. Input, ADB, graphics and device capabilities must be tested on the exact image and profile.

## 3.3 Operate a website or chatbot using a user-authenticated account

The agent creates an isolated guest and opens the target application. The user joins the same desktop, signs in directly and completes required MFA, passkey, consent or manual approval steps. The user explicitly returns control. The agent then operates the authorized session, saves outputs and returns file references through the calling client.

## 3.4 Coordinate multiple signed-in agentic services

A user asks the host agent to prepare a researched brief and supporting spreadsheet. The user signs in to the research chatbot, spreadsheet agent and writing service inside the shared VM. After the user returns control, the host agent assigns the relevant tasks, passes context and files between services, compares their outputs, runs local validation and returns the completed artifacts in the original conversation. The user can inspect the same live sessions and take control at any point.

This can replace a super-agent product when its main value is a bundled workspace, service-specific connectors, specialist-agent adapters and artifact handoff. It does not replace the host agent's reasoning or the models and capabilities of the signed-in services.

## 3.5 Cross-platform testing and reproducible environments

The agent creates separate Linux, Windows or Android computers, installs the required browsers and test tools, runs the same test suite, collects screenshots and logs, and compares the results. The platform provisions and exposes the computers; the agent owns the build and test logic.

## 3.6 Disk preparation and file workflows

An agent uploads a disk image, inspects its format and size, converts or enlarges a working copy, attaches it to a computer, and retrieves the resulting artifact. When a guest partition or filesystem must also grow, the agent uses the guest OS tools or a supported offline tool and verifies the result.

## 3.7 Repeatable environments and parallel tasks

An agent creates a base computer, configures the guest, snapshots or clones its disk where supported, and starts multiple independent instances for parallel development or testing. Each computer has its own identity, storage and session permissions.

## 3.8 Hosting a guest application

An agent starts a service inside the guest and requests a temporary private or authorized public URL. VM Interface creates the permitted network exposure and provides the URL. The agent remains responsible for configuring and running the application.

## 3.9 Remote host utilities

An agent running on another machine requests an approved host utility job such as `qemu-img` inspection or conversion. The managed host executes it within a scoped workspace and returns structured output and artifact references. The agent does not need host SSH or filesystem access.

---

# 4. Computer Model and Declarative Configuration

## 4.1 Computer as the main resource

A computer has a stable identity and a desired machine specification. The platform owns machine provisioning and runtime state. The agent owns task-specific guest configuration.

**Machine desired state** includes:

- Architecture and QEMU machine type.
- Firmware and boot configuration.
- vCPU and memory.
- Disk images, sizes, persistence and boot media.
- Virtual devices such as keyboard, pointer, touch and optional TPM 2.0.
- Display dimensions and display adapter.
- Network mode, restrictions and service exposure.
- Runtime preference and backend-specific options.

**Guest desired state** belongs to the agent. It may describe OS users, packages, files, applications, tests and checkpoints. The agent chooses how to realize that state through GUI interaction, shell/PowerShell, unattended answer files, browser automation, SSH, WinRM, ADB or other tools.

QMP controls QEMU and virtual hardware. It does not install applications, create guest users or configure the guest OS.

## 4.2 ComputerSpec and resolved capabilities

The API accepts a versioned declarative ComputerSpec rather than requiring clients to build QEMU command lines. Keep common portable fields small and allow validated QEMU-specific device properties through an explicit extension for advanced configurations.

Example shape (illustrative, not a frozen schema):

```json
{
  "apiVersion": "vm-interface/v1",
  "name": "windows-workspace",
  "runtime": { "prefer": "native-qemu" },
  "machine": {
    "architecture": "x86_64",
    "machineType": "pc-q35",
    "firmware": "uefi",
    "vcpus": 4,
    "memoryMiB": 8192
  },
  "devices": {
    "tpm": { "mode": "emulated", "version": "2.0" },
    "input": {
      "keyboard": "virtual",
      "pointer": "absolute",
      "touch": "virtio-multitouch"
    }
  },
  "disks": [
    { "id": "system", "sizeGiB": 120, "persistence": "managed" }
  ],
  "boot": [
    { "type": "iso", "image": "windows-install-media" }
  ],
  "display": { "width": 1280, "height": 800 },
  "network": { "mode": "restricted-internet" }
}
```

Before provisioning, validate the spec against the actual host/runtime. Return both the requested and resolved configuration, including the selected execution engine, actual machine/device models, resource limits, supported input modes, rejected or adjusted fields and reasons. Never silently replace an unsupported device or runtime requirement.

Support dry-run/plan validation so an agent can inspect resource and capability resolution before creating a computer.

## 4.3 Lifecycle and state

The API should support the following operations where the selected backend provides them:

- Create, inspect, list and configure computers.
- Start, stop, pause, resume, reset and destroy.
- Attach/eject installation and configuration media.
- Observe display state and capture screenshots.
- Send normalized input.
- Upload/download files and artifacts.
- Inspect, snapshot, clone, resize and export virtual disks where supported.
- Expose and revoke guest services.
- Share and revoke viewer/control sessions.

Distinguish persistent disk state from live RAM/device state. Do not claim that snapshots or save/restore are supported across all backends without testing them.

---

# 5. Public API and Remote-Agent Access

## 5.1 API contract

The canonical interface is a versioned REST/OpenAPI contract with machine-readable ComputerSpec and capability schemas. A remote agent must be able to operate the platform without SSH access, a local QEMU installation, access to the host filesystem or access to the QMP socket.

The initial logical API surface is:

| Operation | Responsibility |
|---|---|
| `computer.create(spec)` | Validate and create a computer |
| `computer.plan / validate` | Resolve capabilities and resources before provisioning |
| `computer.get / list` | Inspect configuration, status and supported capabilities |
| `computer.configure` | Apply supported machine configuration changes |
| `computer.lifecycle` | Start, stop, pause, resume, reset and destroy |
| `computer.observe` | Capture screenshot/display state or subscribe to an authorized display stream |
| `computer.input` | Send normalized keyboard, pointer, scroll and supported touch events |
| `computer.media` | Attach/eject media and configure boot order |
| `computer.files` | Import, export, upload and download files |
| `computer.storage` | Manage virtual disks and supported snapshots/clones |
| `computer.connect` | Create authenticated human/agent session connections |
| `computer.expose` | Publish authorized guest services through a controlled gateway |
| `computer.exec` | Optional guest command channel through configured SSH, WinRM, ADB or another guest mechanism |
| `artifact.register / get` | Register files and inspect immutable artifact metadata |
| `artifact.acquire` | Optional platform-side URL download, only when requested |
| `host.jobs` | Run approved host utilities remotely and retrieve structured results/artifacts |

Operations that may take time should return job IDs and structured progress/events. Use idempotency keys for retried creation and mutation requests. Return structured error codes, capability explanations and artifact references instead of requiring an agent to parse terminal output for routine tasks.

## 5.2 Remote host jobs and advanced operations

A remote agent sometimes needs the same host-side utility it would otherwise run in a CLI, such as `qemu-img`, image inspection/conversion, checksum verification or disk preparation. Provide a constrained host-job facility for this long tail rather than adding a bespoke endpoint for every utility and flag.

A host job should accept an approved execution profile, executable and argument array, scoped working directory, artifact/file inputs, environment policy, timeout, output limits and resource limits. It returns exit status, bounded logs and output artifact references.

This provides useful remote CLI capability without giving an ordinary agent an unrestricted root shell. Jobs run in isolated workspaces with narrow filesystem and network permissions, audit records and cancellation. They must not launch competing unmanaged QEMU processes or bypass the VM lifecycle owner. Keep host jobs separate from optional guest `computer.exec`.

For advanced QEMU configuration, accept validated backend-specific options in ComputerSpec. Keep raw QMP sockets private; do not turn the public API into an unrestricted monitor proxy.

## 5.3 Developer interfaces

The REST/OpenAPI API is primary. A CLI, SDK or generated MCP adapter may be provided where it reduces integration work, but each remains a client of the same API and is not required for access. Existing agents should be able to use ordinary HTTP calls and their own tools.

---

# 6. Images, Artifacts, Storage and File Transfer

## 6.1 Artifact acquisition

Accept installation media and workspace files through equivalent paths:

1. The agent downloads an official image using its own browser, shell or tools, then uploads/registers it.
2. The user uploads a local ISO, disk image or file.
3. The client references an artifact already stored by the platform.
4. The platform optionally downloads a user-specified URL when requested.

All paths create an immutable artifact record containing ID, file name, format, size, architecture/OS hints, provenance where known, checksum/signature status and storage location. Verify publisher checksums/signatures when available and record when verification is unavailable.

Large artifacts require resumable upload/download or controlled streaming. Optional URL acquisition must defend against SSRF, private/link-local addresses, redirect abuse, DNS rebinding and oversized files. Treat uploaded images as untrusted input.

## 6.2 Disk and storage operations

Use QEMU's native image utilities for supported image operations: create, inspect, convert, resize, clone, snapshot, import and export. Preserve source artifacts and report the new artifact ID and metadata.

Virtual disk resizing is not the same as expanding a guest partition or filesystem. After enlarging an image, the guest must rescan the disk and use its own partition/filesystem tools, or supported offline tooling must be used. Report exactly which operation was performed and verify the result.

## 6.3 File transfer

Provide a consistent file-transfer API while allowing backend-specific transports:

- QEMU Guest Agent when installed and supported.
- SSH/SFTP for Linux-like guests.
- WinRM or another Windows-native channel.
- ADB for Android.
- Shared folders, virtiofs/9p, mounted ISO/data disks or normal GUI/browser upload/download.

No guest agent or guest connectivity service is mandatory for basic GUI control. Where a transfer method is unavailable, the API must say so rather than implying QMP can copy arbitrary files into any guest.

---

# 7. Native QEMU Runtime and Input/Display Architecture

## 7.1 Execution sequence

**Native QEMU is the first execution target.** Run full-system QEMU on a managed Linux host, using KVM acceleration where compatible and QEMU TCG where needed and practical. Keep the runtime behind a stable Computer API.

The native host manager owns QEMU lifecycle and a private QMP connection for each VM. Each computer has one authoritative lifecycle owner: either the direct QEMU manager or libvirt. Do not have both independently manage the same VM.

## 7.2 Direct QEMU versus libvirt

Both are candidates for the native host-management layer.

- **Direct QEMU management** gives full control over QEMU arguments, devices and QMP, but requires implementation of process supervision, host capability discovery, storage/network integration, recovery and related management.
- **Libvirt-managed QEMU** may remove lifecycle, host capability, storage and network code. Required QEMU-specific devices and QMP runtime operations must remain accessible through supported APIs. If libvirt owns a domain, the VM Interface service must not independently launch or kill its QEMU process.

Prototype the same representative machines and workflows using both. Compare total code and operational complexity, QEMU device/firmware/TPM coverage, QMP input, disk/media operations, display, recovery, isolation and maintenance. Choose the smaller reliable solution; do not support both in the MVP without a demonstrated need.

## 7.3 Host-side keyboard, pointer and touch input

The native public `computer.input` operation maps to QMP `input-send-event`. It supports keyboard events, pointer buttons, relative/absolute pointer movement and multi-touch events where the relevant QEMU build and guest-visible device support them.

Configure guest-visible virtual input devices:
- Virtual keyboard for key events.
- Mouse or absolute tablet for pointer control.
- QEMU virtio multi-touch device, such as `virtio-multitouch-pci`, when touch is required and supported by the guest.

A USB tablet or absolute pointer is not a touchscreen. Genuine multi-touch requires a compatible guest device and driver. QEMU documents the QMP multi-touch `mtt` event from QEMU 8.1; capability-check the actual QEMU build and device. Android/AOSP input must be validated on the selected image, including its virtio-input support.

An event accepted by QEMU does not prove the guest OS acted on it. Report the tested input modes and contact count for each supported guest profile.

## 7.4 Display and browser access

For native QEMU, expose the guest display through QEMU VNC over WebSockets and use noVNC as the browser viewer. QEMU supports WebSocket VNC in supported builds; websockify is conditional, not a required proxy.

A session gateway routes the authenticated browser session to the private VNC endpoint. QMP and VNC must not be exposed directly to the public internet. The gateway provides authentication, scoped access, routing, session permissions, expiry and revocation.

The browser viewer is separate from the agent input API. Human and agent input must obey the same control lease. Configure viewer sessions so direct RFB input cannot bypass authorization; route controlled human input through the common input gateway when required.

---

# 8. Shared Human/Agent Sessions and Security

## 8.1 One live computer

A human and one or more authorized agents attach to the same computer identity and live display. A session supports:
- Observe-only and control permissions.
- Scoped, revocable invitations.
- A control lease for keyboard, pointer and touch input.
- Human takeover and return of control.
- Session expiry and explicit revocation.
- Audit events for access and control changes.

After a handoff, the agent should take a fresh observation before acting. A human should be able to pause agent control and continue using the same guest without creating another VM.

## 8.2 Authenticated application sessions

When a user signs in to a website, chatbot or agentic service inside the guest, that authenticated session is deliberately delegated to the authorized VM operator. A user should understand that an agent given control of the VM may be able to use the signed-in account and see its accessible data.

During password, passkey or one-time-code entry, pause agent input and disable agent observation/recording where supported. Do not capture credentials in ordinary screenshots, logs, prompts or artifacts. Keep authenticated browser profiles and session state inside the guest's storage boundary. Respect application permissions, terms, rate limits and human-verification requirements.

## 8.3 Host and guest boundaries

Guest administrator rights must not imply host administrator rights. Run VMs and host jobs under restricted host identities. Isolate VM storage, QMP sockets, artifacts and network policy. Validate uploaded images and treat guest contents and network traffic as untrusted.

The API server must not be an unrestricted root shell. Privileged operations should run through a narrow host worker with explicit profiles and auditable inputs. Keep host credentials, provider secrets and other tenants' data outside guest disks.

A session URL alone is not sufficient authorization. Use authenticated, scoped, expiring tokens with revocation, audit and resource/network isolation.

---

# 9. Networking and Guest Connectivity

Support the network modes required by the selected runtime, beginning with a safe outbound configuration. Allow authorized clients to request a guest service URL or port mapping through a controlled gateway.

- Guest services are private by default.
- Public exposure requires explicit authorization, expiry and revocation.
- Record mappings and revoke them when a computer or exposure grant ends.
- Keep tunnel-provider credentials outside guest disks.
- Use one shared host ingress/gateway where practical rather than a separate tunnel per VM.

SSH, WinRM, ADB and QEMU Guest Agent are optional guest-side channels for commands or files. They are not substitutes for QMP-backed host-side GUI input and are not required to observe or operate a basic graphical guest.

---

# 10. Future Runtime: QEMU-WASM in the Browser

QEMU-WASM is the second execution adapter, after the native API, machine model and input path are validated.

Run the emulator in a dedicated browser worker, render the guest display in the page and translate keyboard, pointer and supported touch events through QEMU's input subsystem. Reuse QMP input events if they are present in the build; otherwise provide a narrow worker-to-WASM bridge to QEMU's existing input machinery. Do not inject guest input into the user's physical host OS.

The browser runtime should reuse the ComputerSpec and API vocabulary, but it must report its smaller capability set. In particular:
- Disk persistence may use OPFS or IndexedDB and is subject to browser quotas, eviction and origin boundaries.
- Import/export is needed for portability.
- Browser worker termination or tab closure may stop the running machine. Disk persistence does not imply RAM/device-state persistence.
- Networking, threading, memory, device support and performance must be measured on target browsers.
- Remote agent access requires an authenticated session relay when the agent is not colocated with the browser.

A browser-local VM runs on the user's physical device; it does not remove the need for hardware. Do not promise native KVM performance or native QEMU feature parity.

---

# 11. Future VPS and Cloud Providers

Keep a thin provider adapter beneath the Computer API for a later stage. A provider may offer a VPS through its own API, or a host on which VM Interface can run QEMU. These are different capabilities.

A generic VPS is not automatically equivalent to a QEMU host: arbitrary ISO boot, nested virtualization, TPM, custom devices, live snapshots and direct display/input may not be available. Each provider must report its actual capabilities. This is a future extension, not a prerequisite for the native or browser runtime.

---

# 12. Technology Direction

| Area | Current direction |
|---|---|
| Native full-system VM | QEMU |
| Native acceleration | KVM where available; TCG fallback where practical |
| Native machine/runtime control | Private QMP connection per VM |
| Host-side GUI input | QMP `input-send-event` and guest-visible QEMU input devices |
| Native browser desktop | QEMU VNC over WebSockets + noVNC |
| Public client interface | Versioned REST/OpenAPI |
| Machine configuration | Versioned ComputerSpec with capability resolution and QEMU-specific extensions |
| Disk/image operations | QEMU disk tooling such as `qemu-img` |
| Artifact handling | Upload/register/download with immutable metadata and checksums |
| Remote CLI-equivalent tasks | Restricted asynchronous host jobs |
| Optional guest operations | QGA, SSH/SFTP, WinRM, ADB or shared folders |
| Second runtime | QEMU-WASM in a browser worker |
| Browser storage candidate | OPFS/IndexedDB with explicit persistence limits |
| Public ingress | Authenticated gateway; Cloudflare Tunnel optional |

The following choices remain open until a prototype provides evidence: direct QEMU versus libvirt; service implementation language and framework; database and job/event storage; object storage; and future cloud provider. Do not add a database, queue, SDK, MCP server, tunnel or framework without a concrete requirement.

---

# 13. Validation and Acceptance Criteria

A feature is supported only for the exact host, backend, guest and device combination that has been tested.

| Dimension | Initial validation |
|---|---|
| Host | Linux native host |
| Architecture | x86_64 first; ARM64 after the initial baseline |
| Acceleration | KVM and TCG, with actual selected accelerator reported |
| Guests | Linux, Windows and a selected Android/AOSP image |
| Firmware | BIOS and UEFI |
| TPM | Emulated TPM 2.0 on supported native profiles |
| Storage | qcow2 and raw initially; create, attach, convert, resize and export |
| Input | Keyboard, relative/absolute pointer, scroll and validated touch |
| Display | QEMU VNC WebSocket and noVNC |
| Networking | Outbound guest access and controlled inbound service exposure |
| File transfer | Tested guest channel for Linux and Windows |
| Host jobs | Approved utilities, isolation, timeout and artifact return |
| Sharing | Human viewer, agent controller, control lease and revocation |
| QEMU manager | Direct QEMU/libvirt comparison on identical workflows |
| Browser runtime | QEMU-WASM boot, display, input, storage, networking and performance |

The core product is validated when:

- A remote agent with no host access can create, boot, inspect and operate a computer through the API.
- The agent can register installation media, configure hardware declaratively and receive the resolved configuration.
- The agent can observe the real display and send guest-visible input without a required guest automation agent.
- A human can open a secure URL to the same running computer and take over control.
- Authenticated application sessions can be delegated deliberately, with credentials protected and access revocable.
- The agent can upload/download files and manage supported disk artifacts.
- Host utilities can be invoked through restricted jobs without exposing host shell access.
- Multiple VMs can support parallel workflows within resource limits.
- The same Computer API can later support QEMU-WASM while reporting its distinct capability set.
- The platform reduces the need for bespoke execution wrappers where ordinary computer use is sufficient.
