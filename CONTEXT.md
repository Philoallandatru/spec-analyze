# NVMe Base 2.1 Knowledge Context

Canonical vocabulary for the concept wiki derived from NVM Express Base Specification Revision 2.1.

## Specification model

**Base Specification**:
The transport-independent controller protocol and common I/O Command Set foundation.

**Transport Binding**:
The mapping of NVMe protocol behavior and controller properties to one specific transport.

**0's-based value**:
An encoded count where `0h` represents one and zero itself cannot be represented.

## System model

**NVM Subsystem**:
The boundary containing controllers, ports, namespaces, and their underlying NVM resources.

**Controller**:
The host-facing interface to an NVM subsystem that exposes properties, processes commands, and accesses namespaces.

**Static Controller Model**:
A subsystem-wide controller allocation model in which controller identity and saved state persist across Controller Level Reset and, for Fabrics, may persist across associations.

**Dynamic Controller Model**:
A subsystem-wide controller allocation model in which controllers are allocated on demand without state preserved from prior associations.

**I/O Controller**:
A controller that accesses user data through one or more I/O Command Sets and may also provide management capabilities.

**Discovery Controller**:
A controller that returns discovery information and has neither I/O queues nor access to non-volatile storage.

**Administrative Controller**:
A management-only controller without I/O queues, namespace attachment, or user-data access.

**Asymmetric Controller Behavior**:
Namespace access characteristics that vary with subsystem configuration or the controller used to access the namespace.

**Namespace**:
A formatted quantity of NVM presented as a logical-block storage abstraction through one or more controllers.

**Namespace Identifier (NSID)**:
A controller command handle for a namespace whose allocation is subsystem-scoped and whose active/inactive status is controller-relative.
_Avoid_: Namespace identity when durable identity is intended

**Active NSID**:
An allocated NSID whose namespace is attached to the controller evaluating it.

**Allocated NSID**:
A valid NSID that refers to an existing namespace in the NVM subsystem.

**Association**:
The exclusive relationship between one host and one controller that encompasses the controller's Admin Queue and all its I/O Queues.
_Avoid_: Connection when referring to the controller-level relationship

**Candidate Command**:
A submitted command transferred into the controller and judged ready for processing.

**Command Arbitration**:
The mechanism that selects the Submission Queue from which the controller starts processing the next candidate command or fused pair.

**Arbitration Burst**:
The maximum number of commands a controller may start from a selected Submission Queue before arbitration occurs again.

**Fused Operation**:
An optional two-command operation submitted adjacently in one Submission Queue and executed in sequence as an atomic unit.

**Outstanding Command**:
A submitted command for which the host has neither received a completion nor obtained protocol-defined evidence that further controller processing is impossible.

**Common Command Format**:
The 64-byte Submission Queue Entry shared by Admin and I/O commands, containing common command identity, namespace, metadata, data-pointer, and command-specific regions.

**Completion Queue Entry (CQE)**:
The controller-produced record that correlates a completed command, reports consumed Submission Queue position, and carries command-specific results and Status.

**Completion Status**:
The CQE outcome encoding composed of retry/log hints plus a Status Code Type and Status Code.

**Phase Tag**:
A memory-based Completion Queue slot generation bit whose expected posted value toggles each time the controller's posting position wraps the queue.

**Physical Region Page (PRP) Entry**:
A 64-bit pointer describing one physical-memory page, with a command-first-entry offset permitted by the command and `CC.MPS`.

**PRP List**:
A packed, page-sized array of PRP entries that may use its last entry to chain to another page of entries.

**Scatter Gather List (SGL)**:
A sequence of typed, length-bearing descriptors organized into qword-aligned segments to describe a command data buffer.

**SGL Segment**:
A qword-aligned contiguous array of 16-byte SGL descriptors whose final descriptor may link to the next or last segment.

**Bit Bucket Descriptor**:
An SGL descriptor that consumes but does not place part of source data when describing a destination buffer.

**Feature Value**:
A controller operating parameter addressed by Feature Identifier with a current value, immutable default, and optional saved value.

**Feature Scope**:
The controller, namespace, subsystem, Domain, Endurance Group, or NVM Set boundary to which a Feature value applies.

**NVMe Qualified Name (NQN)**:
A permanent, globally unique, null-terminated UTF-8 name for a host or NVM subsystem, encoded in the domain-authority or UUID form.

