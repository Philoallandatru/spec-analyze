---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 78
headings: ["3.1.4.3 Offset Ch: INTMS – Interrupt Mask Set", "3.1.4.4 Offset 10h: INTMC – Interrupt Mask Clear", "3.1.4.5 Offset 14h: CC – Controller Configuration"]
---

# Source page 78

NVM Express® Base Specification, Revision 2.1

56
Figure 38: NVM Express Base Specification Version Property
Reset Values
Specification
Version 1 MJR Field MNR Field TER Field
1.1 1h 1h 0h
1.2 1h 2h 0h
1.2.1 1h 2h 1h
1.3 1h 3h 0h
1.4 1h 4h 0h
2.0 2h 0h 0h
2.1 2h 1h 0h
Notes:
1. The specification version listed includes lettered versions (e.g., 1.4
includes 1.4, 1.4a through 1.4c, etc.).
 Offset Ch: INTMS – Interrupt Mask Set
This property is used to mask interrupts when using pin-based interrupts, single message MSI, or multiple
message MSI. When using MSI -X, the interrupt mask table defined as part of MSI -X should be used to
mask interrupts. Host software shall not access this property when configured for MSI -X; any accesses
when configured for MSI-X is undefined. For interrupt behavior requirements, refer to the Interrupts section
of the NVMe over PCIe Transport Specification.
Figure 39: Offset Ch: INTMS – Interrupt Mask Set
Bits Type Reset Description
31:00 RWS 0h
Interrupt Vector Mask Set (IVMS): This field is bit significant. If a ‘1’ is written to a
bit, then the corresponding interrupt vector is masked from generating an interrupt
or reporting a pending interrupt in the MSI Capability Structure. Writing a ‘0’ to a bit
has no effect. When read, t his field returns the current interrupt mask value within
the controller (not the value of this property). If a bit has a value of a ‘1’, then the
corresponding interrupt vector is masked. If a bit has a value of ‘0’, then the
corresponding interrupt vector is not masked.
 Offset 10h: INTMC – Interrupt Mask Clear
This property is used to unmask interrupts when using pin-based interrupts, single message MSI, or multiple
message MSI. When using MSI -X, the interrupt mask table defined as part of MSI -X should be used to
unmask interrupts. Host software shall not access this property when configured for MSI-X; any accesses
when configured for MSI-X is undefined. For interrupt behavior requirements, refer to the Interrupts section
of the NVMe over PCIe Transport Specification.
Figure 40: Offset 10h: INTMC – Interrupt Mask Clear
Bits Type Reset Description
31:00 RWC 0h
Interrupt Vector Mask Clear (IVMC): This field is bit significant. If a ‘1’ is written to
a bit, then the corresponding interrupt vector is unmasked. Writing a ‘0’ to a bit has
no effect. When read, this field returns the current interrupt mask value within the
controller (not the value of t his property). If a bit has a value of a ‘1’, then the
corresponding interrupt vector is masked . If a bit has a value of ‘0’, then the
corresponding interrupt vector is not masked.
 Offset 14h: CC – Controller Configuration
This property modifies settings for the controller. Host software shall set the Arbitration Mechanism Selected
(CC.AMS), the Memory Page Size (CC.MPS), and the I/O Command Set Selected (CC.CSS) to valid values
