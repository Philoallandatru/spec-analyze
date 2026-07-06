---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 148
headings: ["3.9.3.2 Command Based Keep Alive on the Host"]
---

# Source page 148

NVM Express® Base Specification, Revision 2.1

126
For Command Based Keep Alive:
• The controller shall start the Keep Alive Timer if the Keep Alive Timer becomes active (refer to
section 3.9.2).
• The controller shall restart the Keep Alive Timer if the Keep Alive Timer is active, and:
o a Keep Alive command completes successfully; or
o a Set Features command specifying the Keep Alive Timer feature and a non -zero KATO
field (refer to section 5.1.25.1.8) completes successfully.
• The controller shall expire the Keep Alive Timer if:
o the Keep Alive Timer is active in the controller; and
o KATT has elapsed since the Keep Alive Timer was most recently started or restarted.
If the Keep Alive Timer on the controller expires, then the controller shall consider a Keep Alive Timeout to
have occurred. Upon the occurrence of a Keep Alive Timeout, the controller shall perform the cleanup
actions described in section 3.9.5.
 Command Based Keep Alive on the Host
The host may use Command Based Keep Alive regardless of the Keep Alive mode used by the controller .
To prevent the controller from detecting a Keep Alive Timeout during the use of Command Based Keep
Alive on the host , the host should send Keep Alive commands at KATT/2 to account  for delays (e.g.,
transport round -trip times, transport delays, command processing times, and the Keep Alive Timer
granularity) while the Keep Alive Timer is active on the controller. If the host receives a successful
completion to a Set Features command for the Keep Alive feature, then the host should adjust the time at
which the host sends the next Keep Alive command because the controller restarts the Keep Alive Timer .
If a host detects a Keep Alive Timeout and has outstanding commands for which that host has not received
completions (refer to section 3.4.5), then it is strongly recommended that the host take the steps described
in section 9.6 to avoid possible data corruption caused by interaction between outstanding commands and
subsequent commands submitted by that host to another controller.
For an example host implementation of Command Based Keep Alive, the host maintains a Keep Alive Send
Timer for each controller to which the host is connected. The host uses the Keep Alive Send Timer to track
when the host sends a Keep Alive command to the corresponding controller. The host does not know when
the controller fetches the Keep Alive command. Conservatively, the host assumes the controller fetches
the Keep Alive command immediately upon the host sending the Keep Alive command. The host tracks
this time as the last expired timestamp of the Keep Alive Send Timer, for use when starting or restarting
the Keep Alive Send Timer. While the host does restart the Keep Alive Send Timer after a successful Set
Features command for the KATO feature, the host does not change the last expired timestamp when
sending that Set Features command because at that point in time the host does not know the results of that
command.
This example host implementation of Command Based Keep Alive behaves as follows :
• The host enables the Keep Alive Send Timer if the host requests activation of the Keep Alive Timer
on the controller (e.g., the host enables the controller (refer to section 3.9.2)). The host records the
current time as the last expired timestamp when the Keep Alive Send Timer becomes enabled.
• The host disables the Keep Alive Send Timer if the host requests deactivation of the Keep Alive
Timer on the controller and the host receives a successful response from the controller (e.g., the
host disables the controller (refer to section 3.9.2)).
• The host starts a Keep Alive Send Timer if:
o the Keep Alive Send Timer becomes enabled; or
o a successful Keep Alive command completion is processed by the host.
• The host restarts a Keep Alive Send Timer if:
o the Keep Alive Send Timer is enabled;
o a Keep Alive command is not outstanding; and,
