---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 122
headings: ["3.4.4.1 Round Robin Arbitration", "3.4.4.2 Weighted Round Robin with Urgent Priority Class Arbitration"]
---

# Source page 122

NVM Express® Base Specification, Revision 2.1

100
the Recommended Arbitration Burst field of the Identify Controller data structure in Figure 312), taking into
consideration any latency requirements. Refer to section 5.1.25.1.1.
 Round Robin Arbitration
If the round robin arbitration mechanism is selected, the controller shall implement round robin command
arbitration amongst all Submission Queues, including the Admin Submission Queue. In this case, all
Submission Queues are treated with equal priority. The controller may select multiple candidate commands
for processing from each Submission Queue per round based on the Arbitration Burst setting.
Figure 80: Round Robin Arbitration

 Weighted Round Robin with Urgent Priority Class Arbitration
In this arbitration mechanism, there are three strict priority classes and three weighted round robin priority
levels. If Submission Queue A is of higher strict priority than Submission Queue B, then all candidate
commands in Submission Queue A shall start  processing before candidate commands from Submission
Queue B start processing.
The highest strict priority class is the Admin class that includes any command submitted to the Admin
Submission Queue. This class has the highest strict priority above commands submitted to any other
Submission Queue.
The next highest strict priority class is the Urgent class. Any I/O Submission Queue assigned to the Urgent
priority class is serviced next after commands submitted to the Admin Submission Queue, and before any
commands submitted to a weighted round robin priority level. Host software should use care in assigning
any Submission Queue to the Urgent priority class since there is the potential to starve I/O Submission
Queues in the weighted round robin priority levels as there is no fairness protocol between Urgent and non
Urgent I/O Submission Queues.
The lowest strict priority class is the Weighted Round Robin class. This class consists of the three weighted
round robin priority levels (High, Medium, and Low) that share the remaining bandwidth using weighted
round robin arbitration. Host software contr ols the weights for the High, Medium, and Low service classes
via the Set Features command. Round robin is used to arbitrate within multiple Submission Queues
assigned to the same weighted round robin level. The number of candidate commands that may start
processing from each Submission Queue per round is either the Arbitration Burst setting or the remaining
weighted round robin credits, whichever is smaller.
ASQ
SQ
SQ
SQ
RR
