---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 342
headings: []
---

# Source page 342

NVM Express® Base Specification, Revision 2.1

320
Figure 312: Identify – Identify Controller Data Structure, I/O Command Set Independent
Bytes I/O1 Admin1 Disc1 Description
5
Reservations Support (RESERVS): If this bit is set to ‘1’, then the controller
supports reservations. If this bit is cleared to ‘0’, then the controller does not
support reservations. If the controller supports reservations, then the
following commands associated with reservations shall be supported:
Reservation Report, Reservation Register, Reservation Acquire, and
Reservation Release. Refer to section 8.1.22 for additional requirements.
4
Save and Select Feature Support (SSFS): If this bit is set to ‘1’, then the
controller supports the Save field set to a non-zero value in the Set Features
command and the Select field set to a non -zero value in the Get Features
command. If this bit is cleared to ‘0’, then the controller does not support the
Save field set to a non -zero value in the Set Features command and  does
not support the Select field set to a non -zero value in the Get Features
command.
3
Write Zeroes Support Variants (NVMWZSV): If this bit is  set to ‘1’, then
the controller supports the NVM Command Set Write Zeroes command and
the Write Zeroes Size Limit (WZSL) field indicates the recommended
maximum data size for Write Zeroes commands. If this bit is cleared to ‘0’,
then controller support of the NVM Command Set Write Zeroes command
is indicated by a non-zero data size limit in the WZSL field.
2
Dataset Management Support Variants (NVMDSMSV): If this bit is set to
‘1’, then the controller supports the NVM Command Set Dataset
Management command and limits, if any, on controller support of the
Dataset Management command are indicated by non -zero values in the
Dataset Management Ranges Limit (DMRL) field, the Dataset Management
Size Limit (DMSL) field and the Dataset Management Range Size Limit
(DMRSL) field. If this bit is cleared to ‘0’, then controller support of the NVM
Command Set Dataset Managem ent command is indicated by a non -zero
data size limit in the DMRL, DMSL, and DMRSL fields.
1
Write Uncorrectable Support Variants (NVMWUSV): If this bit is set to ‘1’,
then the controller supports the NVM Command Set Write Uncorrectable
command and the Write Uncorrectable Size Limit (WUSL) field indicates the
recommended maximum data size for Write Uncorrectable commands. If
this bit is cleared to ‘0’, then controller support of the NVM Command Set
Write Uncorrectable command is indicated by a non -zero data size limit in
the WUSL field.
0
Compare Command Support (NVMCMPS): If this bit is set to ‘1’, then the
controller supports the NVM Command Set Compare command. If this bit is
cleared to ‘0’, then the controller does not support the NVM Command Set
Compare command.
NOTE: This field applies to all I/O Command Sets. The original name has been retained
for historical continuity.
523:522 M M R
Fused Operation Support (FUSES): This field indicates the fused operations that the
controller supports. Refer to section 3.4.2.
Bits Description
7:1 Reserved
0
Fused Compare and Write Supported (FCWS): If this bit is set to ‘1’, then
the controller supports the NVM Command Set Compare and Write fused
operation. If this bit is cleared to ‘0’, then the controller does not support the
NVM Command Set Compare and Write fused operation. Compare shall be
the first command in the sequence.