**Namespace Global Identifier**:
A durable namespace identity expressed through EUI64, NGUID, or UUID, distinct from the NSID command handle.

**Controller Global Identity**:
The composite of a globally unique NVM subsystem identifier and a controller identifier unique within that subsystem.

**Protocol UTF-8 String**:
An NVMe-carried UTF-8 byte string stored and compared without Unicode normalization or locale-specific processing at receipt.

**Command Retry Delay (CRD)**:
An advisory CQE selector for immediate retry or one of three Identify Controller delay values, applicable only when retry is allowed and Advanced Command Retry is enabled.

**PRP or SGL for Data Transfer (PSDT)**:
The Command Dword 0 selector that determines whether common-command metadata and data pointers use PRP or SGL forms.

**Keep Alive Timeout Time (KATT)**:
The effective per-controller watchdog interval configured through `KATO`, subject to transport limits and controller `KAS` granularity.

**Command Based Keep Alive**:
A watchdog mode in which successful Keep Alive command processing is the controller's liveness evidence.

**Traffic Based Keep Alive**:
A watchdog mode in which fetched Admin or I/O commands count as liveness, allowing ordinary traffic to avoid explicit Keep Alive commands.

**Capsule**:
The NVMe over Fabrics exchange unit containing a command or response and optionally data and SGLs.

**In-Capsule Data Offset (ICDOFF)**:
An Identify Controller value locating command data at `64 + ICDOFF * 16` bytes from the start of a command capsule.

**Submission Queue Flow Control**:
The per-queue-pair Fabrics agreement in which `SQHD` completion values release consumed Submission Queue slots for host reuse; it may be disabled by Connect negotiation.

**Individual I/O Queue Deletion**:
A mutually supported Fabrics capability that allows Disconnect or a transport error to remove one I/O queue pair without terminating its association.

**NVM Subsystem Port**:
One NVMe over Fabrics protocol interface between an NVM subsystem and a fabric, composed of one or more physical fabric interfaces.

**Discovery Service**:
An NVM subsystem that contains only Discovery controllers and supplies hosts with accessible subsystem and path information.

**Discovery Log Page Entry**:
A host-visible record describing how to connect to an NVM subsystem or referring the host to another Discovery Service.

**Discovery Log Snapshot**:
A Discovery log retrieval that is atomic in one command or validated across multiple commands by re-reading an unchanged Generation Counter.

**Extended Discovery Log Page Entry**:
A Discovery record with the fixed connection header followed by total-length and count fields that frame zero or more extended attributes.

**Host Discovery Log**:
A Discovery-controller inventory of registered hosts, optionally filtered by the requesting Host NQN and carried in variable-length entries with Host Identifier attributes.

**AVE Discovery Log**:
A discovery-plane inventory of Authentication Verification Entities and their IPv4 or IPv6 TCP transport records.

**Identify Controller or Namespace Structure (CNS)**:
The Identify command selector that chooses the returned 4,096-byte structure and determines which namespace, controller, command-set, or other qualifiers are meaningful.

**I/O Command Set Independent Identify Controller Structure**:
The common 4,096-byte controller profile selected by `CNS=01h`, combining subsystem/controller identity, transfer limits, event support, and cross-cutting capability attributes.

**Maximum Data Transfer Size (MDTS)**:
An Identify Controller power-of-two limit, in minimum memory pages, on command data transferred between host-accessible memory and the controller; zero reports no limit through this field.

**Identify Controller Capability Directory**:
The dependency-gated region of the I/O Command Set Independent Identify Controller structure that advertises management surfaces, optional Admin commands and logs, firmware and power behavior, host-memory requests, capacity, and protected-memory support.

**Identify Controller I/O Envelope**:
The Identify Controller region that reports required and maximum I/O queue entry sizes, per-queue outstanding-command guidance, the maximum valid NSID, aggregate optional-command support, and fused-operation support.

**Command Quiesce Time (CQT)**:
The reported worst-case interval for a controller to satisfy Immediate Abort requirements for outstanding commands after Keep Alive timeout or other communication loss.

**Explicit Persistent Discovery Connection**:
A Discovery-controller connection requested with a non-zero Keep Alive Timer and maintained with Keep Alive plus asynchronous discovery-change notification.

**Fabric Secure Channel**:
A transport security gate that, when required, must be established before a controller accepts any command on that NVMe Transport.

