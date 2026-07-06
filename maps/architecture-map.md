# Architecture map

Explanatory overview synthesized from the Theory of Operation and Architecture sections:

```text
                              NVM Subsystem
        +----------------------------------------------------------+
        |                                                          |
Host <--+--> Controller(s) <-- command sets --> Namespace(s)       |
        |        ^                                  |              |
        |        | properties / queues              v              |
        |        +-------------------------- storage resources      |
        |                                                          |
        +----------------------------------------------------------+
             ^                                      ^
             | memory-based transport               | message-based transport
             | (e.g. PCIe)                          | (e.g. Fabrics)
```

The controller is the host-facing interface to the NVM subsystem; namespaces are addressable storage abstractions; transports define how commands, completions, and data cross the boundary. [PDF pp. 39-46](../_source/pages/page-039.md) [PDF p. 58](../_source/pages/page-058.md)

For Fabrics, a Discovery Service identifies accessible subsystem paths before a host connects; required secure-channel and in-band-authentication gates then constrain command acceptance. [PDF pp. 45-46](../_source/pages/page-045.md)

This architecture is governed by the Base Specification, extended by I/O Command Set specifications, and carried by transport-specific bindings. That family relationship is applicability, not another architecture layer. [PDF p. 23](../_source/pages/page-023.md)

For remote export, an [Exported NVM Subsystem](../concepts/exported-and-underlying-resources.md) is a logical presentation of underlying namespaces and ports. Namespace sharing may stay within one subsystem or span participating subsystems as a dispersed namespace. [PDF pp. 30-31, 33-36](../_source/pages/page-030.md)

Controllers have two independent classifications: a subsystem chooses one allocation model (static or dynamic), while each controller has a functional type (I/O, Administrative, or Discovery). Memory-based controllers are static; Discovery controllers are dynamic. All types retain the common Admin Queue pair, while only I/O controllers provide user-data I/O. [PDF pp. 58-62](../_source/pages/page-058.md)

The [controller property space](../concepts/controller-properties.md) is a common transport-independent control surface. Message-based controllers access it with Property Get/Set, while memory-based controllers use binding-defined mechanisms; capability fields constrain the configuration selected before controller enablement. [PDF pp. 71-79](../_source/pages/page-071.md)

Memory-based controllers may expose two distinct [controller memory windows](../concepts/controller-memory-windows.md): the CMB for supported queue/list/data placement and the PMR with an explicit readiness and persistence-barrier contract. Their enabled controller address ranges may not overlap. [PDF pp. 86-94](../_source/pages/page-086.md)

Shared namespaces preserve one NSID and Identify Namespace view across attached controllers, but command atomicity follows the receiving controller and cross-controller ordering remains a host/application responsibility. Independent ports provide reset-failure isolation; access characteristics may still be asymmetric by controller. [PDF pp. 55-57](../_source/pages/page-055.md)

## Storage resource hierarchy

```text
NVM Subsystem
`-- Domain
    `-- Endurance Group
        `-- either NVM Set(s) -> Namespace
            or Reclaim Group(s) -> Reclaim Unit(s) -> Namespace
