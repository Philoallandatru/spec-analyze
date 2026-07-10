# Identify Command Model

Identify is the Admin read operation that selects a 4,096-byte description of an NVM subsystem, Domain, controller, namespace, or related inventory. Its `CNS` selector chooses the returned structure; `NSID`, `CNTID`, `CSI`, `CNSSID`, and optional UUID index qualify only the operations that define them. [PDF pp. 317-319](../_source/pages/page-317.md)

## Mental model

```text
Identify command
    |
    +-- CNS ---------> which structure or list?
    +-- NSID --------> which namespace?       (CNS-dependent)
    +-- CNTID -------> which controller?      (CNS-dependent)
    +-- CSI ----------> which I/O command set? (CNS-dependent)
    +-- CNSSID -------> selector defined by that CNS
    `-- UIDX ---------> UUID-qualified view, when supported
                         |
                         v
                  4,096-byte result
```

This explanatory selector tree condenses Figures 306-310 without reproducing the full CNS value table. [PDF pp. 317-319](../_source/pages/page-317.md)

## Contract and invariants

| Concern | Contract |
|---|---|
| Result size | Every Identify operation returns a 4,096-byte data structure; unused entries/space are zero-filled. |
| Data pointer | With PRPs, the data pointer is not a PRP List pointer because the buffer crosses at most one page boundary. |
| Unused selectors | An unused `CNTID` is cleared by the host and ignored by the controller; unused `CSI` is cleared to zero. |
| Unsupported selection | Unsupported `CNS` fails with `Invalid Field in Command`; an incompatible namespace command-set association fails with `Invalid I/O Command Set`. |
| Transport partition | Common structures, memory-based-only structures, and message-based-only structures are defined in separate Identify subsections. |

These rules summarize the rendered command format and selection behavior. [PDF pp. 317-319](../_source/pages/page-317.md)

## Selector families

| Family | Representative results | Selector pattern |
|---|---|---|
| Active namespace management | controller data, active NSID lists, namespace descriptors, NVM Set list | `CNS 00h-0Ah`; namespace and CSI selectors vary by result |
| Controller/namespace management | allocated NSID lists, controller lists, primary/secondary controller data, UUID list | `CNS 10h-17h`; support is capability-dependent |
| Resource and command-set inventory | Domain, Endurance Group, I/O Command Set, underlying namespace/ports | `CNS 18h-1Eh`; `CNS 1Dh` is prohibited when any controller uses a memory-based transport |
| Allocated namespace and controller-state views | allocated command-set-specific namespace data, independent namespace data, controller state formats | `CNS 1Ah-20h`; support remains capability- and revision-dependent |

The full `CNS` matrix is intentionally not duplicated here: it is an exact cross-page table with mandatory/optional conditions and per-selector applicability. The `CSI` namespace contains standardized command sets at `00h`-`04h`, vendor-specific values at `30h`-`3Fh`, and reserved ranges elsewhere. [PDF pp. 319-320](../_source/pages/page-319.md)

## Identify Controller capability profile

The I/O Command Set Independent Identify Controller structure describes the controller processing the command. Its leading region combines subsystem identity, controller identity, transfer limits, asynchronous-event support, and capability flags rather than defining one capability per structure. [PDF pp. 321-326](../_source/pages/page-321.md)

```text
Identify Controller (CNS 01h), leading capability region

bytes 0..75    identity + active firmware + arbitration recommendation
      76       topology / sharing capability (CMIC)
      77       host-memory transfer ceiling (MDTS)
      78..91   controller identity, version, Runtime D3 latencies
      92..95   optional asynchronous-event support (OAES)
      96..99   cross-cutting controller attributes (CTRATT)
     100..101  supported read-recovery levels (RRLS)
