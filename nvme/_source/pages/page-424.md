---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 424
headings: ["5.1.25.1.28.1 PCIe Transport Implementations", "5.1.25.1.28.2 NVMe over Fabrics Implementations", "5.1.25.1.29 Reservation Notification Mask (Feature Identifier 82h)", "5.1.25.1.30 Reservation Persistence (Feature Identifier 83h)"]
---

# Source page 424

NVM Express® Base Specification, Revision 2.1

402
 PCIe Transport Implementations
The Host Identifier is an optional feature when implemented on a controller using a PCIe transport. The
controller may support a 64-bit Host Identifier and/or an extended 128-bit Host Identifier. It is recommended
that implementations support the extended 128 -bit Host Identifier as indicated in the Controller Attributes
field in the Identify Controller data structure.
The Host Identifier for PCIe transport implementations shall have a default value of 0h and shall not have
a saved value. NVM Express Base Specification , Revision 2.0 and earlier allowed saving a Host Identifier
value, if a non-zero Host Identifier value had been saved then that saved value shall be reset to the default
value.
 NVMe over Fabrics Implementations
The Host Identifier is a mandatory feature in NVMe over Fabrics implementations. The Host Identifier shall
be an extended 128-bit Host Identifier.
5.1.25.1.29 Reservation Notification Mask (Feature Identifier 82h)
This Feature controls the masking of reservation notifications on a per namespace basis. A Reservation
Notification log page is created whenever a reservation notification occurs on a namespace and the
corresponding reservation notification type is not masked on that namespace by this Feature. If reservations
are supported by the controller, then this Feature shall be supported. The attributes are specified in
Command Dword 11.
A Set Features command that uses a namespace ID other than FFFFFFFFh modifies the reservation
notification mask for the corresponding namespace only. A Set Features command that uses a namespace
ID of FFFFFFFFh modifies the reservation notification mask of  all namespaces that are attached to the
controller and that support reservations. A Get Features command that uses a namespace ID other than
FFFFFFFFh returns the reservation notification mask for the corresponding namespace. A Get Features
command that u ses a namespace ID of FFFFFFFFh shall be aborted with status  code of Invalid Field in
Command. If a Set Features  command or a Get Features command attempts to access the Reservation
Notification Mask on a namespace that does not support reservations or is invalid, then that command is
aborted with status code of Invalid Field in Command.
If a Get Features command successfully completes for this Feature, the attributes specified in Figure 439
are returned in Dword 0 of the completion queue entry for that command.
Figure 439: Reservation Notification Configuration – Command Dword 11
Bits Description
31:04 Reserved
03
Mask Reservation Preempted Notification (RESPRE):  If this bit is set to ‘1’, then mask the reporting
of reservation preempted notification by the controller. If  this bit is cleared to ‘0’, then the notification is
not masked and a Reservation Notification log page is created whenever notification occurs.
02
Mask Reservation Released Notification (RESREL): If this bit is set to ‘1’, then mask the reporting of
reservation released notification by the controller. If this bit is cleared to ‘0’, then the notification is not
masked and a Reservation Notification log page is created whenever the notification occurs.
01
Mask Registration Preempted Notification (REGPRE): If this bit is set to ‘1’, then mask the reporting
of registration preempted notification by the controller. If this bit is cleared to ‘0’, then the notification is
not masked and a Reservation Notification log page is created whenever the notification occurs.
00 Reserved
5.1.25.1.30 Reservation Persistence (Feature Identifier 83h)
Each namespace that supports reservations has a Persist Through Power Loss (PTPL) state that may be
modified using either a Set Features command or a Reservation Register command (refer to section 7.6).
The Reservation Persistence feature attributes are specified in Command Dword 11.
The PTPL state is contained in the Reservation Persistence feature that is namespace specific. A Set
Features command that uses the namespace ID FFFFFFFFh modifies the PTPL state associated with all
