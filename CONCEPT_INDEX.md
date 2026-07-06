# Concept index

## System model

- [Specification Family and Scope](concepts/specification-family-and-scope.md)
- [NVM Subsystem](concepts/nvm-subsystem.md)
- [Controller](concepts/controller.md) (static/dynamic models; I/O, Administrative, and Discovery types)
- [Controller Virtualization Resources](concepts/controller-virtualization-resources.md) (primary/secondary VQ and VI resource pools, assignments, and controller directory)
- [Namespace](concepts/namespace.md)
- [Namespace Identifiers](concepts/namespace-identifiers.md) (validity, allocation, controller attachment, broadcast, identity)
- [Namespace Access Models](concepts/namespace-access-models.md) (private/shared/dispersed namespaces, multi-path I/O, namespace sharing, asymmetric access)
- [Namespace Topology and Change Logs](concepts/namespace-topology-and-change-logs.md) (dispersed participants, reachability groups/associations, allocated-namespace changes)
- [Namespace Management Lifecycle](concepts/namespace-management-lifecycle.md) (create/delete allocation and persistent controller attachment)
- [Namespace Write Protection](concepts/namespace-write-protection.md) (protection states, transition restrictions, and cache commit barrier)
- Host
- [Transport Models](concepts/transport-models.md) (ports, fabric interfaces, capsules, and SGL/data placement)
- [Association and Command Lifecycle](concepts/association-and-command-lifecycle.md)
- [Fabrics Discovery and Authentication](concepts/fabrics-discovery-and-authentication.md) (filter negotiation, consistent snapshots, referrals, fixed/extended entries, and security gates)
- [Fabric Zoning Data Transfer](concepts/fabric-zoning-data-transfer.md) (lookup, key/transaction context, fragmented receive/send, and shared errors)

## Communication

- [Data Representation](concepts/data-representation.md)
- [Transport Models](concepts/transport-models.md)
- [Queue Pair](concepts/queue-pair.md) (memory and message-based creation, flow control, capacity, and deletion)
- [Command Sets](concepts/command-sets.md) (queue routing and namespace lifetime binding)
- [Command Ordering and Arbitration](concepts/command-ordering-and-arbitration.md) (fused operations, candidate selection, priorities, outstanding commands)
- [Command Lifecycle](concepts/association-and-command-lifecycle.md)
- [Common Command Format](concepts/common-command-format.md) (64-byte SQE, CDW0, NSID, metadata and data pointers)
- [Data Pointer Layouts](concepts/data-pointer-layouts.md) (PRP entries/lists and SGL segment boundary)
- [Completion Queue Entry and Status](concepts/completion-queue-entry-and-status.md) (correlation, SQ consumption, status families, retry hints, Phase Tag lifecycle)
- [Feature Values and Scope](concepts/feature-values-and-scope.md) (default/current/saved values, reset restoration, scope selectors)
- [Common Controller Features](concepts/common-controller-features.md) (arbitration, power, temperature thresholds, hysteresis, and volatile cache)
- [Security Protocol Exchange](concepts/security-protocol-exchange.md) (Security Send/Receive envelope, discovery, and NVMe RPMB selection)
- [Identity, Name, and List Formats](concepts/identity-name-and-list-formats.md) (EUI64/NGUID/UUID, NQN, ordered lists, UTF-8 processing)
- [Host Metadata and Management Addresses](concepts/host-metadata-and-management-addresses.md) (agent URIs and typed controller/namespace annotations)
- [Host Memory Buffer](concepts/host-memory-buffer.md) (host-memory ownership, descriptor list, disable barrier, and power-state restriction)
- [Identify Command Model](concepts/identify-command-model.md) (CNS-selected 4 KiB structures, dependency-gated controller capabilities, runtime limits, and aggregate I/O envelope)
- [Power State Descriptors](concepts/power-state-descriptors.md) (operational state, latency, relative performance, power, and power-loss timing)
- Capsule and in-capsule data (see [Transport Models](concepts/transport-models.md))

## Storage organization

- [Storage Resource Hierarchy](concepts/storage-resource-hierarchy.md) (NVM Set and Reclaim Group organizations)
- [NVM Sets and Endurance Groups](concepts/nvm-sets-and-endurance-groups.md)
- [NVM Capacity Model](concepts/nvm-capacity-model.md) (capacity allocation, Media Units, Channels, and organization tradeoffs)
- [Flexible Data Placement Configurations](concepts/flexible-data-placement-configurations.md) (configuration catalog, Placement Identifier split, runtime usage, statistics, and events)
- [Domains and Divisions](concepts/domains-and-divisions.md) (fault boundaries, identifiers, ANA/reservation recovery)
- [Exported and Underlying NVM Resources](concepts/exported-and-underlying-resources.md) (directories, creation/clear, namespace association, and deletion gates)
- Domain (see [Domains and Divisions](concepts/domains-and-divisions.md))
- Endurance Group
- NVM Set
- Reclaim Group
- Reclaim Unit
- Media Unit

## Control planes