```

This explanatory byte-range map is reconstructed from the opening portion of Figure 312; it is not a replacement for the normative field layout. [PDF pp. 321-326](../_source/pages/page-321.md)

| Capability cluster | Key contract |
|---|---|
| Identity and topology | `SN` and `MN` identify a non-Discovery subsystem; `CNTLID` is unique within that subsystem; `CMIC` reports multiple ports/controllers, SR-IOV function association, and ANA reporting support. |
| Transfer limit | Non-zero `MDTS` is `2^n` minimum-memory-pages. Exceeding it is an invalid field; `CTRATT.MEM` decides whether interleaved metadata counts, and destination Bit Bucket lengths count. |
| Event advertisement | `OAES` advertises optional event families, but host software must enable a family before the controller sends it. |
| Cross-capability constraints | `CTRATT.FDPS` and `CTRATT.FCM` are mutually exclusive; namespace granularity requires Namespace Management; the traffic-based Keep Alive bit selects that watchdog mode. |
| Recovery levels | If read-recovery levels are supported, both Fast Fail level 15 and Default level 4 are mandatory supported bits. |

These clusters preserve the capability dependencies while avoiding a field-by-field page split. [PDF pp. 321-326](../_source/pages/page-321.md)

## Capability advertisement and dependency gates

The middle of the I/O Command Set Independent Identify Controller structure is a capability directory: it advertises controller type and management surfaces, then gates optional Admin commands, logs, firmware behavior, power/thermal features, host memory, capacity reporting, and protected memory. A set bit does not replace the detailed command or Feature contract; it tells the host which contract may be used. [PDF pp. 327-334](../_source/pages/page-327.md)

```text
Identify Controller capability directory (Figure 312, bytes 102..315)

102..134   platform + identity + reachability
     |
253..255   subsystem / management-interface presence
     |
256..262   Admin commands, AER, firmware, log attributes
     |
263..271   power states, temperature, activation time
     |
272..311   host-memory request + accessible/unallocated capacity
     `---- 312..315 protected-memory capabilities (RPMB)
```

This explanatory range map was reconstructed from the rendered continuation of Figure 312; reserved gaps are omitted and the normative byte/bit layout remains in the source table. [PDF pp. 327-334](../_source/pages/page-327.md)

| Capability cluster | Dependency or invariant |
|---|---|
| Controller and FRU identity | Revision 1.4-or-later controllers report a non-zero controller type; a non-zero `FGUID` is stable for the FRU and identical across its controllers. |
| Optional Admin surface | `OACS` gates command families. Lockdown support is subsystem-consistent, Host Managed Live Migration is not advertised by secondary controllers, and a non-zero `ACL`/`AERL` value is zero-based. |
| Firmware and logs | `FRMW` reports Domain firmware slots and activation/overlap behavior; `LPA` separately gates telemetry areas, Persistent Event, extended Get Log Page fields, effects logs, and per-namespace SMART. |
| Power and thermal | `NPSS` is zero-based with at least power state 0; `APSTA` gates autonomous transitions; warning and critical temperature thresholds are reported independently. |
| Host memory and migration | `HMPRE` non-zero advertises Host Memory Buffer support and must be at least `HMMIN`; if any subsystem controller advertises Host Managed Live Migration, `HMPRE` is zero. |
| Capacity and protected memory | `TNVMCAP`/`UNVMCAP` are required with Namespace or Capacity Management support. Non-zero RPMB count requires Security Send/Receive, and all reported RPMB targets share the encoded capabilities. |

The table groups dependencies that are easy to miss when reading Figure 312 field-by-field. [PDF pp. 328-334](../_source/pages/page-328.md)

## Operational and I/O capability envelope

The next portion of the same structure connects advertised features to their operating limits and then defines the controller's common I/O envelope. Self-test, firmware, thermal, Sanitize, ANA, Key Per I/O, and command-quiesce attributes are controller or subsystem capability gates; queue entry sizes, outstanding-command guidance, namespace range, optional-command support, and fused-operation support constrain the I/O interface exposed after discovery. [PDF pp. 335-342](../_source/pages/page-335.md)

```text
Identify Controller, bytes 316..523

316..387  operational capability gates
   |       self-test | firmware | thermal | sanitize | ANA
   |       host memory | storage hierarchy | Key Per I/O | quiesce
   v
512..519  I/O queue + namespace envelope
   |       SQ/CQ entry sizes | per-queue MAXCMD | maximum valid NSID
   v
520..523  aggregate optional-command + fused-operation support
```

This explanatory range map is reconstructed from Figure 312. Reserved gaps and exact bit positions are intentionally omitted; the normative layout remains in the source table. [PDF pp. 335-342](../_source/pages/page-335.md)

