---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 151
headings: ["3.9.5 Keep Alive Timeout Cleanup"]
---

# Source page 151

NVM Express® Base Specification, Revision 2.1

129
• The host enables the Keep Alive Send Timer if the host requests activation of the Keep Alive Timer
on the controller (e.g., the host enables the controller (refer to section 3.9.2)). The host records the
current time as the last expired timestamp when the Keep Alive Send Timer becomes enabled .
• The host disables the Keep Alive Send Timer if the host requests deactivation of the Keep Alive
Timer on the controller and the host receives a successful response from the controller (e.g., the
host disables the controller (refer to section 3.9.2)).
• The host starts a Keep Alive Send Timer if:
o the Keep Alive Send Timer becomes enabled; or
o a successful Keep Alive command completion is processed by the host.
• The host restarts a Keep Alive Send Timer if:
o the Keep Alive Send Timer is enabled;
o a Keep Alive command is not outstanding; and
o a successful Set Features command completion is processed by the host where the
command specified the Keep Alive Timer feature (i.e., Feature Identifier 0Fh) and a non -
zero KATO field.
• If a Keep Alive Send Timer starts or restarts, the host sets the Keep Alive Send Timer to:
o KATT/4 minus the time elapsed since the last expired timestamp; or
o zero, if the time elapsed since the last expired timestamp is greater than KATT/4.
• The host stops a Keep Alive Send Timer if the Keep Alive Send Timer becomes disabled
• If a Keep Alive Send Timer expires, (i.e., the Keep Alive Send Timer is still enabled and time has
elapsed equal to the value to which the Keep Alive Send Timer was set since the host most recently
started or restarted the Keep Alive Send Timer), then the host records the current time as the last
expired timestamp and the host either:
o starts a new Keep Alive Send Timer, if at least one Admin command or I/O command was
submitted to the controller and the completion of that command was processed by the host
since the last Keep Alive Send Timer started (not restarted); or
o sends a Keep Alive command.
• The host detects a Keep Alive Timeout if the host sends a Keep Alive command and does not
receive a completion for that Keep Alive command before KATT elapses from when the Keep Alive
command was submitted.
 Keep Alive Timeout Cleanup
If a controller detects a Keep Alive Timeout, then the controller shall  perform the following actions within
the time specified by the CQT field (refer to Figure 312):
• record an Error Information Log Entry with the status code Keep Alive Timeout Expired;
• stop processing commands;
• set the Controller Fatal Status (CSTS.CFS) bit to ‘1’; and
• for message-based transports:
o terminate the NVMe Transport connections for this association; and
o break the host to controller association.
For message-based transports, after completing these steps, a controller may accept a Connect command
(refer to section 6.3) for the Admin Queue from the same or another host in order to form a new association.
If a host detects a Keep Alive Timeout and has outstanding commands for which that host has not received
completions (refer to section 3.4.5), then it is strongly recommended that the host takes steps described in
