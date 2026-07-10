---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 369
headings: ["5.1.13.4.2 Get Ports List (CNS 1Eh)"]
---

# Source page 369

NVM Express® Base Specification, Revision 2.1

347
Figure 333: Underlying Namespace List Data Structure
Bytes Description
Underlying Namespace List
335:16 Underlying Namespace Entry 1:  The first Underlying Namespace Entry data structure
(refer to Figure 334) in the Underlying Namespace List. If any.
655:336 Underlying Namespace Entry 2: The second Underlying Namespace Entry data structure
(refer to Figure 334) in the Underlying Namespace List. If any.
… …
320*NUMENT+15:
320*(NUMENT-1)+16
Underlying Namespace Entry NUMENT: The last Underlying Namespace Entry data
structure (refer to Figure 334) in the Underlying Namespace List. If any.

Figure 334: Underlying Namespace Entry Data Structure
Bytes Description
255:00 Underlying NVM Subsystem NQN (UNSNQN): The Underlying NVM Subsystem NQN which contains
the Underlying Namespace.
259:256
Namespace Identifier (NSID): This field indicates the NSID value of the namespace in the Underlying
NVM Subsystem identified by the Underlying NVM Subsystem NQN field.
If the value in this field is not an NSID reported in the Underlying Namespace List (refer to 5.1.13.4.1),
then the command shall be aborted with a status code set to Invalid Field in Command.
261:260 Controller ID (CNTLID): Contains the Underlying NVM Subsystem unique controller identifier associated
with the Underlying Namespace. Refer to section 5.1.13.2.1.
319:262 Reserved
5.1.13.4.2 Get Ports List (CNS 1Eh)
A Ports List data structure (refer to Figure 335) is returned containing a list of Underlying Ports that may be
used to export NVMe over Fabrics NVM subsystems.
Figure 335: Ports List Data Structure
Bytes Description
07:00
Generation Counter (GENCTR): This field indicates the version of Port information. This
field shall be cleared to 0h as a result of NVM Subsystem Reset of the Underlying NVM
Subsystem. For each change in the Ports List, this counter is incremented by one. If the
value of this field i s FFFFFFFF_FFFFFFFFh, then the field shall be cleared to 0h when
incremented (i.e., rolls over to 0h).
15:08 Number Entries (NUMENT): Number of Fabrics Transport Entries in the Ports List.
Ports List
591:16 Fabrics Transport Entry 1:  The first Underlying Fabrics Transport Entry data structure
(refer to Figure 336) in the Ports List, if any.
1167:592 Fabrics Transport Entry 2: The second Underlying Fabrics Transport Entry data
structure (refer to Figure 336) in the Ports List, if any..
… …
576*NUMENT+15:
576*(NUMENT-1)+16
Fabrics Transport Entry NUMENT: The last Underlying Fabrics Transport Entry data
structure (refer to Figure 336) in the Ports List, if any..

Figure 336: Underlying Fabrics Transport Entry Data Structure
Bytes Description
255:00
Transport Address (TRADDR):  This field indicates the address of the NVM subsystem that may be
used for a Connect command as an ASCII string. The Transport Address Family field describes the
reference for parsing this field. Refer to section 1.4 for ASCII string requirements. For the definition of
this field, refer to the appropriate NVMe Transport specification.
511:256 Transport Specific Address Subtype (TSAS): This field indicates NVMe Transport specific information
about the address. For the definition of this field, refer to the appropriate NVMe Transport specification.