**NVMe In-band Authentication**:
A post-Connect authentication gate that limits the newly created queue to authentication commands until authentication completes.

## Communication

**Transport Model**:
The memory-based or message-based communication model by which commands, responses, and data move between a host and an NVM subsystem.

**Queue Pair**:
An associated Submission Queue and Completion Queue used to submit commands and return their completions.

**Command Set**:
A defined family of commands with common purpose and queue-processing rules, such as Admin, I/O, or Fabrics.

**Admin Command Set**:
The commands submitted through the Admin Submission Queue, partitioned into common, memory-transport-specific, and message-transport-specific groups.

**Immediate Abort**:
An abort completed before the Abort command CQE is posted and constrained to prevent all later effects of the target command.

**Deferred Abort**:
An optional abort performed after the Abort command CQE, observable through the target command's completion status.

**Asynchronous Event Request (AER)**:
A no-timeout Admin command completed by the controller when an asynchronous event is available for the host.

**Controller Data Queue (CDQ)**:
A controller-managed, host-memory-backed queue to which the controller posts type-specific data; its returned identifier is local to the creating controller.

**User Data Migration Queue**:
A Controller Data Queue associated with one controller that logs user-data modifications during controller migration.

**Device Self-test Operation**:
A short, extended, Host-Initiated Refresh, or vendor-specific diagnostic operation whose active scope is controller-local or subsystem-wide according to `SDSO`.

**Directive**:
A typed host/controller information-exchange mechanism carried by paired Directive Receive and Directive Send Admin commands.

**Pending Asynchronous Event**:
Retained event information awaiting a future AER when no request was outstanding at occurrence time, except for immediate-event semantics.

**Log Page Identifier (LID)**:
The Get Log Page selector that names log content and, through that log definition, its scope and addressing capabilities.

**Index Offset**:
A Get Log Page starting position expressed as an entry index, legal only when the selected LID advertises Index Offset Supported.

**Error Information Log**:
A controller-global newest-first history of recent command-specific and non-command-specific errors.

**SMART / Health Information Log**:
A lifetime-oriented health view containing warning state, temperature, spare-capacity, endurance, and activity information.

**Changed Attached Namespace List**:
A controller log containing namespace attachment, detachment, deletion, or Identify-data changes since its last read; overflow requires full re-enumeration.

**Commands Supported and Effects Log**:
A controller-published per-opcode catalog of command support, possible affected scope, submission/execution recommendations, and capability or data changes.

**Feature Identifiers Supported and Effects Log**:
An interface-instance-specific per-FID catalog of support, one reported scope, UUID selection, and possible capability or data changes.

**NVMe-MI Commands Supported and Effects Log**:
A per-management-opcode catalog of controller support, one reported scope, and possible capability or data changes for commands carried by NVMe-MI Send.

**Command and Feature Lockdown**:
An optional subsystem capability that prohibits selected command opcodes or Set Features identifiers separately for Admin Queue and Management Endpoint receipt paths.

**Command Scope**:
The resource levels that a command may affect, from namespace through controller, NVM Set, Endurance Group, Domain, and NVM subsystem.

**Telemetry Host-Initiated Log**:
A host-triggered, generation-numbered snapshot of implementation-defined controller or NVM-subsystem state carried in 512-byte data blocks.

**Telemetry Controller-Initiated Log**:
A controller-triggered persistent diagnostic capture whose in-band availability is acknowledged by a successful read with `RAE=0`.

**Telemetry Data Area**:
A progressively bounded prefix of telemetry data blocks beginning at block 1; its last-block field determines the area extent.

**Predictable Latency Mode**:
An NVM-Set operating mode that alternates between a bounded Deterministic Window and a preparation-oriented Non-Deterministic Window.

**ANA Group Descriptor**:
A controller-relative record containing an ANA Group's state, optional change count, and attached namespace membership.

**Persistent Event Log Reporting Context**:
A frozen vendor-maintained selection of persistent events used to make a segmented Persistent Event Log retrieval internally consistent.

**NVM Subsystem Hardware Error Event**:
A persistent, vendor-supported hardware-failure record with a standardized error code and optional code-specific additional information.

**Media Unit Status Descriptor**:
A Domain-scoped record of one Media Unit's resource assignment, wear indicators, capacity adjustment, and connected Channels.

**Persistent Event Format**:
A self-delimiting event record with common controller, timestamp, and port attribution followed by optional vendor information and independently revised type-specific data.

