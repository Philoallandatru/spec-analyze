---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 280
headings: []
---

# Source page 280

NVM Express® Base Specification, Revision 2.1

258
Figure 258: Endurance Group Configuration Descriptor
Bytes Description
63:48
Endurance Estimate (EE): This field is an estimate of the total number of data bytes that may
be written to the Endurance Group over the lifetime of the Endurance Group assuming a write
amplification of 1 (i.e., no increase in the number of write operations performed by the devic e
beyond the number of write operations requested by a host). This value is reported in billions
(i.e., a value of 1h corresponds to 1,000,000,000 bytes written) and is rounded up (e.g., a value
of 1h indicates the number of bytes written is from 1 to 1,000,000,000, 2h indicates the number
of bytes written is from 1,000,000,001 to 2,000,000,000).
A value of FFFFFFFF_FFFFFFFF_FFFFFFFF_FFFFFFFFh means that value and all higher
values.
A value of 0h indicates that the Endurance Estimate is not reported.
The relationship between this value and the value in the Endurance Estimate field in the
Endurance Group Information log page (refer to section 5.1.12.1.10) is outside the scope of
this specification.
79:64 Reserved
NVM Set Information
81:80
Number of NVM Sets (EGSETS):  This field indicates the number of NVM Set Identifiers in
this Endurance Group Configuration Descriptor. A value of 0h indicates that no NVM Set
Identifiers are reported for this Endurance Group.
NVM Set Identifier List
83:82 NVM Set 0 Identifier: This field indicates the identifier of the first NVM Set assigned to this
Endurance Group, if reported. Refer to section 3.2.2.
85:84 NVM Set 1 Identifier: This field indicates the identifier of the second NVM Set assigned to this
Endurance Group, if reported.
… …
(EGSETS*2)+81 :
(EGSETS*2)+80
NVM Set EGSETS-1 Identifier: This field indicates the identifier of the last NVM Set assigned
to this Endurance Group, if reported.
Channel Configuration Information
(EGSETS*2)+83 :
(EGSETS*2)+82
Number of Channels (EGCHANS): This field indicates the number of Channel Configuration
Descriptors in this Endurance Group Configuration Descriptor. If this field is cleared to 0h, then
no Channel Configuration Descriptors are reported for this Endurance Group.
Channel Configuration Descriptor List
NOTE 1 Channel 0 Configuration Descriptor:  This field contains the Channel Configuration
Descriptor (refer to Figure 259) for the first Channel in this Endurance Group, if any.
NOTE 1 Channel 1 Configuration Descriptor:  This field contains the Channel Configuration
Descriptor for the second Channel in this Endurance Group, if any.
… …
NOTE 1 Channel EGCHANS -1 Configuration Descriptor:  This field contains the Channel
Configuration Descriptor for the last Channel in this Endurance Group, if any.
Notes:
1. Channel Configuration Descriptors may be different lengths.
The Channel Configuration Descriptor (refer to Figure 259) lists the Media Units attached to a Channel.
Media Unit Configuration Descriptors (refer to Figure 260) shall be listed in ascending order by Media Unit
Identifier, and each Media Unit Identifier shall appear only once.
Figure 259: Channel Configuration Descriptor
Bytes Description
Header
1:0 Channel Identifier (CHID): This field indicates the identifier of this Channel. A value of FFFFh indicates
that the Channel Identifier is not specified.
3:2
Number of Channel Media Units (CHMUS):  This field indicates the number of Media Units that are
attached to this Channel. If this field is cleared to 0h, then no Media Unit Configuration Descriptors are
reported for this Channel.
