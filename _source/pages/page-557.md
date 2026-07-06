---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 557
headings: ["8.1.17.1 Non-Operational Power States"]
---

# Source page 557

NVM Express® Base Specification, Revision 2.1

535
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
6 5 W 20 µs 5,000 µs 4 3 5 1
The Idle Power (IDLP) field indicates the typical power consumed by the NVM subsystem over 30 seconds
in the power state when idle (e. g., there are no pending commands , property accesses, background
processes, nor device self -test operations). The measurement starts after the NVM subsystem has been
idle for 10 seconds.
The Active Power (ACTP) field indicates the largest average power of the NVM subsystem over a 10 second
window on a particular workload (refer to section 8.1.17.3). Active Power measurement starts when the first
command is submitted and ends when the last command is completed. The largest average power over a
10 second window, consumed by the NVM subsystem in that state is reported in the Active Power field.  If
the workload completes faster than 10 seconds, the average active power should be measured over the
period of the workload. Non-operational states shall set Active Power Scale, Active Power Workload, and
Active Power fields to 0h.
The host may dynamically modify the power state using the Set Features command and determine the
current power state using the Get Features command.  The host may directly transition between any two
supported power states.  The Entry Latency (ENLAT) field in the Power State Descript or data structure
indicates the maximum amount of time in microseconds to enter that power state and the Exit Latency
(EXLAT) field indicates the maximum amount of time in microseconds to exit that state.
The maximum amount of time to transition between any two power states is equal to the sum of the old
state’s exit latency and the new state’s entry latency.  The host  is not required to wait for a previously
submitted power state transition to complete before initiating a new transition.  The maximum amount of
time for a sequence of power state transitions to complete is equal to the sum of transition times for each
individual power state transition in the sequence.
Associated with each power state descriptor are Relative Read Throughput (R RT), Relative Write
Throughput (RWT), Relative Read Latency (RRL) and Relative Write Latency (RWL) fields that provide the
host with an indication of relative performance in that power state. Relative performance values provide an
ordering of performance characteristics between power states.  Relative performance values may repeat,
may be skipped, and may be assigned in any order (i.e., increasing power states are not required to have
increasing relative performance values).
A lower relative performance value indicates better performance (e.g., higher throughput or lower latency).
For example, in Figure 625 power state 1 has higher read throughput than power state 2, and power states
0 through 3 all have the same read latency.  Relative performance ordering is only with respect to a single
performance characteristic. Thus, although the relative read throughput value of one power state may equal
the relative write throughput value of another power state, this does not imply that the actual read and write
performance of these two power states are equal.
The default NVM Express power state is implementation specific and shall correspond to a state that does
not consume more power than the lowest value specified in the applicable form factor specification, if any.
Refer to the Power Management section in the applicable NVM Express transport specification for transport
specific power requirements impacting NVMe power states, if any.
 Non-Operational Power States
A power state may be a non -operational power state, as indicated by Non -Operational State (NOPS) field
in Figure 313. Non-operational power states allow the following operations:
• property accesses;
• PMR accesses, if any;