**Controller Support Requirement**:
The Mandatory, Optional, Prohibited, or conditional support classification of a command, log page, or Feature for one controller type.

**Power State Descriptor**:
A per-state record of I/O operational status, transition latency, relative performance, power estimates, and optional power-loss processing time.

**Controller Property**:
A dword or qword controller attribute in the common control surface, accessed at its starting offset with its declared width through a transport-defined mechanism.

**Controller Capabilities (CAP)**:
The mandatory 64-bit read-only property describing the limits and low-level behaviors that constrain legal controller configuration.

**Controller Configuration (CC)**:
The mandatory property through which the host selects operating parameters and enables or disables the controller.

**Controller Status (CSTS)**:
The mandatory read-only property reporting readiness, shutdown progress/type, fatal state, processing pause, and subsystem-reset observation.

**Controller Reset**:
The Controller Level Reset initiated by clearing `CC.EN` from one to zero, deleting I/O queues and returning controller processing to an idle disabled state.

**NVM Subsystem Reset**:
A reset affecting the NVM subsystem scope, optionally initiated through `NSSR` and required to recover from a completed NVM Subsystem Shutdown.

**Controller Memory Buffer (CMB)**:
Optional controller-associated memory whose advertised capabilities determine whether queues, lists, and command data may be placed there.

**Persistent Memory Region (PMR)**:
Optional persistent controller-associated memory with explicit enable/readiness, health, error, and write-persistence barrier semantics.

**Boot Partition**:
One of two equal-sized optional controller-resident partitions whose contents may be copied into a host-selected memory buffer for boot use.

**Capability/Configuration Handshake**:
The host reads `CAP` constraints, selects valid `CC` settings, and initializes queue entry sizes before enabling the controller or creating I/O queues.

**Controller Initialization**:
The transport-specific bootstrap followed by configuration, enablement, identification, namespace discovery, I/O queue creation, and optional asynchronous-event setup.

**Controller Ready Timeout (CRTO)**:
The worst-case ready-with-media and, when supported, ready-independent-of-media waits after enabling a controller.

**Controller Ready With Media**:
A readiness mode in which `CSTS.RDY=1` means command processing, attached namespaces, and media needed by Admin commands are ready.

**Controller Ready Independent of Media**:
A readiness mode in which non-media command processing may become ready before attached namespaces and media, with separate `CRTO.CRIMT` and `CRTO.CRWMT` deadlines.

**Controller Shutdown**:
A shutdown mechanism requested through `CC.SHN` for one controller; `CSTS.ST=0` distinguishes its progress from NVM Subsystem Shutdown.

**NVM Subsystem Shutdown**:
A shutdown initiated through one controller that covers every controller in the associated Domain or the whole NVM subsystem and requires an NVM Subsystem Reset before normal execution resumes.

**Queue Level Reset**:
Deletion followed by recreation of an I/O queue or queue pair, with transport-specific delete/disconnect and create/connect procedures.

**Firmware Update Sequence**:
A non-overlapping series of Firmware Image Download and Firmware Commit commands kept on one controller or Management Endpoint through commit.

**Pending Firmware Activation**:
A committed firmware image awaiting the Controller Level, NVM Subsystem, or Conventional Reset required to activate it.

**Format NVM**:
An Admin operation that changes media format attributes and optionally erases user data at a capability-selected namespace scope.

**User Data Erase**:
A Format NVM secure-erase mode that removes all affected user content without defining the post-erase media pattern.

**Cryptographic Erase**:
A Format NVM secure-erase mode that makes affected user content inaccessible by deleting its encryption key.

**I/O Command Set Association**:
The exactly-one binding chosen when a namespace is created that fixes its data model and command interpretation for the namespace's lifetime.

**Multi-Path I/O**:
Two or more completely independent paths between one host and one namespace.

**Namespace Sharing**:
Two or more hosts accessing one common shared namespace through different controllers.

## Storage resources

**Domain**:
A storage-resource grouping inside one NVM subsystem that contains Endurance Groups.

**Division**:
An event or action that disrupts communication between Domains in an NVM subsystem and may partition global state or namespace access.

**Domain Identifier**:
A subsystem-scoped identifier used to associate controllers and Endurance Groups with a shared Domain and potential failure boundary.

**Endurance Group**:
A storage-resource grouping inside one Domain that provides an endurance and resource-management boundary.

