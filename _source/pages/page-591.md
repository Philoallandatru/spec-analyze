---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 591
headings: ["8.1.24.2 Sanitize Operation Types and Support"]
---

# Source page 591

NVM Express® Base Specification, Revision 2.1

569
• fails to deallocate all media allocated for user data, then the sanitization target enters the Restricted
Failure state or the Unrestricted Failure state, as described in section 8.1.24.3.7.
A Controller Level Reset may cause the sanitize operation not to include the Media Verification state and
the Post-Verification Deallocation state, as described in section 8.1.24.3.
 Sanitize Operation Types and Support
The scope of a sanitize operation is all locations in the NVM subsystem that are able to contain user data,
including caches, Persistent Memory Regions, and unallocated or deallocated areas of the media.
If the composition of the NVM subsystem (refer to section 3.2.5) changes (e.g., a new domain is added, or
a division event occurs) and that change prevents the successful completion of a sanitize operation, then
the sanitize operation shall fail.
If the composition of the NVM subsystem changes (e.g., a new domain is added, or a division event occurs)
and that change prevents verification of media allocated for user data, then the Media Verification state is
canceled and the MVCNCLD bit is set to ‘1’.
Sanitize operations do not affect the Replay Protected Memory Block, boot partitions, or other media and
caches that do not contain user data. A sanitize operation also may alter log pages and features as
necessary (e.g., to prevent derivation of user data from log page information  or feature information ). A
sanitize operation is only able to be started if the NVM subsystem is not divided (refer to section 3.2.5). A
sanitize operation in the Restricted Processing state, the Unrestricted Processing state, the Media
Verification state, or the Post-Verification Deallocation state is not able to be aborted and continues after a
Controller Level Reset , including across power cycles.  Refer to Annex A  for further information about
sanitize operations.
The Sanitize command (refer to section 5.1.22) is use d to start a sanitize operation, to recover from a
previously failed sanitize operation , or to exit the Media Verification state . All sanitize operations are
performed in the background (i.e., completion of the Sanitize command that starts a sanitize operation does
not indicate completion of that sanitize operation).  The completion of a sanitize operation and the optional
transition into the Media Verification state are indicated in the Sanitize Status log page, and with:
• the Sanitize Operation Completed asynchronous event,
• the Sanitize Operation Completed With Unexpected Deallocation asynchronous event, or
• the Sanitize Operation Entered Media Verification State asynchronous event.
If the Sanitize command that started a sanitize operation was submitted to a controller’s Admin Submission
Queue, then the asynchronous event shall be reported only by that controller. If the Sanitize command that
started a sanitize operation was submitted  to a Management Endpoint (refer to the NVM Express
Management Interface Specification), then the asynchronous event shall not be reported by any controller
in the NVM subsystem.
The Sanitize Capabilities (SANICAP) field of the Identify Controller data structure (refer to Figure 312)
indicates the sanitize operation types supported and controller attributes specific to sanitize operations.
The sanitize operation types are:
• the Block Erase sanitize operation, which alters user data with a low-level block erase method that
is specific to the media for all locations on the media within the sanitization target  in which user
data may be stored;
• the Crypto Erase sanitize operation, which alters user data by changing the media encryption keys
for all locations on the media within the sanitization target in which user data may be stored; and
• the Overwrite sanitize operation , which alters user data by writing a fixed data pattern or related
patterns one or more times to all locations on the media within the sanitization target in which user
data may be stored. Figure 647 defines the data pattern or patterns that are written.
Controller attributes specific to sanitize operations include:
• the No-Deallocate Modifies Media After Sanitize (NODMMAS) field, which indicates whether media
is modified by the controller as part of sanitize processing that had been requested with the No -
