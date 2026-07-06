# Exported and Underlying NVM Resources

Exporting separates the physical or virtual resources that provide storage from the logical NVM subsystem presented for remote access. An Exported NVM Subsystem may contain Exported Namespaces, controllers, Exported Ports, and optionally an Allowed Host List. [PDF pp. 30-31](../_source/pages/page-030.md)

## Mental model

```text
Underlying NVM Subsystem(s)
|-- Underlying Namespace(s) -- associate --> Exported Namespace(s)
`-- Underlying Port(s) ------ Ports List --> Exported Port(s)
                                             |
                                             v
                                  Exported NVM Subsystem
                                  |-- controller(s) [0..*]
                                  |-- Exported Namespace(s) [0..*]
                                  |-- Exported Port(s) [0..*]
                                  `-- Allowed Host List [optional]
```

This is an explanatory ownership and mapping view; the exact management data structures are defined later in the specification. [PDF pp. 30-31, 35-36](../_source/pages/page-030.md) [PDF p. 35](../_source/pages/page-035.md) [PDF p. 36](../_source/pages/page-036.md)

## Contract and invariants

| Concept | Role |
|---|---|
| Exported NVM Subsystem | Logical subsystem that presents underlying resources remotely |
| Exported Namespace | Namespace belonging to an Exported NVM Subsystem |
| Exported Port | Fabrics-transport endpoint identified by an Exported Port ID |
| Underlying Namespace | Namespace eligible to be associated with an exported subsystem |
| Underlying Port | Port attaching an underlying subsystem to a transport |

Exported resources are created for remote access; they are not synonyms for the underlying physical resources. [PDF pp. 30-31, 35-36](../_source/pages/page-030.md) [PDF p. 35](../_source/pages/page-035.md) [PDF p. 36](../_source/pages/page-036.md)

## Identify directories

Message-based controllers expose two generation-counted directories. `CNS=1Dh` lists all Underlying Namespaces accessible through a physical or virtual function; each entry binds an Underlying NVM Subsystem NQN, its NSID, and the associated subsystem-unique Controller ID. `CNS=1Eh` lists Underlying Ports usable to export Fabrics subsystems. [PDF pp. 368-370](../_source/pages/page-368.md)

| Directory | Entry identity | Connection data |
|---|---|---|
| Underlying Namespace List | subsystem NQN + NSID + Controller ID | identifies storage eligible for export |
| Ports List | Underlying Port ID | transport address, subtype, type, address family, and requirements |

Both generation counters clear on reset of the Underlying NVM Subsystem, increment once per directory change, and wrap from all ones to zero. The Ports transport address and transport-specific subtype are interpreted by the applicable NVMe Transport specification. [PDF pp. 368-370](../_source/pages/page-368.md)

## Exported subsystem creation and clearing

```text
Create (restricted or unrestricted)
          |
          v
[Exported subsystem: empty Allowed Host List, no Exported Namespaces]
          |
          +-- later associate namespaces, ports, and allowed hosts
          |
disconnect every host
          |
          v
Clear all exported configuration --X--> underlying namespaces/subsystems
```

This explanatory lifecycle distinguishes logical exported configuration from the underlying resources that survive a clear operation. [PDF pp. 448-449](../_source/pages/page-448.md)

Create selects the initial access mode: unrestricted accepts any host, while restricted access consults the subsystem's Allowed Host List. A successful creation starts with an empty Allowed Host List and no linked Exported Namespaces, and returns the new `SUBNQN` in the command data buffer. The command is prohibited on an Exported NVM Subsystem. [PDF pp. 448-449](../_source/pages/page-448.md)

Clear Exported NVM Resource Configuration deletes all exported configuration but does not affect Underlying Namespaces or Underlying NVM Subsystems. Every host must first be disconnected from every Exported NVM Subsystem or the command fails with Command Sequence Error; the command is likewise prohibited on an Exported NVM Subsystem. [PDF p. 448](../_source/pages/page-448.md)

## Exported namespace association lifecycle

```text
[valid unassociated ENSID] + [attached active underlying namespace]
                              |
                              v
                    Associate Namespace
                              |
             [allocated Exported Namespace]
             same user data as underlying namespace
                              |
                  Namespace Attachment (separate)
                              |
               detach from every controller
                              |
                              v
                   Disassociate Namespace
                              |
                    [ENSID is deleted]
```

