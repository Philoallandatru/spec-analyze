---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 696
headings: ["Annex A. Sanitize Operation Considerations (Informative)", "A.1 Overview", "A.2 Hidden Storage (Overprovisioning)", "A.3 Integrity checks and No-Deallocate After Sanitize"]
---

# Source page 696

NVM Express® Base Specification, Revision 2.1

674
Annex A. Sanitize Operation Considerations (Informative)
A.1 Overview
The Sanitize command starts a sanitize operation that makes all user data previously written to the
sanitization target inaccessible (i.e., all user data has been purged, as defined in IEEE Std 2883 -2022). It
is very difficult to prove that the sanitize operation successfully purged all user data . This annex provides
some context and considerations for understanding the result of the operation and the practical limitations
for auditing the result of the sanitize operation.
A.2 Hidden Storage (Overprovisioning)
Sanitize operations purge all physical storage in the sanitization target that is able to hold user data. Many
NVMe SSDs contain more physical storage than is addressable through the interface ( e.g.,
overprovisioning). That physical storage is used for vendor specific purposes which may include providing
increasing endurance, improving performance, and providing extra capacity to allow retiring bad or worn -
out storage without affecting capacity. This excess capacity as well as any retired storage are not accessible
through the interface. Vendor specific innovative use of this extra capacity supports advantages to the end
user, but the lack of observability makes it difficult to ensure that all storage within the sanitization target
was correctly purged . Only the accessible storage is able to be audited for the results of a sanitization
operation.
A.3 Integrity checks and No-Deallocate After Sanitize
Another issue is availability of the data returned through the interface. Some of the sanitize operations (e.g.,
Block Erase) affect the physical devices in such a way that directly reading the accessible storage may
trigger internal integrity checks resulting in error responses instead of returning the contents of the storage.
Other sanitize operations (e.g., Crypto Erase) may scramble the vendor specific internal format of the data,
also resulting in error responses instead of returning the contents of the storage.
To compensate for these issues, a controller may perform additional internal write operations to media that
is able to be allocated for user data (i.e., additional media modification, refer to section 8.1.24.1) so that all
media that is allocated for user data is readable without error. However, this has the side effect of potentially
significant additional wear on the media as well as the side effect of obscuring the results of the initial
sanitize operation (i.e., the writes destroy the ability to forensically audit the result of the initial sanitize
operation). Given this side effect, process audits of sanitize behavior on ly provide effective results when
the No-Deallocate After Sanitize bit is set the same way (e.g., set to ‘1’) for both process audits and the
individual forensic device audits.
The Sanitize command introduced in NVM Express Base Specification, Revision 1.3 included a mechanism
to specify that sanitized media allocated for user data not be deallocated, thereby allowing observations of
the results of the sanitization operation. However, some architectures and products (e.g., integrity checking
circuitry) interact with this capability in such a way as to defeat the sanitize result obser vability purpose.
New features were added to NVM Express Base Specification, Revision  1.4 that include extended
information about the sanitization capabilities of devices, a new asynchronous event, and configuration of
the response to No -Deallocate After Sanitize requests. These features are intended to both support new
systems that understand the new capabilities, as well to help manage legacy systems that do not
understand the new capabilities without losing the ability to sanitize as requested.
These issues in returning the contents of accessible storage do not apply if the sanitization target is in the
Media Verification state (refer to section 8.1.24.3.6). In that state, failure of internal integrity checks do not
cause error responses to Read commands (refer to the Media Verification section of the NVM Command
Set Specification). Because the Sanitize command that caused entry to the Media Verification st ate
specified the Enter Media Verification State (EMVS) bit set to ‘1’, the controller does not perform the
additional media modification described in this section.
