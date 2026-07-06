---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 561
headings: ["8.1.18 Predictable Latency Mode"]
---

# Source page 561

NVM Express® Base Specification, Revision 2.1

539
Figure 627: HCTM Example
TMT1
Vendor
Specific
TMT2
No Thermal Management
e.g., light throttle
e.g., heavy thottling
Lines represent the
Composite Temperature

Note: Since the host controlled thermal management (HCTM) feature uses the Composite Temperature,
the actual interactions between a platform (e.g., tablet, or laptop) and two different device implementations
may vary even with the same Thermal Management Temperature 1 and Thermal Management
Temperature 2 temperature settings.  The use of this feature requires validation between those devices ’
implementations and the platform in order to be used effectively.
The SMART / Health Information log page (refer to section 5.1.12.1.3) contains statistics related to Host
Controller Thermal Management.
 Predictable Latency Mode
Predictable Latency Mode is used to achieve predictable latency for read and write operations. When
configured to operate in this mode using the Predictable Latency Mode Config Feature  (refer to section
5.1.25.1.12), the namespaces in an NVM Set (refer to section  3.2.2) provide windows of operation for
deterministic operation or non-deterministic operation.
When Predictable Latency Mode is enabled:
• NVM Sets and their associated namespaces have vendor specific quality of service attributes;
• I/O commands that access NVM in the same NVM Set have the same quality of service attributes;
and
• I/O commands that access NVM in one NVM Set do not impact the quality of service of I/O
commands that access NVM in a different NVM Set.
The quality of service attributes apply within the NVM subsystem and do not include the PCIe or fabric
connection. To enhance isolation, the host should submit I/O commands for different NVM Sets to different
I/O Submission Queues.
