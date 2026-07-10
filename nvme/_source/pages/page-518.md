---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 518
headings: ["8.1.6.2 Controller Data Queue Phase Tag", "8.1.7 Device Self-test Operations"]
---

# Source page 518

NVM Express® Base Specification, Revision 2.1

496
• the Enable Tail Pointer Trigger (ETPT) bit is set to ‘1’.
A CDQ entry has been consumed by the host when the host submits a Set Features command specifying
the Controller Data Queue feature with a new value that indicates that the CDQ Head Pointer has moved
past the slot in which that CDQ entry was placed. A Set Features command specifying the Controller Data
Queue feature may indicate that one or more CDQ entries have been consumed.
Altering a CDQ entry after that entry has been posted but before that entry has been consumed results in
undefined behavior.
Refer to the specific type of CDQ to determine the behavior of a CDQ that has become full.
Refer to section 3.3.1.4 for the definition of an empty CDQ. Refer to section  3.3.1.5 for the definition of a
full CDQ.
If a CQD entry is constructed via multiple writes, the Controller Data Queue Phase Tag (CDQP) bit shall be
updated in the last write of that CDQ entry.
Refer to the applicable I/O Command Set specifications for any specific requirements on the use of CDQs
 Controller Data Queue Phase Tag
The Controller Data Queue Phase Tag (CDQP) bit indicates whether a CDQ entry is new. The CDQP bit
for each CDQ entry in the CDQ shall be initialized to ‘0’ by the host before creating the CDQ by submitting
a Controller Data Queue command (refer to section  5.1.4) specifying the Create Queue management
operation (refer to section 5.1.4.1.1).
When the controller posts a new CDQ entry to the CDQ, the controller shall invert the CDQP bit in that CDQ
entry (i.e., the inverting of the CDQP bit enables the host to detect the new CDQ entry).
When a CDQ entry is posted to a CDQ slot in the CDQ for the first time after the CDQ is created, the CDQP
bit for that completion queue entry is set to ‘1’.
This continues for each CDQ entry that is posted until the controller posts a CDQ entry to the highest
numbered CDQ slot and wraps to CDQ slot number 0 as described in section 8.1.6.1. When that CDQ wrap
condition occurs, the CDQP bit is then cleared to ‘0’ in each CDQ entry that is posted. This continues until
another CDQ wrap condition occurs. Each time a CDQ wrap condition occurs, the value of the CDQP bit is
inverted (i.e., changes from ‘1’ to ‘0’ or changes from ‘0’ to ‘1’).
 Device Self-test Operations
A device self-test operation is a diagnostic testing sequence that tests the integrity and functionality of the
controller and may include testing of the media associated with namespaces  or refresh operations . The
operation is broken down into a series of segments, where each segment is a set of vendor specific tests
or refresh operations. The segment number in the Self-test Result data structure (refer to section 5.1.12.1.7)
is used for reporting purposes to indicate where a test failed, if any. The test performed in each segment
may be the same for the short device self-test operation and the extended device self-test operation.
A device self -test operation is performed in the background allowing concurrent processing of some
commands and requiring suspension of the device self -test operation to process other commands. Which
commands may be processed concurrently versus require su spension of the device self -test operation is
vendor specific.
If the controller receives any command that requires suspension of the device self-test operation to process
and complete, then the controller shall:
1) suspend the device self-test operation;
2) process and complete that command; and
3) resume the device self-test operation.
During a device self -test operation, the performance of the NVM subsystem may be degraded (e.g.,
controllers not performing the device self-test operation may also experience degraded performance).
The following device self-test operations are defined:
