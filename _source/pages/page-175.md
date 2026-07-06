---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 175
headings: ["4.3.2 Scatter Gather List (SGL)"]
---

# Source page 175

NVM Express® Base Specification, Revision 2.1

153
 Scatter Gather List (SGL)
A Scatter Gather List (SGL) is a data structure in memory address space used to describe a data buffer.
The controller indicates the SGL types that the controller supports in the Identify Controller data structure.
A data buffer is either a source buffer or a destination buffer. An SGL contains one or more SGL segments.
The total length of the Data Block and Bit Bucket descriptors in an SGL shall be equal to or exceed the
amount of data requested to be transferred. If the amount of data requested to be transferred exceeds the
total length of the Data Block and Bit Bucket descriptors in an SGL, data shall not be transferred to or from
locations that are not described by the SGL.
An SGL segment is a qword aligned data structure in a contiguous region of physical memory describing
all, part of, or none of a data buffer and the next SGL segment, if any. An SGL segment consists of an array
of one or more SGL descriptors. Only the last  descriptor in an SGL segment may be an SGL Segment
descriptor or an SGL Last Segment descriptor.
A last SGL segment is an SGL segment that does not contain an SGL Segment descriptor, or an SGL Last
Segment descriptor.
A controller may support byte or dword alignment and granularity of Data Blocks. If a controller supports
only dword alignment and granularity as indicated in the SGL Support field of the Identify Controller data
structure (refer to Figure 312), then the values in the Address and Length fields of all Data Block descriptors
shall have their two least significant bits cleared to 00b. This requirement applies to Data Block descriptors
that indicate data and/or metadata memory regions.
The SGL Descriptor Threshold (SDT) field in the Identify Controller data structure (refer to Figure 312)
indicates the recommended maximum number of SGL descriptors for a command. If the SDT field is set to
a non-zero value, and a command is submitted for which the sum of:
a) the number of SGL Bit Bucket descriptors with non-zero Length field contents; and
b) the number of SGL Data Block descriptors with a non-zero Length field contents,
exceeds the value of the SDT field, then the performance of the controller may be reduced.
The value of the SDT field shall be less than or equal to the value of the Maximum SGL Data Block
Descriptors (MSDBD) field in the Identify Controller data structure (refer to Figure 312 for the definition of
the MSDBD field).
A Keyed SGL Data Block descriptor is a Data Block descriptor that includes a key that is used as part of
the host memory access. The maximum length that may be specified in a Keyed SGL Data Block descriptor
is (16 MiB – 1).
A Transport SGL Data Block descriptor is a Data Block descriptor that specifies a data block that is
transferred by the NVMe Transport using a transfer mechanism and data buffers that are specific to the
NVMe Transport.
The SGL Identifier Descriptor Sub Type field may indicate additional information about a descriptor. As an
example, the Sub Type may indicate that the Address field is an offset rather than an absolute address.
The Sub Type may also indicate NVMe Transport specific information.
The controller shall abort a command if:
• an SGL segment contains an SGL Segment descriptor or an SGL Last Segment descriptor in other
than the last descriptor in the segment;
• a last SGL segment contains an SGL Segment descriptor, or an SGL Last Segment descriptor ; or
• an SGL descriptor has an unsupported format.
The controller should abort a command if:
• an SGL Data Block descriptor contains Address or Length fields with either of the two least
significant bits set to 1b and the controller supports only dword alignment and granularity as
indicated in the SGL Support field of the Identify Controller data structure. Refer to Figure 312.
Figure 113 defines the SGL segment.