- [Controller Properties](concepts/controller-properties.md) (access contract, capability/configuration handshake, version and interrupt masks)
- [Controller Memory Windows](concepts/controller-memory-windows.md) (CMB and PMR address, capability, readiness, and health)
- [Boot Partitions](concepts/boot-partitions.md)
- [Controller Support Requirements](concepts/controller-support-requirements.md) (commands, log pages, and Features by controller type)
- [Admin Command Model](concepts/admin-command-model.md) (opcode taxonomy, transport partition, Format/Sanitize gates)
- [Command Abort Semantics](concepts/command-abort-semantics.md) (immediate/deferred abort and effects barrier)
- [Common I/O Control Commands](concepts/common-io-control-commands.md) (Cancel selection, Flush persistence, and I/O management exchange)
- [Asynchronous Event Reporting](concepts/asynchronous-event-reporting.md) (AER lifecycle, masking, log clearing, event families)
- [Capacity Management Operations](concepts/capacity-management-operations.md) (fixed/variable configuration, Endurance Group and NVM Set lifecycle)
- [Controller Data Queues](concepts/controller-data-queues.md) (host-memory queue lifecycle and User Data Migration Queue)
- [Device Self-test](concepts/device-self-test.md) (scope, start/abort transitions, log interaction)
- [Directive Exchange](concepts/directive-exchange.md) (shared Directive Send/Receive envelope)
- [Controller Data Queues](concepts/controller-data-queues.md) (host-memory backing, create/delete lifecycle, User Data Migration Queue)
- [Device Self-test](concepts/device-self-test.md) (namespace scope, actions, in-progress state transitions)
- [Directive Exchange](concepts/directive-exchange.md) (Receive/Send envelope and type-dependent operations)
- [Log Page Retrieval](concepts/log-page-retrieval.md) (LID selection, offsets, scope, and support discovery)
- [Reservation Notification Log](concepts/reservation-notification-log.md) (destructive-read queue, sequence gaps, notification type, and namespace)
- [Namespace Reservation Lifecycle](concepts/namespace-reservation-lifecycle.md) (registration, acquisition, preemption, release, persistence, and status reporting)
- [Sanitize Operation Status](concepts/sanitize-operation-status.md) (persistent progress, outcome, time estimates, state detail, erase evidence, and verification cancellation)
- [Sanitize Operation Lifecycle](concepts/sanitize-operation-lifecycle.md) (background erase start, media verification, option gates, and serialization)
- [Resource and Management Inventory Logs](concepts/resource-and-management-inventory-logs.md) (rotational-media lifetime data and management URIs)
- [Error and Health Logs](concepts/error-and-health-logs.md) (recent errors, command correlation, SMART warnings and condition)
- [Command Effects and Support](concepts/command-effects-and-support.md) (opcode support, affected scope, serialization, and capability changes)
- [Command and Feature Lockdown](concepts/command-and-feature-lockdown.md) (lockable and currently prohibited operations by identifier namespace and submission path)
- [Controller Migration](concepts/controller-migration.md) (state-format selection, snapshot consistency, suspend/resume/restore lifecycle)
- [Migration Change Tracking](concepts/migration-change-tracking.md) (UDMQ logging and destructive host-memory changed-range reporting)
- [Telemetry Capture Logs](concepts/telemetry-capture-logs.md) (host/controller capture, block areas, retention, availability, and generation)
- [Predictable Latency Mode](concepts/predictable-latency-mode.md) (NVM Set windows, budgets, estimates, and event aggregation)
- [Persistent Event Log](concepts/persistent-event-log.md) (persistent history, frozen reporting context, generation validation, reset and hardware-error evidence)
- Fabrics Command Set
- I/O Command Set
- [Feature Values and Scope](concepts/feature-values-and-scope.md)
- [Log Page Retrieval](concepts/log-page-retrieval.md)
- [Asynchronous Event Reporting](concepts/asynchronous-event-reporting.md)
- [Fabrics Discovery and Authentication](concepts/fabrics-discovery-and-authentication.md)
- [Identify Command Model](concepts/identify-command-model.md)

## Lifecycle and resilience

- [Controller Enable, Shutdown, and Reset](concepts/controller-enable-shutdown-reset.md) (controller/subsystem shutdown and subsystem/controller reset scope)
- [Controller Initialization](concepts/controller-initialization.md) (memory-based and Fabrics bootstrap sequences)
- [Controller Ready Modes and Timeouts](concepts/controller-ready-modes.md) (media-independent readiness, deadlines, timeout failure)
- [Keep Alive](concepts/keep-alive.md) (configuration, activation, command/traffic watchdog modes, timeout cleanup)
- [Firmware Update Lifecycle](concepts/firmware-update-lifecycle.md) (download, commit, activation, reset, and serialization)
- [Format NVM Lifecycle](concepts/format-nvm-lifecycle.md) (format scope, secure erase, concurrency, and rejection)
- Power States
- [Sanitize Operation Status](concepts/sanitize-operation-status.md)
- [Error and Health Logs](concepts/error-and-health-logs.md)