| Capability cluster | Dependency or invariant |
|---|---|
| Self-test and firmware | `DSTO.HIRS` requires Device Self-test support; `DSTO.SDSO` chooses subsystem-wide single-operation versus one operation per controller. `FWUG` describes download granularity/alignment, while revision 2.1 controllers supporting activation without reset report a non-zero `MPTFAWR`. |
| Thermal and Sanitize | Host-controlled thermal management requires Set/Get Features support for FID `10h`, with requested thresholds bounded by `MNTMT` and `MXTMT`. A supported Sanitize command requires non-zero `SANICAP`; its operation bits and no-deallocate policy gate the detailed Sanitize state machine. |
| ANA topology | ANA support requires non-zero transition time, maximum ANA Group Identifier, and controller group count; `NANAGRPID` does not exceed `ANAGRPMAX`. `ANACAP` separately advertises reportable states and whether namespace management may assign or change group IDs while attached. |
| Runtime bounds | `KPIOC` distinguishes subsystem-wide from per-namespace Key Per I/O enablement. `CQT` is the worst-case interval to satisfy Immediate Abort after Keep Alive timeout or communication loss; zero means unreported, while an implementation needing no time should report 1 ms. |
| Queue and namespace envelope | NVM I/O SQ entries have a required 64-byte minimum and CQ entries a required 16-byte minimum. `MAXCMD` is per queue and may exceed SQ size; it is mandatory for Fabrics and optional for PCIe. `NN` is the maximum valid NSID, not the number of supported namespaces. |
| Optional I/O behavior | `ONCS` aggregates command support across supported or inherited I/O Command Sets; the Commands Supported and Effects log resolves support for one specific command set. `FUSES` separately advertises Compare-and-Write, with Compare first. |

These rows summarize cross-field contracts verified against the rendered table; they do not reproduce the exact bit-field encodings. [PDF pp. 335-342](../_source/pages/page-335.md)

## Data-path, migration, and Fabrics envelope

The final controller region extends the I/O envelope from command availability to data integrity, cache behavior, namespace protection, transfer descriptors, attachment limits, migration tracking, and Fabrics capsule geometry. [PDF pp. 343-350](../_source/pages/page-343.md)

```text
Identify Controller, bytes 524..2047

524..535   Format/cache/atomic/write-protect/Copy attributes
536..539   SGL capability matrix
540..587   namespace attachments + refresh + change tracking/CDQ limits
768..1023  subsystem NQN
1792..1806 Fabrics capsule/connection/Discovery attributes
2048..3071 power-state descriptors begin
```

This explanatory range map omits reserved gaps and exact bit positions. It was checked against the rendered continuation of Figure 312. [PDF pp. 343-350](../_source/pages/page-343.md)

| Capability cluster | Dependency or invariant |
|---|---|
| Format and cache | `FNA` separates broadcast acceptance from format and secure-erase scope; `VWC` separately reports cache presence and whether broadcast Flush is legal. |
| Atomicity and protection | `AWUN`, `AWUPF`, and `ACWU` apply only to logical-block command sets. Namespace write protection requires the base No-Write-Protect/Write-Protect pair before power-cycle or permanent states may be advertised. |
| Copy and SGL | `CDFS` enumerates supported Copy descriptor formats. `SGLS` gates base SGL support and then independently advertises keyed, bit-bucket, metadata-pointer, offset, transport, length, and alignment behaviors. |
| Namespace and refresh limits | `MNAN` is a supported-namespace count ceiling distinct from maximum valid NSID `NN`; Domain/controller attachment limits may be unreported. Refresh interval/time fields are zero unless Host-Initiated Refresh is supported. |
| Change tracking and migration | Memory-range descriptor/granularity limits depend on host-memory tracking support; User Data Migration Queue maxima depend on user-data tracking support. Controller and subsystem CDQ range limits are reported separately. |
| Fabrics | Capsule/response sizes use 16-byte units; `ICDOFF` begins after the 64-byte SQE. Fabrics attributes report NVM Set selection, controller model, capsule SGL count, optional Disconnect, and Discovery controller type. |

