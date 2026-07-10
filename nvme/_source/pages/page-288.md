---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 288
headings: ["5.1.12.1.23 Dispersed Namespace Participating NVM Subsystems (Log Page Identifier 17h)"]
---

# Source page 288

NVM Express® Base Specification, Revision 2.1

266
Figure 269: Rotational Media Information Log Page
Bytes Description
15:12
Failed Spinup Count  (FSPINC): Contains the total number of failed spinup events for this Endurance
Group over the lifetime of the Endurance Group. If the Failed Spinup Count is less than FFFFFFFFh, then
the controller shall increment this count by one for each failed spinup event.
A failed spinup event occurs when the controller fails an attempt to transition from a non -operational
power state to an operational power state.
19:16
Load Count (LDC): Contains the total number successful actuator load events for this Endurance Group
over the lifetime of the Endurance Group. If the Load Count is less than FFFFFFFFh, then the controller
shall increment this count by one for each successful actuator load event.
A successful actuator load event occurs if an actuator transitions from a non -operational state to an
operational state.
23:20
Failed Load Count (FLDC): Contains the number of failed actuator load events for this Endurance Group
over the lifetime of the Endurance Group. If the Failed Load Count is less than FFFFFFFFh, then the
controller shall increment this count by one for each failed actuator load event.
A failed actuator load event occurs if an actuator fails an attempt to transition from a non-operational state
to an operational state.
511:24 Reserved
5.1.12.1.23 Dispersed Namespace Participating NVM Subsystems (Log Page Identifier 17h)
This log page is used to provide a list of NQNs for all participating NVM subsystems which contain
controllers that are able to provide access to the dispersed namespace for the specified NSID. A typical
Dispersed Namespace implementation uses Fabrics based NVM subsystems.
If an NSID of 0h or FFFFFFFFh is specified by the host in the Get Log Page command , then the controller
shall abort that command with a status code of Invalid Namespace or Format as described in Figure 92. If
the NSID specified by the host in the Get Log Page command is associated with a namespace that is not
a dispersed namespace (i.e., the Dispersed Namespace (DISNS) bit is cleared to ‘0’ in the Namespace
Multi-path I/O and Namespace Sharing Capabilities  (NMIC) field of the I/O Command Set Independent
Identify Namespace data structure (refer to Figure 319) or the Identify Namespace data structure
associated with that namespace), then the controller shall abort that command with a status code of Invalid
Field in Command as described in section 5.1.12.
The method used by the NVM subsystem containing the controller processing the command to discover
the NQNs of separate participating NVM subsystems which contain controllers that are able to provide
access to the same dispersed namespace is outside the scope of this specification.
The log page returned is defined in Figure 270. Each entry in the Participating NVM Subsystems List shall
contain an NQN that matches the NQN contained in the NVM Subsystem NVMe Qualified Name (SUBNQN)
field of that participating NVM subsystem’s Identify Controller data structure (refer to Figure 312). The
Participating NVM Subsystem Entry 0 field (i.e., first entry in the Participating NVM Subsystems List) shall
contain the NQN of the NVM subsystem containing the controller processing the command.
Figure 270: Dispersed Namespace Participating NVM Subsystems Log Page
Bytes Description
Header
07:00
Generation Counter (GENCTR): This field contains a value that is incremented each time
the participating NVM subsystem information for the specified NSID changes (i.e., each time
participating NVM subsystems are added or removed for the specified NSID) since the last
time the host read this log page. If the value of this field is FFFFFFFF_FFFFFF FFh, then
the field shall be cleared to 0h when incremented (i.e., rolls over to 0h).
15:08 Number of Participating NVM Subsystems (NUMPSUB): This field indicates the number
of participating NVM subsystems contained in the log page.
255:16 Reserved
Participating NVM Subsystems List