This explanatory lifecycle distinguishes storage association from controller attachment and preserves the detach-before-delete gate. [PDF pp. 460-462](../_source/pages/page-460.md)

Association identifies the exported subsystem by NQN and the underlying namespace by the tuple of Underlying Subsystem NQN, Controller ID, and active NSID. The underlying namespace must be allocated and attached to that controller. Successful association allocates the Exported Namespace in the exported subsystem, but does not attach it to a controller; exported and underlying operations address the same user data. [PDF pp. 460-461](../_source/pages/page-460.md)

Disassociation is allowed only after the Exported Namespace is detached from every controller. Success removes the exported-subsystem association, deletes the ENSID, and reports Namespace Attribute Changed; association success reports the same asynchronous event. [PDF pp. 461-462](../_source/pages/page-461.md)

## Exported subsystem deletion gate

| Remaining dependency | Delete result |
|---|---|
| active controller | Command Sequence Error |
| associated Exported Namespace | Command Sequence Error |
| existing Exported Port | Command Sequence Error |
| none of the above | Exported NVM Subsystem is deleted |

The Manage Exported Namespace and Manage Exported NVM Subsystem commands are themselves unsupported by Exported NVM Subsystems. [PDF pp. 460, 462-463](../_source/pages/page-460.md)

## Exported subsystem access control

```text
[unrestricted: every host allowed]
                |
                | Change Access Mode: restricted
                v
[restricted: Allowed Host List only]
     | Grant: add host/subsystem/underlying-port combinations
     ` Revoke: remove combinations and disconnect affected hosts
```

This explanatory state model preserves the immediate disconnect effect of restriction and revocation. [PDF pp. 463-466](../_source/pages/page-463.md)

Changing to restricted access disconnects any connected host absent from the subsystem's Allowed Host List; unrestricted mode enables all hosts. Grant and Revoke operate over non-empty host and exported-subsystem lists, associating each target subsystem with an Underlying Port ID. A failing list entry is localized by byte offset in the Error Information Log. [PDF pp. 464-466](../_source/pages/page-464.md)

Each host entry combines a 128-bit Host Identifier with `HOSTNQN`; each subsystem entry combines `SUBNQN` with the Underlying Port ID through which access is granted or revoked. Revocation disconnects affected connected hosts on the selected exported ports. [PDF pp. 465-466](../_source/pages/page-465.md)

## Exported port lifecycle

Creating an Exported Port binds an exported subsystem to one Underlying Port and transport service identifier. The host may supply a unique Exported Port ID or request controller generation; successful completion returns the chosen ID. [PDF pp. 467-468](../_source/pages/page-467.md)

Deletion names both `SUBNQN` and the associated Exported Port ID. The port should be unused: deleting an in-use port terminates all host/controller associations through it, but outstanding-command and underlying fabric-resource behavior is undefined. [PDF pp. 468-469](../_source/pages/page-468.md)

## Relationships

Fabric Zoning defines access-control configurations between hosts and NVM subsystems. It is distinct from an Allowed Host List, which is optional state of an Exported NVM Subsystem. [PDF p. 31](../_source/pages/page-031.md)

## Evidence

- [Exported resource definitions, PDF pp. 30-31](../_source/pages/page-030.md)
- [Underlying namespace definitions, PDF p. 35](../_source/pages/page-035.md)
- [Underlying subsystem and port definitions, PDF p. 36](../_source/pages/page-036.md)
- [Underlying Namespace List header and entries, PDF pp. 368-369](../_source/pages/page-368.md)
- [Ports List and Fabrics transport entry, PDF pp. 369-370](../_source/pages/page-369.md)
- [Clear and Create Exported NVM Subsystem commands, PDF pp. 448-449](../_source/pages/page-448.md)
- [Exported namespace association and disassociation, PDF pp. 460-462](../_source/pages/page-460.md)
- [Exported subsystem management and deletion gates, PDF pp. 462-463](../_source/pages/page-462.md)
- [Access mode and Allowed Host List modification, PDF pp. 463-466](../_source/pages/page-463.md)
- [Exported Port creation and deletion, PDF pp. 467-469](../_source/pages/page-467.md)