These dependencies prevent hosts from treating non-zero limits as independent features. [PDF pp. 343-350](../_source/pages/page-343.md)

The NVM subsystem NQN is mandatory from revision 1.2.1 onward. Discovery controllers return either their unique Discovery Service NQN or the well-known discovery NQN. Power-state descriptors follow the Fabrics region and define up to 32 states, beginning with mandatory state 0. [PDF pp. 349-350](../_source/pages/page-349.md)

## Namespace and command-set views

```text
CNS 02h  active NSIDs after cursor
CNS 03h  durable namespace identifier descriptors
CNS 04h  NVM Set attributes after NVMSETID cursor
CNS 05h  CSI-specific namespace data
CNS 06h  CSI-specific controller data
CNS 07h  CSI-filtered active NSIDs after cursor
CNS 08h  command-set-independent namespace data
```

The list selectors are paginated rather than snapshot-all operations: Active Namespace lists return up to 1,024 increasing NSIDs greater than the input NSID, while the NVM Set list returns up to 31 ordered 128-byte entries starting at or after a selected NVM Set Identifier. [PDF pp. 353-356](../_source/pages/page-353.md)

Namespace Identification Descriptors are variable-length and end when `NIDL=0`. A list contains no duplicate type, includes at least one durable identity (`EUI64`, `NGUID`, or UUID), and includes the namespace's Command Set Identifier when multiple I/O Command Sets are enabled. Identity descriptors survive reset and format for the namespace lifetime. [PDF pp. 353-354](../_source/pages/page-353.md)

CSI-specific Namespace/Controller views fail for unsupported command sets, while a supported command set with no defined controller structure returns zeroes. Inactive namespaces return a zero-filled Namespace structure. With `NSID=FFFFFFFFh`, capability-only views may be returned for namespace creation/formatting: reported/common capabilities remain and non-capability fields are zero. [PDF pp. 355-357](../_source/pages/page-355.md)

The command-set-independent Namespace view begins with per-namespace cache override, rotational-media and durable-ID-reuse attributes, private/shared/dispersed access, and reservation-type support. Its “Reported” column distinguishes fields valid in the broadcast capability template from fields meaningful only for one namespace. [PDF pp. 357-359](../_source/pages/page-357.md)

## Namespace runtime state and resource membership

The independent Namespace view continues from static capabilities into current state and storage-resource membership. Format progress is meaningful only when its support bit is set; namespace status separately reports readiness and whether I/O performance degradation is reported or active. Key Per I/O has distinct supported and enabled bits, and its maximum key tag is valid only when the capability is enabled. [PDF pp. 359-360](../_source/pages/page-359.md)

```text
namespace capability + current state
        |
        +-- FPI ------ format-progress support / remaining percentage
        +-- NSTAT ---- ready + I/O performance impact
        +-- KPIOS ---- Key Per I/O supported / enabled -> MAXKT
        `-- membership
             +-- ANAGRPID / RGRPID  access and reachability groups
             `-- NVMSETID / ENDGID storage allocation hierarchy
```

This relationship diagram groups the fields by host decision rather than byte position. Group identifiers are cleared when their reporting mechanism is unsupported; NVM Set and Endurance Group identifiers are also cleared for namespaces without formatted storage. A changed ANA or Reachability Group identifier produces its corresponding change notice only when that notice mechanism is supported and enabled. [PDF pp. 359-360](../_source/pages/page-359.md)

## Allocated, topology, and resource directories

| `CNS` | Result | Cursor and boundary contract |
|---|---|---|
| `10h` | allocated NSID list | up to 1,024 increasing NSIDs greater than the input NSID |
| `11h` | allocated namespace data | zero-filled for an unallocated NSID; invalid command set if the namespace has no logical blocks |
| `12h` / `13h` | attached-controller / I/O-controller list | up to 2,047 controller IDs greater than or equal to `CNTID` |
| `16h` / `17h` | namespace granularity / UUID list | command-set-specific granularity; UUID entries terminate at the first zero entry |
| `18h` / `19h` | Domain / Endurance Group list | ordered resource entries starting at or after the supplied resource identifier |
| `1Ah` / `1Bh` | CSI-filtered allocated list / namespace data | allocated namespace view constrained by the selected I/O Command Set |
| `1Ch` | I/O Command Set combinations | up to 512 vectors; the selected profile enables only sets present in its vector |
| `1Fh` / `20h` | independent allocated namespace / controller-state formats | allocated independent view; migration format-version and vendor-UUID directories |

