---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 537
headings: ["8.1.9.5 ANA Considerations", "8.1.9.6 Reservation Considerations"]
---

# Source page 537

NVM Express® Base Specification, Revision 2.1

515
• any of the Admin commands listed in Figure 615 that specify the NSID of a dispersed namespace
in the command.
The Additional Restrictions column in Figure 615 lists additional restrictions that apply while the HDISNS
field is cleared to 0h in the Host Behavior Support feature for Admin commands that are capable of affecting
dispersed namespaces without specifying the NSID of a dispersed namespace.
Figure 615: Dispersed Namespaces Command Restrictions - Prohibited Admin Commands
Admin Command Additional Restrictions
Directive Receive None.
Directive Send None.
Set Features
This command is prohibited if a dispersed namespace is attached to the controller and
an NSID of FFFFFFFFh is specified for a feature capable of affecting all namespaces
attached to that controller.
Format NVM
This command is prohibited if a dispersed namespace exists in the participating NVM
subsystem and the scope of the command includes all  namespaces that exist in that
NVM subsystem (refer to Figure 188).
This command is prohibited if a dispersed namespace is attached to the controller and
the scope of the command includes all namespaces attached to that controller (refer to
Figure 188).
 ANA Considerations
If more than one participating NVM subsystem contains controllers that provide a host with access to a
dispersed namespace, then Asymmetric Namespace Access (ANA) and Asymmetric Namespace Access
Reporting (refer to section 8.1.1) should be supported in all participating NVM subsystems. If ANA is used
with dispersed namespaces, then globally unique namespace identifiers (refer to section 8.1.9.2) are used
to determine when multiple paths are available to the same dispersed namespace. Hosts specify to
controllers that their host software (e.g., multipath I/O software) uses globally unique namespace identifiers
to determine when multiple paths are available to the dispersed namespace by setting the HDISNS field to
1h in the Host Behavior Support feature, as described in section 8.1.9.3.
ANA Group usage with dispersed namespaces is described in section 8.1.1.2.
 Reservation Considerations
Reservations for a dispersed namespace that is able to be accessed by controllers in multiple participating
NVM subsystems are intended to be coordinated between each participating NVM subsystem. To ensure
the uniqueness of a host’s identity and prevent po tential data corruption, each host that sets the Host
Dispersed Namespace Support (HDISNS) field to 1h in the Host Behavior Support feature specifies a Host
Identifier that is unique across all hosts that connect to any participating NVM subsystem, as desc ribed in
section 8.1.9.3.
If a host is a reservation registrant on a dispersed namespace, and that host submits a Reservation Report
command to that namespace, then for reservations established using controllers that are not contained in
the same participating NVM subsystem as the controller processing the Reservation Report command, the
Controller ID (CNTLID) field shall be set to FFFDh in the Registered Controller data structure (refer to Figure
583) or Registered Controller Extended data structure (refer to Figure 584). Multiple identical Registered
Controller or Registered Controller Extended data structures with the CNTLID field set to FFFDh shall not
be returned by a single Reservation Report command; as a result, a Registered Controller or Registered
Controller Extended data structure with the CNTLID field set to FFFDh indicates one or more controllers
are associated with a host that is a registrant of the dispersed namespace in another participating NVM
subsystem. A Registered Controller or Registered Controller Extended data structure with the CNTLID field
set to FFFDh is returned for each host (i.e., as identified by the HOSTID field in that data structure) that is
a registrant of the dispersed namespace in any other participating NVM subsystem.
The Dispersed Namespace Reservation Support (DISNSRS) bit in a command is set to ‘1’ by hosts that
support reservations on dispersed namespaces (i.e., hosts that support receiving a value of FFFDh in the
