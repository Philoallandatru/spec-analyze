# Controller Support Requirements

NVMe defines controller support independently across commands, log pages, and Features. Each item is Mandatory (`M`), Optional (`O`), or Prohibited (`P`) for each functional controller type, with notes that may make support conditional on transport, another capability, controller role, or subsystem kind. [PDF pp. 65-70](../_source/pages/page-065.md)

## Mental model

```text
                         Controller type
                 +-----------+-----------+-----------+
Capability plane | I/O       | Admin.    | Discovery |
+----------------+-----------+-----------+-----------+
| Admin commands | broad     | management| discovery |
| Fabrics cmds   | by transport and command          |
| I/O commands   | data path | prohibited| prohibited|
| Log pages      | I/O/health| management| discovery |
| Features       | I/O/control| management| discovery|
+----------------+-----------+-----------+-----------+

Each cell is M, O, P, or conditional - never infer support from the row name alone.
```

This is a navigation summary of Figures 28-32, not a replacement for their exact entries and notes. [PDF pp. 65-70](../_source/pages/page-065.md)

## Three support planes

| Plane | How a host discovers exact support | Important invariant |
|---|---|---|
| Commands | Commands Supported and Effects log page | I/O-Command-Set-specific requirements also come from that command-set specification |
| Log pages | Supported Log Pages plus controller-type requirements | Scope and prerequisites may change `O` into mandatory or prohibited |
| Features | Feature Identifiers Supported and Effects log page | Supporting any Feature requires Get Features and Set Features |

[PDF pp. 65, 67-70](../_source/pages/page-065.md)

## Transport and role boundaries

```text
NVMe over PCIe
  `-- Fabrics commands: prohibited
  `-- Create/Delete I/O queues: mandatory for I/O controllers

NVMe over Fabrics
  `-- Property Get/Set + Connect: mandatory for every controller type
  `-- Disconnect: optional for I/O; prohibited for Admin/Discovery

Discovery controller
  `-- discovery commands/logs: required by role
  `-- ordinary I/O commands and namespace data path: prohibited
```

[PDF pp. 65-68](../_source/pages/page-065.md)

## Conditionality is part of the contract

An `O` entry is not always freely optional. Examples include reservation commands becoming mandatory when reservations are supported, FDP commands/log pages becoming mandatory with Flexible Data Placement, and persistent Discovery connections requiring Keep Alive and asynchronous-event support. Notes attached to the matrix are therefore semantically part of each cell. [PDF pp. 66-70](../_source/pages/page-066.md)

## Evidence

- [Admin command support matrix, PDF pp. 65-66](../_source/pages/page-065.md)
- [Fabrics and common I/O command matrices, PDF pp. 66-67](../_source/pages/page-066.md)
- [Log page support matrix, PDF pp. 67-69](../_source/pages/page-067.md)
- [Feature support matrix, PDF pp. 69-70](../_source/pages/page-069.md)
