---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 556
headings: ["8.1.17 Power Management"]
---

# Source page 556

NVM Express® Base Specification, Revision 2.1

534
c) Commands that do not specify an NSID, and the execution of which would modify a namespace
that is write protected (e.g., Sanitize command).
 Power Management
The power management capability allows the host to manage NVM subsystem power statically or
dynamically. Static power management consists of the host determining the maximum power that may be
allocated to an NVM subsystem and setting the NVM Express power state to one that consumes this
amount of power or less. Dynamic power management is illustrated in Figure 624 and consists of the host
modifying the NVM Express power state to best satisfy changing power and performance objectives.  This
power management mechanism is meant to complement and not replace autonomous power management
or thermal management performed by a controller.
Figure 624: Dynamic Power Management

The number of power states implemented by a controller is returned in the Number of Power States
Supported (NPSS) fie ld in the Identify Controller d ata structure. If the controller supports this feature,  at
least one power state shall be defined and optionally, up to a total of 32 power states  may be supported.
Power states shall be contiguously numbered starting with zero such that each subsequent power state
consumes less than or equal to the maximum power consumed in the previous state.  Thus, power state
zero indicates the maximum power that the NVM subsystem is capable of consuming.
Associated with each power state is a Power State Descriptor in the Identify Controller data structure (refer
to Figure 313). The descriptors for all implemented power states may be viewed as forming a table as
shown in the example in Figure 625 for a controller with seven implemented power states. Note that Figure
625 is illustrative and does not include all fields in the power state descriptor.  The Maximum Power (MP)
field indicates the sustained maximum power that may be consumed in that state , where power
measurement methods are outside the scope of this specification. The controller may employ autonomous
power management techniques to reduce power consumption below this level, but under no circumstances
is power allowed to exceed this level  except for non -operational power states as described in section
8.1.17.1.
Figure 625: Example Power State Descriptor Table
Power
State
Maximum
Power
(MP)
Entry
Latency
(ENLAT)
Exit
Latency
(EXLAT)
Relative
Read
Throughput
(RRT)
Relative
Read
Latency
(RRL)
Relative
Write
Throughput
(RWT)
Relative
Write
Latency
(RWL)
0 25 W 5 µs 5 µs 0 0 0 0
1 18 W 5 µs 7 µs 0 0 1 0
2 18 W 5 µs 8 µs 1 0 0 0
3 15 W 20 µs 15 µs 2 0 2 0
4 10 W 20 µs 30 µs 1 1 3 0
5 8 W 50 µs 50 µs 2 2 4 0
