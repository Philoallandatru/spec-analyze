---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 294
headings: ["5.1.12.1.28 Flexible Data Placement (FDP) Configurations (Log Page Identifier 20h)"]
---

# Source page 294

NVM Express® Base Specification, Revision 2.1

272
5.1.12.1.28 Flexible Data Placement (FDP) Configurations (Log Page Identifier 20h)
The FDP Configurations log page (refer to Figure 279) identifies a list of static FDP configurations that are
allowed to be applied to the specified Endurance Group. The Endurance Group Identifier in the Log Specific
Identifier field as defined in Figure 217 specifies the Endurance Group. The creation of an Endurance Group
or the enablement of an FDP configuration in an Endurance Group within the NVM subsystem may affect
which FDP configurations are currently allowed (refer to the FDP Configuration Valid bit in Figure 280).
Refer to section 8.1.10 for the usage of this log page by the host.
Figure 279: FDP Configurations Log Page
Bytes Description
Header
01:00 Number of FDP Configurations (N UMFDPC): This field indicates the number of FDP
Configuration Descriptors contained in this log page. This is a 0’s based number.
02 Version (VER): This field indicates the version of the log page that includes the header and the
FDP Configuration Descriptor List. This field shall be cleared to 0h.
03 Reserved
07:04 Size (SZE): This field identifies the size of this log page in bytes.
15:08 Reserved
FDP Configuration Descriptor List
Variable Sized:
16
FDP Configuration Descriptor 0: The FDP Configuration Descriptor (refer to Figure 280) of the
first configuration.
Variable Sized FDP Configuration Descriptor 1: The FDP Configuration Descriptor (refer to Figure 280) of the
second configuration, if any.
… …
Variable Sized FDP Configuration Descriptor NUMFDPC-1: The FDP Configuration Descriptor (refer to Figure
280) of the last configuration, if any.
Figure 280 defines the format of an FDP Configuration Descriptor. The Reclaim Unit Handles in the Reclaim
Unit Handle list shall be listed in ascending order of Reclaim Unit Handle Identifier.
Figure 280: FDP Configuration Descriptor
Bytes Description
01:00 Descriptor Size  (DSZE): This field indicates the size in bytes of this FDP Configuration
Descriptor.
02
FDP Attributes (FDPA): This field identifies attributes of this FDP configuration.
Bits Description
7
FDP Configuration Valid (FDPCV): This bit is set to ‘1’ to indicate that this FDP
configuration is valid. This bit is cleared to ‘0’ to indicate that this entry is not
currently available and the host should ignore this FDP configuration.
6:5 Reserved
4
FDP Volatile Write Cache (FDPVWC): If this bit is set to ‘1’, then a volatile write
cache is present for this FDP configuration. If this bit is cleared to ‘0’, then a volatile
write cache is not present for this FDP configuration. Refer to section 5.1.25.1.4.
If the controller reports one or more FDP configurations with this bit set to ‘1’, then
the Volatile Write Cache Present (VWCP) bit in the Identify Controller data structure
(refer to Figure 312) shall be set to ‘1’. This results in the VWCP bit being set to ‘1’
even though there is no volatile write cache in the controller to support hosts not
aware of the Flexible Data Placement capability (refer to section 8.1.10).
Refer to section 7.2 to determine if a volatile write cache is present if a namespace
exists in an Endurance Group that has Flexible Data Placement enabled.
3:0
Reclaim Group Identifier Format (RGIF): This field identifies the number of most-
significant bits in the Placement Identifier that contain the Reclaim Group Identifier
(refer to Figure 282 and Figure 283). If the NRG field in this data structure is set to
1h (i.e., there is a single Reclaim Group), then this field may be cleared to 0h. If the
NRG field is greater than 1h, then this field shall be a non-zero value.
