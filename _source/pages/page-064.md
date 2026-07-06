---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 64
headings: ["3.1.3.3.1 Discovery Controller Asynchronous Event Configuration", "3.1.3.3.2 Discovery Controller Asynchronous Event Information – Requests and Notifications"]
---

# Source page 64

NVM Express® Base Specification, Revision 2.1

42
Figure 27: Controller IDs FFF0h to FFFFh
Controller ID Definition
FFF0h to FFFCh Reserved. Use of this value in a Connect command results in a status code of Connect Invalid
Parameters being returned, as described in section 6.3.
FFFDh
This value in the Controller ID (CNTLID) field of the Registered Controller data structure or
Registered Controller Extended data structure for a dispersed namespace indicates that the
controller is not contained in the same participating NVM subsystem as the controller processing
the command (refer to section 8.1.9.6). Use of this value in a Connect command results in a
status code of Connect Invalid Parameters being returned, as described in section 6.3.
FFFEh This value is sent in a Connect command to specify that any available static controller is allowed
to be allocated.
FFFFh This value is sent in a Connect command to specify that any available dynamic controller is
allowed to be allocated.
The Controller ID values returned in the Discover y Log Page Entries indicate whether an NVM subsystem
supports the dynamic or static controller model. The controller ID value of FFFFh is used for NVM
subsystems that support the dynamic controller model indicating that any available controller may be
returned. The Controller ID value of FFFEh is used for NVM subsystems that support the static controller
model indicating that any available controller may be returned. An NVM subsystem supports the dynamic
controller model if Discovery Log  Page Entries use the Controller ID value of FFFFh. An NVM subsystem
supports the static controller model if Discovery Log Page Entries use a Controller ID value that is less than
FFFFh. The Identify Controller data structure also indicates whether an NVM subsystem is dynamic or
static.
If an NVM subsystem implements the dynamic controller model, then multiple Discovery Log Page Entries
(refer to Figure 294) with the Controller ID set to FFFFh may be returned for that NVM subsystem (e.g., to
indicate multiple NVM subsystem ports) in the Discovery log page. If an NVM subsystem implements the
static controller model, then multiple Discovery Log Page Entries that indicate different Controller ID values
may be returned for that NVM subsystem in the Discovery log page. If an NVM subsystem that implements
the static controller model includes any Discovery Log Page Entries that indicate a Controller ID of FFFEh,
then the host should remember the Controller ID returned from the Fabrics Connect command and re -use
the allocated Controller ID for future associations to that particular controller.
3.1.3.3.1 Discovery Controller Asynchronous Event Configuration
Discovery controllers that support Asynchronous Event Notifications shall implement the Get Features and
Set Features commands. A Discovery controller shall enable Asynchronous Discovery Log Event
Notifications, if a non-zero Keep Alive Timeout (KATO) value is received in the Connect command (refer to
section 6.3) sent to that controller.
Figure 391 defines Discovery controller Asynchronous Event Notifications.
3.1.3.3.2 Discovery Controller Asynchronous Event Information – Requests and Notifications
If a Discovery controller detects an event about which a host has requested notification, then the Discovery
controller shall send an Asynchronous Event with the:
• Asynchronous Event Type field set to Notice (i.e., 2h);
• Log Page Identifier field set to either Discovery (i.e., 70h) or Host Discovery (i.e., 71h) depending
on which log page has changed; and
• Asynchronous Event Information field set as defined in Figure 151.
As a result of a Discovery controller updating Discovery log page(s), that Discovery controller shall send a
Discovery Log Page Change Asynchronous Event notification ( i.e., the Asynchronous Event Information
field set to F0h) to each entity that has requested asynchronous event notifications of this type  (refer to
Figure 151).
