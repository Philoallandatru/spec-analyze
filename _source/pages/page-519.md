---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 519
headings: ["8.1.7.1 Short Device Self-Test Operation"]
---

# Source page 519

NVM Express® Base Specification, Revision 2.1

497
a) short device self-test operation (refer to section 8.1.7.1);
b) extended device self-test operation (refer to section 8.1.7.2); and
c) Host-Initiated Refresh operation (refer to section 8.1.11).
Figure 595 is an informative example of a device self-test operation with the associated segments and tests
performed in each segment.
Figure 595: Example Device Self-test Operation (Informative)
Segment Test Performed Failure Criteria
1 – RAM Check Write a test pattern to RAM, followed by a read and compare
of the original data.
Any uncorrectable error or
data miscompare
2 – SMART Check Check SMART or health status for Critical Warning bits set
to ‘1’ in SMART / Health Information Log.
Any Critical Warning bit set to
‘1’ fails this segment
3 – Volatile memory
backup
Validate volatile memory backup solution health (e.g.,
measure backup power source charge and/or discharge
time).
Significant degradation in
backup capability
4 – Metadata validation  Confirm/validate all copies of metadata. Metadata is corrupt and is not
recoverable
5 – NVM integrity
Write/read/compare to reserved areas of each NVM. Ensure
also that every read/write channel of the controller is
exercised.
Data miscompare
Extended only
6 – Data Integrity
Perform background housekeeping tasks, prioritizing
actions that enhance the integrity of stored data.
Exit this segment in time to complete the remaining
segments and meet the timing requirements for extended
device self-test operation indicated in the Identify Controller
data structure.
Metadata is corrupt and is not
recoverable
7 – Media Check
Perform random reads from every available good physical
block.
Exit this segment in time to complete the remaining
segments. The time to complete is dependent on the type of
device self-test operation.
Inability to access a physical
block
8 – Drive Life End-of-life condition: Assess the drive’s suitability for
continuing write operations.
The Percentage Used is set to
255 in the SMART / Health
Information Log or an analysis
of internal key operating
parameters indicates that data
is at risk if writing continues
9 – SMART Check Same as 2 – SMART Check
 Short Device Self-Test Operation
A short device self-test operation should complete in two minutes or less. The percentage complete of the
short device self-test operation is indicated in the Current Percentage Complete field in the Device Self-test
log page (refer to section 5.1.12.1.7).
A short device self-test operation:
a) shall be aborted by any Controller Level Reset that affects the controller on which the device self-
test is being performed;
b) shall be aborted by a Format NVM command as described in Figure 596;
c) shall be aborted when a sanitize operation is started (refer to section 5.1.22);
d) shall be aborted if a Device Self-test command with the Self-Test Code field set to Fh is processed;
and
e) may be aborted if the specified namespace is removed from the namespace inventory.