```

This explanatory projection preserves the specification's alternative Endurance Group organizations rather than presenting both branches as simultaneous containment. The hierarchy was checked against rendered Figures 11-15; namespaces may share an NVM Set or span Reclaim Groups within one Endurance Group. [PDF pp. 46-51](../_source/pages/page-046.md)

In a multi-Domain subsystem, [Domains and Divisions](../concepts/domains-and-divisions.md) add explicit communication/fault boundaries. A division can partition subsystem knowledge and namespace access; reservation recovery gates access through ANA until state is synchronized or declared persistently lost. [PDF pp. 104-107](../_source/pages/page-104.md)

For message-based transports, a pre-established transport connection carries Connect on the queue it creates. The resulting queue pair may be authentication-gated; SQ capacity is either released through ordered `SQHD` updates or managed entirely by the host when SQ flow control is disabled, while Completion Queues have no NVMe-level pointer flow control. [PDF pp. 114-119](../_source/pages/page-114.md)

[Command arbitration](../concepts/command-ordering-and-arbitration.md) chooses a Submission Queue, not a global execution order. Round robin treats all queues equally; the optional weighted mechanism applies strict Admin and Urgent classes before weighted High/Medium/Low service. Fused pairs remain adjacent and atomic across this otherwise independent processing model. [PDF pp. 120-123](../_source/pages/page-120.md)

[Controller initialization](../concepts/controller-initialization.md) converges after transport-specific bootstrap: memory-based controllers program Admin queue properties, while Fabrics controllers establish a connection, Connect the Admin pair, and authenticate when requested. Both then select capability-valid configuration, enable, Identify, enumerate namespaces, and create I/O queues. [PDF pp. 124-127](../_source/pages/page-124.md)

Initialization has two readiness contracts: ready-with-media couples `CSTS.RDY` to full media usability, while ready-independent-of-media permits an earlier command-processing milestone bounded by `CRTO.CRIMT` and a later media milestone bounded by `CRTO.CRWMT`. Shutdown status is a separate axis: subsystem shutdown overrides controller shutdown, and `CSTS.ST/SHST` identify its scope and progress even if `CC.EN/CSTS.RDY` remain set. [PDF pp. 128-134](../_source/pages/page-128.md)

[NVM Subsystem Shutdown](../concepts/controller-enable-shutdown-reset.md) operates at whole-subsystem scope in a single-Domain design and at Domain or subsystem scope in a multi-Domain design. Only an NVM Subsystem Reset clears the portable post-shutdown boundary; that reset fans out into Controller Level Resets and PMR disablement across its scope. Queue-level reset is narrower still: delete and recreate only the affected I/O queues. [PDF pp. 135-141](../_source/pages/page-135.md)

The [NVM Capacity Model](../concepts/nvm-capacity-model.md) allocates capacity down `Domain -> Endurance Group -> NVM Set -> formatted Namespace`, while Media Units are assigned to Endurance Groups/NVM Sets and connected to Channels. Those are complementary allocation and data-path views, not extra containment levels beneath namespaces. [PDF pp. 141-143](../_source/pages/page-141.md)

Media Unit organization then chooses a performance tradeoff: one group/set may span every Channel for aggregate bandwidth, vertical groups may isolate one set per Channel, or horizontal SLC/QLC groups may each span all Channels with different capacity adjustments. Capacity accounting is read and updated at the nearest supported parent level. [PDF pp. 143-146](../_source/pages/page-143.md)

[Keep Alive](../concepts/keep-alive.md) adds a per-controller communication watchdog. Command Based mode requires periodic Keep Alive completions; Traffic Based mode treats fetched Admin/I/O commands as liveness and may take up to `2 * KATT` after the last fetch to detect failure. Controller timeout sets fatal status and, for message transports, tears down the association. [PDF pp. 146-151](../_source/pages/page-146.md)

[Firmware Update Lifecycle](../concepts/firmware-update-lifecycle.md) serializes image portions through download and commit, then activates immediately or at a reported reset boundary. Reset activation reconnects the controller to initialization and I/O queue allocation. [PDF pp. 152-154](../_source/pages/page-152.md)

The [Common Command Format](../concepts/common-command-format.md) gives Admin and I/O commands a shared 64-byte SQE envelope. `CDW0.PSDT` selects PRP or SGL interpretation for both metadata and data pointers, while the transport constrains legal encodings; Fabrics commands use a separate common SQE. [PDF pp. 155-159](../_source/pages/page-155.md)

The [Completion Queue Entry and Status](../concepts/completion-queue-entry-and-status.md) closes that command envelope: `CID` and, outside Fabrics, `SQID` correlate the command; `SQHD` returns consumed-SQ capacity; and Status separates retry/log hints from the `SCT`/`SC` outcome code. Fabrics preserves the outcome encoding while reserving memory-queue-only fields. [PDF pp. 160-163](../_source/pages/page-160.md)

Its status families then separate opcode-specific rejection, media/data-integrity failure, and path-specific failure. For memory Completion Queues, the expected Phase Tag toggles on each controller Tail wrap so the host can distinguish newly posted entries from stale slot contents without rewriting consumed slots. [PDF pp. 166-173](../_source/pages/page-166.md)

[Data Pointer Layouts](../concepts/data-pointer-layouts.md) expand the command envelope into memory coverage. PRPs name page-sized regions through fixed 64-bit entries and packed/chained PRP Lists; SGLs describe typed, length-bearing data and segment descriptors. [PDF pp. 173-175](../_source/pages/page-173.md)

SGL segments form a constrained descriptor graph: only the final descriptor may link onward, a Last Segment terminates the graph, and Bit Bucket descriptors may consume destination-stream bytes without placing them in host memory. [PDF pp. 175-180](../_source/pages/page-175.md)

[Feature Values and Scope](../concepts/feature-values-and-scope.md) connects controller configuration to reset scope. A whole-subsystem reset restores affected Features from saved/default state, while a reset covering only part of a multi-controller or multi-Domain subsystem leaves broader resource-scoped values unchanged unless their Feature definition says otherwise. [PDF pp. 181-183](../_source/pages/page-181.md)

[Identity, Name, and List Formats](../concepts/identity-name-and-list-formats.md) separates global durable identity from local command handles: NQN identifies a subsystem, EUI64/NGUID/UUID identify a namespace, and subsystem NQN plus `CNTLID` identifies a controller globally. Protocol-carried UTF-8 names are compared as raw bytes. [PDF pp. 183-190](../_source/pages/page-183.md)

The [Identify Command Model](../concepts/identify-command-model.md) exposes a capability directory rather than a bag of independent flags. Optional Admin commands, logs, firmware activation, host memory, capacity, and RPMB fields have cross-field gates—for example, live-migration support suppresses `HMPRE`, capacity-management support requires capacity reporting, and non-zero RPMB count requires Security Send/Receive. Its continuation binds operational capabilities to ANA, Sanitize, and quiesce limits, then defines queue geometry and aggregate I/O command support. [PDF pp. 327-342](../_source/pages/page-327.md)

The final Identify Controller region advertises Format/cache/atomicity, namespace protection, Copy/SGL behavior, attachment and refresh limits, tracked migration/CDQ capacity, subsystem NQN, and Fabrics capsule/controller attributes before the power-state descriptor array. [PDF pp. 343-350](../_source/pages/page-343.md)

[Power State Descriptors](../concepts/power-state-descriptors.md) turn that array into a host policy surface spanning operational availability, transition latency, relative performance, idle/active/maximum power, and power-loss vault/recovery timing. Identify then switches to paginated namespace/NVM Set identity and CSI-qualified capability views. [PDF pp. 350-358](../_source/pages/page-350.md)

The [Identify Command Model](../concepts/identify-command-model.md) then joins namespace runtime readiness, format progress, access/reachability membership, and storage-resource membership to a family of cursor-based directories. Its I/O Command Set Combination vectors constrain which command sets may be enabled together, while Supported Controller State Formats supplies the version and vendor-UUID indices consumed by migration commands. [PDF pp. 359-366](../_source/pages/page-359.md)

[Controller Virtualization Resources](../concepts/controller-virtualization-resources.md) divide primary-controller VQ/VI capacity into flexible pools shared with secondary controllers and private primary capacity. The secondary directory makes online state, SR-IOV VF identity, and current VQ/VI assignment observable even for offline controllers. Message-based Identify separately exposes generation-counted [underlying namespace and port directories](../concepts/exported-and-underlying-resources.md) used to assemble exported Fabrics resources. [PDF pp. 366-370](../_source/pages/page-366.md)

[Command and Feature Lockdown](../concepts/command-and-feature-lockdown.md) separates its read-side inventory from a subsystem-wide write-side enforcement command. [Controller Migration](../concepts/controller-migration.md) then uses advertised state-format indices to retrieve controller state; suspension plus reset/property stability determines snapshot consistency, while a completion-ordered `SEQIND` sequence transfers, verifies, commits, and reconstructs I/O queue state before Resume. [PDF pp. 370-383](../_source/pages/page-370.md)

The migration write path suspends command fetching, optionally coordinates a User Data Migration Queue, transfers one complete image or an ordered fragment sequence, and commits only after the last fragment is verified. The standard image reconstructs ordered I/O SQ/CQ creation attributes and live pointers; Resume is blocked while received state remains uncommitted. [PDF pp. 375-383](../_source/pages/page-375.md)

[Namespace Management](../concepts/namespace-management-lifecycle.md) separates subsystem allocation from controller attachment: create allocates but does not attach, attachment persists across resets, and delete removes all attachments. [Sanitize](../concepts/sanitize-operation-lifecycle.md) instead launches a subsystem-wide background operation whose command CQE confirms start, while its status log tracks actual progress and outcome. [PDF pp. 383-390](../_source/pages/page-383.md)

[Security Protocol Exchange](../concepts/security-protocol-exchange.md) is a protocol-neutral Send/Receive carrier whose selected security protocol owns payload and correlation semantics. [Feature Values and Scope](../concepts/feature-values-and-scope.md) then defines the common mutable-policy plane: saveability, scope, memory-buffer attributes, UUID qualification, completion ordering, and rediscovery. Initial [common controller Features](../concepts/common-controller-features.md) apply that plane to arbitration, power, temperature, and volatile caching. [PDF pp. 390-398](../_source/pages/page-390.md)

Feature policy then connects notification, time, and power behavior: the AER configuration mask enables event families; APST and permissive non-operational mode independently choose idle entry and background-work legality; Timestamp exposes origin/continuity limits; Keep Alive remains transport-constrained; and Predictable Latency configuration binds mode/windows and warning thresholds to one NVM Set. [PDF pp. 399-407](../_source/pages/page-399.md)

Host Behavior Support creates an explicit compatibility handshake before controllers use retry, telemetry, dispersed-namespace, or Copy-format behavior. Subsequent Features configure subsystem Sanitize policy, Endurance Group events, controller command-set combinations, rotational spinup, power-loss signaling, FDP activation/events, and a sanitize-resettable human namespace label. [PDF pp. 407-415](../_source/pages/page-407.md)

The Feature plane also drives host/controller coordination: CDQ head release and one-shot tail events; management-agent URI discovery; atomic typed metadata annotations; a pre-boot progress marker; and Host Identifier registration used to associate controllers and reservation rights. [PDF pp. 415-423](../_source/pages/page-415.md)

Reservation masks and PTPL configure which namespace reservation changes are reported and whether reservation state survives power loss. Namespace and Boot Partition write-protection Features control write eligibility, while memory-based transport Features negotiate queue counts, interrupt coalescing, and controller-exclusive Host Memory Buffer ownership. [PDF pp. 424-431](../_source/pages/page-424.md)

HMB descriptors map controller-page-sized host-memory extents and expose current allocation attributes. The migration plane then pairs Track Send with destructive Track Receive reporting: one mode logs namespace data changes into a User Data Migration Queue, while another records host-memory ranges written by a selected controller until suspension closes the change window. [PDF pp. 431-438](../_source/pages/page-431.md)

Memory tracking negotiates fallback granularity/capacity when resources are insufficient. PCIe queue management then creates CQ storage before dependent SQs, deletes SQs before CQs, and uses successful SQ deletion as the final explicit/implicit command-completion barrier. Shadow Doorbell/EventIdx pages provide an emulation-oriented mirror, while Virtualization Management reallocates VQ/VI resources through a secondary Offline/Assign/Online lifecycle. [PDF pp. 439-447](../_source/pages/page-439.md)

Message-based Admin management separates exported configuration from underlying storage: exported subsystems begin empty under a selected access mode, while clearing exported configuration preserves underlying namespaces and subsystems. Discovery Information Management independently maintains CDC/DDC registration records through keyed register, de-register, and atomic update tasks. [PDF pp. 448-455](../_source/pages/page-448.md)

Discovery registration entries then attach typed, length-delimited Host Identifier and admin-label attributes. [Fabric Zoning](../concepts/fabric-zoning-data-transfer.md) separately maps lookup data to a Zoning Data Key and transfers Zoning structures as offset fragments, while exported-resource management associates one ENSID with an attached underlying namespace without implicitly attaching the exported namespace to a controller. [PDF pp. 455-462](../_source/pages/page-455.md)

The export-management plane controls both reachability and exposure: Allowed Host Lists gate hosts per exported subsystem and port, while Exported Ports bind those subsystems to underlying transport ports. A CDC can then push segmented Discovery, Host Discovery, or AVE Discovery log bytes to a pull-model DDC; the DDC's completion may request one future resend after an update. [PDF pp. 463-470](../_source/pages/page-463.md)

The Fabrics command plane uses one opcode with typed functions. Authentication Send/Receive carry an external security protocol, Admin Connect allocates a controller and establishes identity, I/O Connect inherits that identity while negotiating per-pair SQ flow control, and Disconnect makes its response the final CQE without directly deleting the transport connection. [PDF pp. 471-479](../_source/pages/page-471.md)

Fabrics Property commands map message capsules onto the common controller-property surface. The common I/O plane then adds three control paths: Cancel selects fetched I/O commands for immediate or deferred abort, Flush creates a volatile-to-non-volatile completion barrier, and I/O Management Receive returns operation-specific state such as FDP Reclaim Unit Handle status. [PDF pp. 479-487](../_source/pages/page-479.md)

I/O Management Send retargets FDP Placement Identifiers to empty Reclaim Units with explicitly partial failure and concurrent-write semantics. Namespace reservations form a separate shared-access state machine: hosts register keys, acquire or preempt typed reservations, release or clear state, select power-loss persistence, and inspect generation-counted registrant records across dispersed participants. [PDF pp. 487-495](../_source/pages/page-487.md)

ANA models access as a controller-to-group relationship. Group members transition together inside one Domain, while each controller may report Optimized, Non-Optimized, Inaccessible, Persistent Loss, or transient Change. Notices expose important transitions, and adverse states selectively alter command availability, retry routing, and reported capacity rather than disabling the entire Admin plane. [PDF pp. 497-503](../_source/pages/page-497.md)

The [Admin Command Model](../concepts/admin-command-model.md) partitions management operations into common, memory-specific, and message-specific groups while preserving one Admin Queue envelope. Format and sanitize narrow the runnable set to an explicit recovery/control allowlist. [PDF pp. 191-193](../_source/pages/page-191.md)

[Command Abort Semantics](../concepts/command-abort-semantics.md) distinguishes an immediate effects barrier from deferred cancellation: the Abort CQE reports whether immediate abort occurred, but the target command's CQE remains authoritative for whether a later deferred abort succeeded. [PDF pp. 194-195](../_source/pages/page-194.md)

[Asynchronous Event Reporting](../concepts/asynchronous-event-reporting.md) keeps long-lived Admin commands available for controller-initiated notifications. Most event types mask after reporting and are cleared by reading an associated log, while immediate and one-shot events use different retention rules. [PDF pp. 195-202](../_source/pages/page-195.md)

[Capacity Management Operations](../concepts/capacity-management-operations.md) turns the capacity hierarchy into an Admin lifecycle: fixed mode instantiates a predefined configuration, while variable mode creates or deletes Endurance Groups and NVM Sets with explicit byte capacity and containment-aware deletion. [PDF pp. 203-206](../_source/pages/page-203.md)

[Controller Data Queues](../concepts/controller-data-queues.md) provide controller-produced streams in stable host-memory mappings; the Base-defined User Data Migration Queue records modified user data for controller migration. [PDF pp. 206-210](../_source/pages/page-206.md)

[Device Self-test](../concepts/device-self-test.md) is a controller/subsystem-scoped state machine whose result log records start and abort transitions. [Directive Exchange](../concepts/directive-exchange.md) instead provides a type-selected pair of host-to-controller and controller-to-host transfer envelopes. [PDF pp. 210-214](../_source/pages/page-210.md)

[Format NVM](../concepts/format-nvm-lifecycle.md) applies a selected media format and optional secure erase at namespace, attached-controller, or whole-subsystem scope according to `FNS`/`SENS`; it gates conflicting I/O until completion. [PDF pp. 217-220](../_source/pages/page-217.md)

[Controller Data Queues](../concepts/controller-data-queues.md) add a separate controller-to-host posting path backed only by host memory. Creation returns a controller-local identifier; deletion or reset ends the queue lifetime, and the defined User Data Migration Queue logs modifications for a selected controller. [PDF pp. 206-210](../_source/pages/page-206.md)

[Device Self-test](../concepts/device-self-test.md) is a stateful Admin operation whose scope may be controller-local or subsystem-wide. Standard starts are rejected while a test is active; abort updates the self-test log in a defined order before completion. [PDF pp. 210-212](../_source/pages/page-210.md)

[Directive Exchange](../concepts/directive-exchange.md) provides symmetric Admin envelopes for host-to-controller and controller-to-host typed data. Directive Type and Operation own the payload and status semantics, while the generic commands own direction, buffer, and length. [PDF pp. 212-214](../_source/pages/page-212.md)

[Log Page Retrieval](../concepts/log-page-retrieval.md) is the diagnostic read plane across controller, namespace, Domain, subsystem, and Endurance Group scopes. Its offset model supports byte slices universally and indexed records only where the selected LID advertises that capability. [PDF pp. 223-228](../_source/pages/page-223.md)

[Error and Health Logs](../concepts/error-and-health-logs.md) split diagnostics into recent discrete failures and longer-lived health state. Error records correlate commands and parameters; SMART/Health state supplies warnings, temperature, spare, and endurance indicators. [PDF pp. 229-231](../_source/pages/page-229.md)

SMART/Health then extends that snapshot with retained activity, power, error, and thermal counters. Firmware-slot and changed-namespace logs expose pending operational state, while the [Commands Supported and Effects](../concepts/command-effects-and-support.md) catalog lets a host serialize potentially conflicting commands and rediscover capabilities after declared changes. [PDF pp. 232-238](../_source/pages/page-232.md)

The [Device Self-test](../concepts/device-self-test.md) log connects the command state machine to observation: it reports current progress and preserves the twenty newest completed or aborted results in newest-first order. [PDF pp. 238-239](../_source/pages/page-238.md)

Self-test results retain validity-gated failure location and completion-style status. [Telemetry Capture Logs](../concepts/telemetry-capture-logs.md) provide a broader vendor-diagnostic snapshot: a host can explicitly freeze state, or consume a persistent controller-triggered capture and acknowledge its asynchronous-event availability with `RAE=0`. [PDF pp. 240-246](../_source/pages/page-240.md)

At storage-resource scope, the Endurance Group log contrasts host traffic with physical-media writes and reports spare, life, error, and capacity state. [Predictable Latency Mode](../concepts/predictable-latency-mode.md) exposes per-NVM-Set deterministic/non-deterministic windows and pending event aggregation. [PDF pp. 247-250](../_source/pages/page-247.md)

The ANA log is a controller-relative snapshot of group state and attached namespaces, guarded by global/per-group change counts. The [Persistent Event Log](../concepts/persistent-event-log.md) instead preserves subsystem-global history and freezes a reporting context for consistent segmented retrieval. [PDF pp. 250-255](../_source/pages/page-250.md)

Persistent-event framing separates a common controller/time/port envelope from independently revised event data. Its first standardized records preserve periodic health snapshots plus firmware, timestamp, cross-controller reset, and hardware-error evidence, while a support bitmap lets parsers discover later optional families. [PDF pp. 256-263](../_source/pages/page-256.md)

Hardware-error records attach PCIe, warning, CQE, power-loss, or readiness state captured at failure time. Namespace, Format, Sanitize, and Set Feature records then form an operational journal that separates accepted parameters from completion state and partial failure. [PDF pp. 263-271](../_source/pages/page-263.md)

Persistent events close with telemetry, thermal, media-verification, vendor, and TCG extension records. At resource scope, Endurance Group event aggregation points to group health detail, while Media Unit Status and Supported Capacity Configuration logs expose the active physical assignment and alternative Domain configurations. [PDF pp. 272-279](../_source/pages/page-272.md)

Supported configurations expand into ordered Endurance Group, NVM Set, Channel, and Media Unit descriptor trees. The adjacent effects catalogs expose interface-specific Feature and NVMe-MI support plus possible scope and state changes; [Command and Feature Lockdown](../concepts/command-and-feature-lockdown.md) then provides a distinct enforcement inventory for in-band and out-of-band paths. [PDF pp. 279-286](../_source/pages/page-279.md)

[Boot Partitions](../concepts/boot-partitions.md) can also be read through `Get Log Page`: the selector chooses one partition and the returned header describes the active partition and size without changing the register-based transfer state. [PDF pp. 286-287](../_source/pages/page-286.md)

[Flexible Data Placement](../concepts/flexible-data-placement-configurations.md) joins a static Endurance Group configuration to three runtime views: handle ownership, saturating write/erase statistics, and an enabled oldest-first event history. Placement Identifier interpretation is configuration-dependent because `RGIF` decides whether high bits select a Reclaim Group. [PDF pp. 294-301](../_source/pages/page-294.md)

Two stateful logs use different consumption contracts. [Reservation Notification](../concepts/reservation-notification-log.md) is a destructive-read FIFO whose sequence exposes lost notifications, while [Sanitize Status](../concepts/sanitize-operation-status.md) is a retained cross-reset observation of operation progress, outcome, estimates, and current/failure state. [PDF pp. 301-305](../_source/pages/page-301.md)

The Fabrics [Discovery log](../concepts/fabrics-discovery-and-authentication.md) negotiates host-filtered, port-local, all-subsystem, and extended-entry views. Multi-command retrieval is a generation-validated snapshot; each record is a referral, a namespace-capable subsystem path, or another path to the current Discovery Service. [PDF pp. 306-311](../_source/pages/page-306.md)

The same discovery plane also exposes registered-host and AVE inventories plus a pull-model DDC-to-CDC request channel. The [Identify Command Model](../concepts/identify-command-model.md) then provides the general 4 KiB introspection operation: `CNS` selects a structure and determines whether namespace, controller, command-set, or UUID qualifiers participate. [PDF pp. 312-319](../_source/pages/page-312.md)
