---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 502
headings: ["8.1.1.9 Asymmetric Namespace Access Change Notifications", "8.1.1.10 Asymmetric Namespace Access States Command Processing Effects"]
---

# Source page 502

NVM Express® Base Specification, Revision 2.1

480
While ANA Change state is reported by a controller for the namespace, the host should:
a) after a short delay, retry the command on the same controller for at least ANATT (refer to Figure
312) seconds (e.g., if ANATT is 30, perform 3 retries at 10 s intervals, or 10 retries at 3 s intervals);
or
b) retry the command on a different controller that is reporting ANA Optimized state or ANA Non -
Optimized state.
 Asymmetric Namespace Access Change Notifications
If Asymmetric Namespace Access Change Notices are enabled on a controller, then an Asymmetric
Namespace Access Change Notice shall be sent as described in section 5.1.25.1.5 by the controllers where
the change occurred:
a) if an ANA Group Identifier (refer to Figure 312) changes;
b) if an asymmetric namespace access state transition fails (e.g., a transition begins, but does not
complete and the controller returns to the state that existed before the transition began); or
c) upon entry to the following ANA states , unless the state entry is a result of a namespace
attachment:
• ANA Optimized State;
• ANA Non-Optimized State;
• ANA Inaccessible State; and
• ANA Persistent Loss State.
 Asymmetric Namespace Access States Command Processing Effects
Processing of Admin commands that:
• are not NVM Command Set specific commands; and
• do not use the Namespace Identifier (i.e., Figure 141 – “Namespace Identifier Used” column
indicates “No”),
are not affected by ANA states, except as specified in Figure 588.
Figure 588 describes Asymmetric Namespace Access effects on command processing.
Figure 588: ANA effects on Command Processing
Command ANA State Effects on command processing
Get Features
ANA Inaccessible, ANA
Persistent Loss, or ANA
Change
The following feature identifiers are not available1:
a) Reservation Notification Mask (i.e., 82h);
b) Reservation Persistence (i.e., 83h); and
c) I/O Command Set specific feature identifiers2.
Get Log Page
ANA Inaccessible, ANA
Persistent Loss, or ANA
Change
The following log pages are affected:
a) Error Information (i.e., 01h): The log page is not required
to contain entries for namespaces whose relationship to
the controller processing the command is in the:
a. ANA Inaccessible state (refer to section 8.1.1.6);
b. the ANA Persistent Loss state (refer to section
8.1.1.7); or
c. the ANA Change state (refer to section 8.1.1.8).
The following log pages are not available 1:
a) Media Unit Status log page (refer to section 5.1.12.1.16);
and
b) Supported Capacity Configuration List log page (refer to
section 5.1.12.1.17).
