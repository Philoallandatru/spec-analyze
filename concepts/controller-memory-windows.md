# Controller Memory Windows

NVMe exposes two optional controller-associated memory facilities with different contracts: the Controller Memory Buffer (CMB) may host queues, lists, or command data according to advertised capabilities; the Persistent Memory Region (PMR) adds persistence, readiness, health, and write-barrier semantics. [PDF pp. 86-94](../_source/pages/page-086.md)

## CMB address model

```text
PCI BAR selected by CMBLOC.BIR
+----------------------------------------------------------+
| BAR base                                                 |
|   + CMBLOC.OFST * CMBSZ.SizeUnit                         |
|        +---------------- CMB ----------------------+      |
|        | queues / PRP-SGL lists / read-write data |      |
|        +-------------------------------------------+      |
+----------------------------------------------------------+

Optional controller-address view:
  CMBMSC.CRE=1 -> CMBLOC/CMBSZ valid
  CMBMSC.CMSE=1 + valid CBA -> host addresses in range select CMB
```

The available size is `CMBSZ.SZ * SizeUnit` but is clipped if offset plus size exceeds the selected BAR. The BAR address is 4 KiB aligned. [PDF pp. 86-89](../_source/pages/page-086.md)

## CMB capability dimensions

| Dimension | Advertised by |
|---|---|
| Submission / Completion Queues | `CMBSZ.SQS`, `CMBSZ.CQS` |
| PRP/SGL lists | `CMBSZ.LISTS` |
| Host-to-controller / controller-to-host data | `CMBSZ.WDS`, `CMBSZ.RDS` |
| Discontiguous or mixed-memory queues | `CMBLOC.CQPDS`, `CMBLOC.CQMMS` |
| Mixed command/data/list placement | `CMBLOC.CDMMMS`, `CDPCILS`, `CDPMLS` |
| Dword queue alignment | `CMBLOC.CQDA` |

A cleared capability generally means the corresponding placement restriction remains enforced; support must not be inferred merely from CMB existence. [PDF pp. 86-88](../_source/pages/page-086.md)

`CMBSTS.CBAI=1` reports that controller-address-space enablement failed because the configured base address is invalid. CMB elasticity-buffer size/bypass behavior and sustained write throughput are separately reported, with zero meaning no information is available. [PDF pp. 89-91](../_source/pages/page-089.md)

## PMR enable/readiness handshake

```text
[PMR disabled / NRDY=1]
          |
          | PMRCTL.EN=1
          | wait PMRCAP.PMRTO * PMRTU
          v
[PMR enabled, wait] ---- PMRSTS.CBAI/health error ---> [diagnose]
          |
          | PMRSTS.NRDY=0
          v
[PMR ready for PCIe reads/writes]
          |
          | persistence barrier:
          |   memory read completion and/or PMRSTS read completion
          v
[prior writes persistent]
```

At least one advertised write-barrier mechanism is supported. PMR time units are either 500 milliseconds or minutes; readiness is valid only when `PMRCTL.EN=1` and `PMRSTS.NRDY=0`. [PDF pp. 92-94](../_source/pages/page-092.md)

## PMR health states

| `PMRSTS.HSTS` | Meaning |
|---|---|
| `000b` | Normal operation |
| `001b` | Persistent and operating, but restore may be incorrect |
| `010b` | Read only |
| `011b` | Unreliable; data or persistence may be invalid |

A non-zero `PMRSTS.ERR` records a prior write error and remains non-zero until PCI Function reset. [PDF p. 94](../_source/pages/page-094.md)

The PMR controller-address base is split across aligned 32-bit properties: `PMRMSCU.CBA` provides the upper 32 bits and `PMRMSCL.CBA` the next 20 bits, with the low 12 address bits implicitly zero. Enabling `PMRMSCL.CMSE` requires a valid range that neither overflows 64-bit address space nor overlaps an enabled CMB controller-address range. [PDF p. 96](../_source/pages/page-096.md)

PMR elasticity-buffer size and sustained write throughput use `value * unit` encodings. A zero property value means the controller provides no information about that characteristic, not necessarily that the facility is absent. [PDF p. 95](../_source/pages/page-095.md)

## CMB versus PMR

```text
                 CMB                         PMR
Purpose          low-latency controller mem persistent controller mem
Persistence      not promised here           explicit barrier + health
Enable/status    CRE/CMSE + CBAI             EN/NRDY + CBAI/HSTS/ERR
Placement        queues/lists/data by bits   read/write data by capability
```

This comparison is explanatory; detailed placement rules are defined later in the specification. [PDF pp. 86-94](../_source/pages/page-086.md)

## Evidence

- [CMB location and size capabilities, PDF pp. 86-88](../_source/pages/page-086.md)
- [CMB address-space control and status, PDF pp. 89-90](../_source/pages/page-089.md)
- [CMB throughput and NVM Subsystem Shutdown, PDF pp. 90-91](../_source/pages/page-090.md)
- [PMR capabilities/control/status, PDF pp. 92-94](../_source/pages/page-092.md)
