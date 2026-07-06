---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 149
headings: ["3.9.4 Traffic Based Keep Alive", "3.9.4.1 Traffic Based Keep Alive on the Controller"]
---

# Source page 149

NVM Express® Base Specification, Revision 2.1

127
o a successful Set Features command completion is processed by the host where the
command specified the Keep Alive Timer feature (i.e., Feature Identifier 0Fh) and a non -
zero KATO field.
• If a Keep Alive Send Timer starts or restarts, the host sets the Keep Alive Send Timer to:
o KATT/2 minus the time elapsed since the last expired timestamp; or
o zero, if the time elapsed since the last expired timestamp is greater than KATT/2.
• The host stops a Keep Alive Send Timer if the Keep Alive Send Timer becomes disabled.
• If a Keep Alive Send Timer expires, (i.e., the Keep Alive Send Timer is still enabled and time has
elapsed equal to the value to which the Keep Alive Send Timer was set since the host most recently
started or restarted the Keep Alive Send Timer), then the host records the current time as the last
expired timestamp and sends a Keep Alive command.
• The host detects a Keep Alive Timeout if the host sends a Keep Alive command and does not
receive a completion for the Keep Alive command before KATT elapses from when the Keep Alive
command was sent.
 Traffic Based Keep Alive
Traffic Based Keep Alive allows the host and controller to avoid a Keep Alive Timeout in the presence of
Admin or I/O command processing without sending Keep Alive commands.
 Traffic Based Keep Alive on the Controller
If the controller supports the Keep Alive Timer feature, then support for Traffic Based Keep Alive is indicated
by the TBKAS bit in the Controller Attributes  field in the Identify Controller data structure (refer to  Figure
312). If the Controller does not support Traffic Based Keep Alive (i.e., the TBKAS bit is cleared to ‘0’), then
the operation of the Keep Alive Timer feature is described in section 3.9.3.
For Traffic Based Keep Alive:
• The controller shall start a Keep Alive Timeout Interval if the Keep Alive Timer becomes active
(refer to section 3.9.2).
• The controller shall consider a Keep Alive Timeout to have occurred if:
o the Keep Alive Timer is active;
o KATT has elapsed since the start of the most recent Keep Alive Timeout Interval; and
o no Admin command or I/O command was fetched by the controller during the Keep Alive
Timeout Interval.
• The controller shall end a Keep Alive Timeout Interval and:
o not start a new Keep alive Timeout Interval if:
▪ a Keep Alive Timeout occurs; or
▪ the Keep Alive Timer becomes inactive (refer to section 3.9.2).
o start a new Keep Alive Timeout Interval if the Keep Alive Timer is active, and:
▪ a Keep Alive command completes successfully;
▪ a Set Features command specifying the Keep Alive Timer feature and a non -zero
KATO field (refer to section 5.1.25.1.8) completes successfully; or
▪ KATT has elapsed since the start of the Keep Alive Timeout Interval and a Keep
Alive Timeout did not occur during the Keep Alive Timeout Interval (e.g., an Admin
command or an I/O command was fetched by the controller during the Keep Alive
Timeout Interval).
Upon the occurrence of a Keep Alive Timeout, the controller shall perform the cleanup actions described in
section 3.9.5.
A controller using Traffic Based Keep Alive may require up to 2 * KATT after the controller fetches the most
recent command to detect a Keep Alive Timeout as shown in Figure 90.
