---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 150
headings: ["3.9.4.2 Traffic Based Keep Alive on the Host"]
---

# Source page 150

NVM Express® Base Specification, Revision 2.1

128
Figure 90: Detecting Timeout Takes up to 2 * KATT
Figure 90 shows that periodic check 3, not periodic check 2, detects the Keep Alive Timeout. Therefore,
the time between fetching the most recent command and the check that detects the timeout (i.e., periodic
check 3 in Figure 90) is up to 2 * KATT.
 Traffic Based Keep Alive on the Host
The host is able to use Traffic Based Keep Alive only if the controller is also using Traffic Based Keep Alive.
The host should not use Traffic Based Keep Alive if the controller is not using Traffic Based Keep Alive
because a controller that uses Command Based Keep Alive detects a Keep Alive Timeout based on the
absence of Keep Alive commands, not the absence of all commands.
Traffic Based Keep Alive on the host is the same as Command Based Keep Alive on the host (refer to
section 3.9.3.2), with two exceptions:
• The host is not required to submit a Keep Alive command if the host submitted an Admin command
or I/O command and the host processed the completion for that command since the most recent
time the host checked whether sending a Keep Alive command was necessary.
• To prevent the controller from detecting a Keep Alive Timeout during the use of Traffic Based Keep
Alive on the host, the host should check for sending a Keep Alive command at a rate of KATT/4,
instead of sending a Keep Alive command at a rate of KATT/2, while the Keep Alive Timer is active
on the controller.
Like Command Based Keep Alive, if the host receives a successful completion to a Set Features command
for the Keep Alive feature, then the host should adjust the time at which the host checks for sending the
next Keep Alive command because the controller restarted the Keep Alive Timer .
If a host detects a Keep Alive Timeout and has outstanding commands for which that host has not received
completions (refer to section 3.4.5), then it is strongly recommended that the host take the steps described
in section 9.6 to avoid possible data corruption caused by interaction between outstanding commands and
subsequent commands submitted by that host to another controller.
For an example host implementation of Traffic Based Keep Alive, the host maintains a Keep Alive Send
Timer for each controller to which that host is connected. The host uses the Keep Alive Send Timer to track
when the host checks for sending a Keep Alive c ommand to the corresponding controller. The host does
not know when the controller fetches the Keep Alive command. Conservatively, the host assumes the
controller fetches the Keep Alive command immediately upon the host sending the Keep Alive command.
Whether or not the host sends a Keep Alive command after the Keep Alive Send Timer expires, the host
tracks this time as the last expired timestamp of the Keep Alive Send Timer, for use when starting or
restarting the Keep Alive Send Timer. While the host does  restart the Keep Alive Send Timer after a
successful Set Features command for the KATO feature, the host does not change the last expired
timestamp when sending that Set Features command because at that point in time the host does not know
the results of that command.
This example host implementation of Traffic Based Keep Alive behaves as follows :
Periodic
check 1



KATT
 KATT
Periodic
check 3

Command
fetched since
last check, no
timeout
No command
fetched since last
check, timeout
detected
Most recent
fetched command
Periodic
check 2