These selectors are related directories, not interchangeable snapshots. UUID entries carry an association of none, PCI Vendor ID, or PCI Subsystem Vendor ID, and a zero entry ends the list. Domain entries report total, unallocated, and optional maximum single-Endurance-Group capacity; an Endurance Group cursor greater than `ENDGIDMAX` succeeds with an empty list. [PDF pp. 361-364](../_source/pages/page-361.md)

An **I/O Command Set Combination** is one vector of simultaneously usable command sets. Combination zero has index zero, later supported combinations occupy successive indices, and the I/O Command Set Profile Feature selects which vector is active; unselected command sets are treated as unsupported. The standardized vector bits represent NVM, Key Value, Zoned Namespace, Subsystem Local Memory, and Computational Programs command sets. [PDF pp. 364-365](../_source/pages/page-364.md)

The Supported Controller State Formats result is a two-part migration directory: one indexed list names supported NVMe Controller State structure versions and a second indexed list names vendor-specific state formats by UUID. Migration Receive and Migration Send use those indices when getting controller state. The vendor-specific payload format itself remains outside this specification. [PDF pp. 365-366](../_source/pages/page-365.md)

## Relationships

- [Identity, Name, and List Formats](identity-name-and-list-formats.md) defines the identifiers carried by several Identify results. [PDF p. 318](../_source/pages/page-318.md)
- [Namespace Identifiers](namespace-identifiers.md) distinguishes active and allocated NSID views selected by different `CNS` values. [PDF p. 319](../_source/pages/page-319.md)
- [Command Sets](command-sets.md) defines the I/O Command Set association that controls whether CSI-qualified Identify operations are valid. [PDF pp. 318-319](../_source/pages/page-318.md)
- [Firmware Update Lifecycle](firmware-update-lifecycle.md), [Log Page Retrieval](log-page-retrieval.md), and [NVM Capacity Model](nvm-capacity-model.md) define the behaviors merely advertised by `FRMW`, `LPA`, and the capacity fields. [PDF pp. 331-334](../_source/pages/page-331.md)
- [Device Self-test](device-self-test.md), [Sanitize Operation Status](sanitize-operation-status.md), and [Namespace Access Models](namespace-access-models.md) define the operations and state views advertised by `DSTO`, `SANICAP`, and the ANA fields. [PDF pp. 335-338](../_source/pages/page-335.md)
- [Queue Pair](queue-pair.md), [Command Sets](command-sets.md), and [Commands Supported and Effects](command-effects-and-support.md) explain the queue geometry and per-command-set support behind `SQES`, `CQES`, `MAXCMD`, and aggregate `ONCS`. [PDF pp. 340-342](../_source/pages/page-340.md)
- [Data Pointer Layouts](data-pointer-layouts.md), [Controller Data Queues](controller-data-queues.md), and [Fabrics Discovery and Authentication](fabrics-discovery-and-authentication.md) define the SGL, migration/CDQ, and Fabrics behaviors advertised in the final controller region. [PDF pp. 346-350](../_source/pages/page-346.md)
- [Namespace Identifiers](namespace-identifiers.md), [NVM Sets and Endurance Groups](nvm-sets-and-endurance-groups.md), and [Power State Descriptors](power-state-descriptors.md) define the paginated resource views and power policy structures selected after Identify Controller. [PDF pp. 350-358](../_source/pages/page-350.md)

## Edge cases

