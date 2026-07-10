---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 589
headings: ["8.1.24 Sanitize Operations", "8.1.24.1 Elements of Sanitize Operations"]
---

# Source page 589

NVM Express® Base Specification, Revision 2.1

567
a) set the Rotational Media bit to ‘1’ in the NSFEAT field of the I/O Command Set Independent Identify
Namespace data structure (refer to the NVM Command Set Specification) for any namespace that
stores data on rotational media;
b) support the Rotational Media Information log page (refer to section 5.1.12.1.22);
c) support the Spinup Control feature (refer to section 5.1.25.1.18);
d) support Endurance Groups (refer to section 3.2.3); and
e) set the EG Rotational Media bit to ‘1’ in the EGFEAT field in the Endurance Group Information log
page for each Endurance Group that stores data on rotational media.
If a namespace that stores data on rotational media is attached to a controller, and the spindle used by that
namespace is not spinning, then that controller shall be in a non-operational power state (i.e., NOPS is set
to ‘1’, refer to Figure 313).
If:
a) a domain contains an Endurance Group that stores data on rotational media;
b) that domain processes an NVM Subsystem Reset; and
c) the Spinup Control feature (refer to section 5.1.25.1.18) is:
a. disabled, then initial spinup for all such Endurance Groups in that domain shall be initiated;
and
b. enabled, then initial spinup for all such Endurance Groups in that domain shall be inhibited
during processing of the NVM Subsystem Reset  until any controller within that domain
processes a Set Features (Power Management) command that specifies an operational
power state.
If the PCIe transport is used for a controller, then the PCIe Slot Power Control feature may affect the power
states supported (refer to the PCI Express Base Specification).
 Sanitize Operations
A sanitize operation alters all user data in the sanitization target such that recovery of any previous user
data from any cache, the non-volatile storage media, or any Controller Memory Buffer is not possible. It is
implementation specific whether Submission Queues and Completion Queues within a Controller Memory
Buffer are altered by a sanitize operation; all other data stored in all Controller Memory Buff ers is altered
by a sanitize operation. If a portion of the user data was not altered and the sanitiz e operation completed
successfully, then the NVM subsystem shall ensure permanent inaccessibility of that portion of the media
allocated for user data for any future use within the NVM subsystem (e.g., retrieval from NVM media,
caches, or any Controller Me mory Buffer) and permanent inaccessibility of that portion of the media
allocated for user data via any interface to the NVM subsystem, including management interfaces as
defined by the NVM Express Management Interface Specification.
 Elements of Sanitize Operations
A sanitize operation consists of:
• sanitize processing, which may include:
o deallocation of all media allocated for user data; and
o additional media modification;
• optional verification of media allocated for user data; and
• post-verification deallocation of all media allocated for user data following media verification, if any.
Sanitize processing is performed in either restricted completion mode (i.e., in the Restricted Processing
state) or in unrestricted completion mode (i.e., in the Unrestricted Processing state), as specified by the
Allow Unrestricted Sanitize Exit (AUSE) bit in the Sanitize command (refer to Figure 372).
Additional media modification may be performed as part of sanitize processing (i.e., in the Restricted
Processing state or the Unrestricted Processing state) to prevent commands that access media after
completion of sanitize processing from encountering da ta integrity errors caused by that sanitize
processing.
