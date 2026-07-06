---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 536
headings: ["8.1.9.4 Prohibiting Host Access to Dispersed Namespaces"]
---

# Source page 536

NVM Express® Base Specification, Revision 2.1

514
• treating all instances of a dispersed namespace as the same namespace, based upon the shared
globally unique namespace identifier when accessing that dispersed namespace through
controllers on multiple participating NVM subsystems; and
• specifying a Host Identifier that is unique across all hosts that connect to any participating NVM
subsystem.
If the HDISNS field is set to 1h, the controller shall not abort any command that the host submits to
dispersed namespaces with a status code of Host Dispersed Namespace Support Not Enabled.
A participating NVM subsystem shall support prohibiting host access to dispersed namespaces when the
HDISNS field is cleared to 0h in the Host Behavior Support feature, as described in section 8.1.9.4. A
participating NVM subsystem may either:
• allow host access to a dispersed namespace when the HDISNS field is cleared to 0h in the Host
Behavior Support feature if the host associated with the Host NQN in the connection is only able to
access the dispersed namespace from a single participating NVM subsystem; or
• prohibit host access to dispersed namespaces when the HDISNS field is cleared to 0h in the Host
Behavior Support feature.
A participating NVM subsystem that allows host access to a dispersed namespace when the HDISNS field
is cleared to 0h in the Host Behavior Support feature if the host associated with the Host NQN in the
connection is only able to access the dispersed names pace from a single participating NVM subsystem
shall implement a method to determine if the host is able to access the dispersed namespace from multiple
participating NVM subsystems. The method used to determine if the host is able to access the dispersed
namespace from multiple participating NVM subsystems is outside the scope of this specification. If a
participating NVM subsystem determines that a Connect command requests a host with the HDISNS field
cleared to 0h in the Host Behavior Support feature hav ing access to the same dispersed namespace
through multiple participating NVM subsystems, then that participating NVM subsystem shall either:
a) abort a Connect command with a status code of Connect Invalid Host if that Connect command
requests the host associated with the Host NQN in the connection having access to a dispersed
namespace through multiple participating NVM subsystems. Aborting a Connect command for this
reason may result in the host being unable to access other namespaces (i.e., namespaces that are
unrelated to the dispersed namespace) on that NVM subsystem . This enforces the requirement
that a host that has not set the HDISNS field to 1h in the Host Behavior Support feature be allowed
to access a dispersed namespace on only one participating NVM subsystem by preventing all
access to any other participating NVM subsystems (including all other namespaces on those other
participating NVM subsystems); or
b) allow a Connect command that requests the host associated with the Host NQN in the connection
having access to a dispersed namespace through multiple participating NVM subsystems, but
prohibit host access to that dispersed namespace from multiple participa ting NVM subsystems
(e.g., prohibit host access to that dispersed namespace on the second participating NVM
subsystem) as described in section 8.1.9.4. This enforces the requirement that a host that has not
set the HDISNS field to 1h in the Host Behavior Support feature be allowed to access a dispersed
namespace on only one participating NVM subsystem by, for each dispersed namespace,
preventing access to that dispersed namespace on any other participating NVM subsystems (and
allowing access to other namespaces that are not dispersed namespaces on those other
participating NVM subsystems).
 Prohibiting Host Access to Dispersed Namespaces
If a participating NVM subsystem prohibits host access to dispersed namespaces when the Host Dispersed
Namespace Support (HDISNS) field is cleared to 0h in the Host Behavior Support feature, then the
controller shall abort the following commands with a status code of Host Dispersed Namespace Support
Not Enabled:
• any I/O commands (refer to section 7 in this specification and the I/O Commands section in the
appropriate I/O Command Set specification) that specify the NSID of a dispersed namespace in
the command; or