**NVM Set**:
A collection of NVM within one Endurance Group from which namespaces may be allocated.

**NVM Set Identifier**:
A non-zero 16-bit subsystem-unique identifier selecting the NVM Set associated with an action.

**Endurance Group Identifier**:
A non-zero 16-bit subsystem-unique identifier selecting an endurance-management boundary.

**Reclaim Group**:
A placement-oriented grouping within one Endurance Group that contains Reclaim Units.

**Reclaim Unit**:
The granular resource managed within a Reclaim Group for host-directed placement behavior.

**Reclaim Unit Handle**:
An Endurance Group-scoped FDP indirection that references one Reclaim Unit in each Reclaim Group and may be retargeted when a unit is filled.

**NVM Capacity Model**:
The optional reporting and allocation hierarchy spanning subsystem, Domain, Endurance Group, NVM Set, formatted namespace, and Media Unit views.

**Fixed Capacity Management**:
Capacity configuration by selecting a predefined descriptor that instantiates its Endurance Groups and NVM Sets.

**Variable Capacity Management**:
Capacity configuration by explicitly creating or deleting Endurance Groups and NVM Sets with requested byte capacities.

**Media Unit**:
A physical-media component assigned to exactly one Endurance Group and, when NVM Sets are supported, exactly one NVM Set; it connects to one or more Channels.

**Channel**:
A data-transfer path connected to one or more Media Units; Media Unit organization across Channels affects bandwidth and isolation.

**Exported NVM Subsystem**:
A logical NVM subsystem that presents underlying NVM resources for remote access.

**Dispersed Namespace**:
A shared namespace concurrently accessible through controllers in two or more participating NVM subsystems.

**Idempotent Command**:
A command whose repeated submission, with no intervening commands, produces the same subsystem end state and result.

**Reachability Group**:
A set of namespaces attached to a controller that share the same reachability attributes.

**Reachability Association**:
A controller-relative set of Reachability Groups sharing reachability characteristics for operations between their represented namespaces.

**FDP Configuration**:
An Endurance Group-scoped static placement configuration describing Reclaim Groups, Reclaim Unit Handles, placement limits, and current validity.

**FDP Runtime Views**:
The Endurance Group-scoped Reclaim Unit Handle Usage, FDP Statistics, and FDP Events logs that observe ownership, lifetime media activity, and enabled placement events for the active configuration.

**Reservation Notification Log**:
A controller-local destructive-read queue of unmasked reservation changes whose sequence count exposes lost notifications.

**Sanitize Status Log**:
A persistent observation of sanitize progress and the most recent operation outcome, retained across reset and power cycle.

**Management Address**:
A typed, null-terminated UTF-8 URI identifying an NVM-subsystem management agent or fabric interface manager.

**I/O Command Set Combination**:
An indexed bitmap of I/O Command Sets that a controller supports using simultaneously; the selected I/O Command Set Profile enables only sets present in that combination.

**Supported Controller State Formats**:
An Identify directory whose indexed NVMe-version and vendor-UUID lists name controller-state formats usable by migration operations.

**Virtual Queue Resource (VQ Resource)**:
A private or flexible controller resource used to provide queue capacity to a primary controller or its associated secondary controllers.

**Virtual Interrupt Resource (VI Resource)**:
A private or flexible controller resource used to provide interrupt capacity to a primary controller or its associated secondary controllers.

**Controller State Snapshot**:
The state image returned by Migration Receive; it is internally consistent only when the target remains suspended and reset/property-change conditions are satisfied during retrieval.

**Controller Suspension**:
A migration state in which the target stops fetching commands, drains previously fetched commands except outstanding AERs, and stops initiating transport transactions while properties, CMB, and PMR remain accessible under defined restrictions.

**Controller State Transfer Sequence**:
One `SEQIND=11b` Migration Send command or a completion-ordered `01b`, zero-or-more `00b`, `10b` sequence that transfers, verifies, and commits a controller-state image.

**Controller Suspension**:
A migration state in which a controller stops fetching new commands, completes previously fetched commands except outstanding AERs, and stops initiating transport transactions while properties, CMB, and PMR remain accessible.

**Controller State Transfer Sequence**:
One complete or completion-ordered first/middle/last series of Migration Send commands whose image is verified and committed only after the final successful fragment.

**Namespace Attachment**:
The persistent relationship that makes an allocated namespace active on an I/O controller; it is managed independently from namespace creation and deletion.