- `CNTID=0h` is a valid controller identifier even though hosts clear an unused `CNTID` to zero; whether it is meaningful is determined by `CNS`, not its numeric value. [PDF p. 318](../_source/pages/page-318.md)
- Hosts talking to revision 1.0 or 1.1 controllers should issue only CNS values defined by that revision; other results are indeterminate. [PDF p. 318](../_source/pages/page-318.md)
- `MDTS=0h` means that this field imposes no maximum, not that transfers are forbidden; commands without host-accessible-memory transfer are outside this limit. [PDF p. 322](../_source/pages/page-322.md)
- Discovery controllers reserve or conditionally expose several identity/capability fields, so the I/O/Admin support columns in Figure 312 cannot be assumed for Discovery. [PDF pp. 321-326](../_source/pages/page-321.md)
- A zero in a capability field is not uniformly “unsupported”: `NSSL=0` means latency is unreported, `HMMIN=0` requests any feasible amount up to `HMPRE`, and zero-based limits/counts encode one supported entry where their enclosing capability is present. [PDF pp. 327, 330-334](../_source/pages/page-327.md)
- `MAXCMD` is not a Submission Queue depth limit, `NN` is not a namespace count, and aggregate `ONCS` is not sufficient to prove support in one selected I/O Command Set. [PDF pp. 340-342](../_source/pages/page-340.md)
- A set `FNA.FNVMBS` means broadcast Format is *not* supported; it also forces the separate format and secure-erase scope bits clear. [PDF p. 343](../_source/pages/page-343.md)
- `SGLS=00b` disables SGL use regardless of individual descriptor bits, and `MNAN=0` does not mean zero namespaces—it falls back to a count no greater than `NN`. [PDF pp. 346-347](../_source/pages/page-346.md)
- The allocated-list cursors are not uniform: NSID lists use strictly greater-than, controller lists use greater-than-or-equal, and Domain/Endurance Group lists start at or after their resource cursor. [PDF pp. 361-364](../_source/pages/page-361.md)

## Evidence

- [Identify overview and data pointer, PDF p. 317](../_source/pages/page-317.md)
- [Command dwords and selection/error rules, PDF p. 318](../_source/pages/page-318.md)
- [CNS selector matrix start, PDF p. 319](../_source/pages/page-319.md)
- [CNS selector matrix continuation, adjacent PDF p. 320](../_source/pages/page-320.md)
- [Identify Controller identity and topology fields, PDF p. 321](../_source/pages/page-321.md)
- [Transfer limit and optional-event support, PDF pp. 322-323](../_source/pages/page-322.md)
- [Controller attributes and read-recovery levels, PDF pp. 324-326](../_source/pages/page-324.md)
- [Boot, shutdown, power-loss, identity, retry, and reachability capabilities, PDF pp. 327-328](../_source/pages/page-327.md)
- [Management and optional Admin command capabilities, PDF pp. 329-330](../_source/pages/page-329.md)
- [Asynchronous-event, firmware, and log attributes, PDF pp. 331-332](../_source/pages/page-331.md)
- [Power, thermal, host-memory, capacity, and RPMB capabilities, PDF pp. 333-334](../_source/pages/page-333.md)
- [Self-test, firmware, thermal, Sanitize, host-memory, and ANA capabilities, PDF pp. 335-338](../_source/pages/page-335.md)
- [Key Per I/O, firmware activation, capacity, hysteresis, and quiesce limits, PDF p. 339](../_source/pages/page-339.md)
- [I/O queue, namespace, optional-command, and fused-operation attributes, PDF pp. 340-342](../_source/pages/page-340.md)
- [Format, cache, atomicity, write-protection, Copy, and SGL attributes, PDF pp. 343-346](../_source/pages/page-343.md)
- [Namespace/attachment, refresh, tracking, migration, and CDQ limits, PDF pp. 347-348](../_source/pages/page-347.md)
- [Subsystem NQN, Fabrics attributes, and power-state descriptor boundary, PDF pp. 349-350](../_source/pages/page-349.md)
- [Power-state descriptor completion and CNS 02h–04h resource lists, PDF pp. 351-355](../_source/pages/page-351.md)
- [CSI-specific and independent Namespace/Controller views, PDF pp. 355-358](../_source/pages/page-355.md)
- [Independent Namespace runtime state and resource membership, PDF pp. 359-360](../_source/pages/page-359.md)
- [Allocated, controller, UUID, Domain, and Endurance Group directories, PDF pp. 361-363](../_source/pages/page-361.md)
- [CSI-allocated views and I/O Command Set combinations, PDF pp. 364-365](../_source/pages/page-364.md)
- [Supported Controller State Formats and memory-transport boundary, PDF p. 366](../_source/pages/page-366.md)
