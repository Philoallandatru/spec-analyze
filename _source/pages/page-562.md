---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 562
headings: ["8.1.18.1 Host Operating Rules to Achieve Determinism"]
---

# Source page 562

NVM Express® Base Specification, Revision 2.1

540
Read Recovery Levels (refer to section 8.1.20) shall be supported when Predictable Latency Mode is
supported. The host configures the Read Recovery Level to specify the tradeoff between the quality of
service versus the amount of error recovery to apply for a particular NVM Set.
The Deterministic Window (DTWIN) is the window of operation during which the NVM Set is able to provide
deterministic latency for read and write operations. The Non-Deterministic Window (NDWIN) is the window
of operation during which the NVM Set is not abl e to provide deterministic latency for read and write
operations as a result of preparing for a subsequent Deterministic Window. Examples of actions that may
be performed in the Non-Deterministic Window include background operations on the non -volatile storage
media. The current window that an NVM Set is operating in is configured by the host using the Predictable
Latency Mode Window Feature or by the controller as a result of an autonomous action.
Figure 628: Deterministic and Non-Deterministic Windows

To remain in the Deterministic Window, the host is required to follow operating rules (refer to section
8.1.18.1) ensuring that certain attributes do not exceed the typical or maximum values indicated in the
Predictable Latency Per NVM Set log page. If the attributes exceed any of the typical or maximum values
indicated in the Predictable Latency Per NVM Set log pag e or a Deterministic Excursion occurs, then the
associated NVM Set may autonomously transition to the Non -Deterministic Window. A Deterministic
Excursion is a rare occurrence in the NVM subsystem that requires immediate action by the controller.
The host configures Predictable Latency Events to report using the Predictable Latency Mode Config
feature. The host may configure a Predictable Latency Event to be triggered when that value exceeds a
specific value in order to manage window changes and av oid autonomous transitions by the controller.
Refer to section 8.1.18.3.
If Predictable Latency Mode is supported, then all controllers in the NVM subsystem shall:
• Support one or more NVM Sets;
• Support Read Recovery Levels;
• Support the Predictable Latency Mode log page for each NVM Set;
• Support the Predictable Latency Event Aggregate log page;
• Support the Predictable Latency Mode Config Feature;
• Support the Predictable Latency Mode Window Feature;
• Support Predictable Latency Event Aggregate Log Change Notices; and
• Indicate support for Predictable Latency Mode in the Controller Attributes field in the Identify
Controller data structure.
 Host Operating Rules to Achieve Determinism
In order to achieve deterministic operation, the host is required to follow operating rules.
An NVM Set remains in the Deterministic Window while attributes do not exceed any of the typical or
maximum values indicated in the Predictable Latency Per NVM Set log page, there is not a Deterministic
Excursion, and the host does not request a transition  to the Non -Deterministic Window. The attributes