**Sanitize Command Completion**:
Confirmation that a background sanitize operation was successfully started, not confirmation that erase, verification, or post-verification deallocation has completed.

**Security Protocol Exchange**:
The Security Send/Receive carrier in which NVMe defines direction and selectors while the selected external security protocol defines payload and request/result association.

**Temperature Threshold Hysteresis**:
A Kelvin margin that changes the recovery boundary of an active over- or under-temperature event without changing its initial trigger threshold.

**Autonomous Power State Transition (APST)**:
A controller policy that moves from a source power state to a selected non-operational state after a configured continuous idle interval.

**Non-Operational Power State Permissive Mode**:
A policy allowing controller-initiated background work to temporarily exceed a non-operational state's power limit up to the last operational-state limit.

**Host Behavior Support**:
A replace-not-merge Feature through which a host explicitly permits controller functionality that depends on host retry, telemetry, namespace-identity, or command-format behavior.

**Namespace Admin Label**:
A saved, null-terminated UTF-8 label intended for human namespace identification and reset to all nulls by successful Sanitize.

**Host Metadata Feature**:
A non-saveable, atomic 4 KiB typed descriptor set describing a host platform, controller view, or controller-specific namespace view.

**Controller Data Queue Tail Trigger**:
A one-shot Feature condition that reports when the controller posts an entry into a selected CDQ slot and clears itself after firing.

**Persist Through Power Loss (PTPL)**:
A namespace reservation state selecting whether reservations and registrants survive power loss.

**Namespace Write Protection**:
A namespace state machine that gates writes and commits volatile namespace data before entering a protected state.

**Host Memory Buffer (HMB)**:
Host memory assigned for exclusive controller use until a successful disable completion returns ownership to the host.

**Migration Change Tracking**:
The Track Send/Track Receive protocol that logs namespace user-data changes or reports host-memory ranges modified by a selected controller during migration.

**Shadow Doorbell Buffer**:
A host-updated, page-aligned mirror of memory-based queue doorbells paired with a controller-updated EventIdx page for an emulated controller.

**Secondary Controller Resource Lifecycle**:
The Virtualization Management sequence that takes a secondary controller offline, assigns flexible VQ/VI resources, and returns it online.

**Discovery Information Management**:
The message-based Admin protocol by which a Host, Direct Discovery Controller, or Centralized Discovery Controller registers, de-registers, or atomically updates keyed discovery records held by a CDC or DDC.

**Discovery Entry Key**:
The selected fields used to match a Discovery Information Management entry to an existing registration record; host entries use transport-address-based keys, while subsystem entries may instead use port-ID-based keys.

**Fabric Zoning Data Key (ZDK)**:
A CDC-assigned key returned by Fabric Zoning Lookup and used by a DDC to select a Zoning data structure for fragmented transfer.

**Exported Namespace Association**:
The mapping between an Exported Namespace ID and an attached active Underlying Namespace, independent of later controller attachment of the Exported Namespace.

**Allowed Host List**:
The per-Exported-NVM-Subsystem set of Host Identifier/NQN entries that may connect through selected exported ports when access mode is restricted.

**Send Discovery Log Page (SDLP)**:
The CDC-to-pull-model-DDC command that transfers Discovery-family log bytes and supports a one-shot resend registration.

**Fabrics SQ Flow Control Negotiation**:
The Connect handshake in which SQ flow control is disabled only when the host requests it and the controller acknowledges with `SQHD=FFFFh`.

**Disconnect Final-Response Barrier**:
The rule that a Fabrics Disconnect response is the deleted I/O queue pair's last CQE and ends further command processing on that pair.

**I/O Cancel**:
An I/O-queue cancellation request whose CQE counts immediate aborts and deferred-abort candidates while target CQEs remain authoritative for eventual outcomes.

**Flush Persistence Barrier**:
The ordering point that commits completed namespace data and metadata from enabled volatile write cache to non-volatile media.

**Namespace Registrant**:
A host that has registered a reservation key for a namespace and may participate in reservation access policy.

**Reservation Holder**:
The registrant whose host currently holds a typed namespace reservation.

**ANA Group**:
A subsystem-unique grouping whose member namespaces share one controller-relative asymmetric access state and transition together.

**ANA Persistent Loss**:
A terminal controller-to-ANA-Group relationship in which user data remains inaccessible and the relationship cannot transition to another ANA state.
