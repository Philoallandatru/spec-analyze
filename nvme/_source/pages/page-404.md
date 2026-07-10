---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 404
headings: ["5.1.25.1.10 Non-Operational Power State Config (Feature Identifier 11h)"]
---

# Source page 404

NVM Express® Base Specification, Revision 2.1

382
Figure 398: HCTM – Command Dword 11
Bits Description
31:16
Thermal Management Temperature 1 (TMT1):  This field specifies the temperature, in Kelvins, when
the controller begins to transition to lower power active power states or performs vendor specific thermal
management actions while minimizing the impact on performance (e.g., light throttling) in order to attempt
to reduce the Composite Temperature.
A value cleared to 0h, specifies that this part of the Feature shall be disabled.
The range of values that are supported by the controller are indicated in the Minimum Thermal
Management Temperature field and Maximum Thermal Management Temperature field in the Identify
Controller data structure in Figure 312.
If the host attempts to set this field to a value less than the value contained in the Minimum Thermal
Management Temperature field or greater than the value contained in the Maximum Thermal
Management Temperature field in the Identify Controller data structure in Figure 312, then the command
shall abort with a status code of Invalid Field in Command.
If the host attempts to set this field to a value greater than or equal to the value contained in the Thermal
Management Temperature 2 field, if non-zero, then the command shall abort with a status code of Invalid
Field in Command.
15:00
Thermal Management Temperature 2 (TMT2):  This field specifies the temperature, in Kelvin s, when
the controller begins to transition to lower power active power states or perform vendor specific thermal
management actions regardless of the impact on performance (e.g., heavy throttling) in order to attempt
to reduce the Composite Temperature.
A value cleared to 0h, specifies that this part of the Feature shall be disabled.
The range of values that are supported by the controller are indicated in the Minimum Thermal
Management Temperature field and Maximum Thermal Management Temperature field in the Identify
Controller data structure in Figure 312.
If the host attempts to set this field to a value less than the value contained in the Minimum Thermal
Management Temperature field or greater than the value contained in the Maximum Thermal
Management Temperature field in the Identify Controller data structure in Figure 312, then the command
shall abort with a status code of Invalid Field in Command.
If the host attempts to set this field to a non -zero value less than or equal to the value contained in the
Thermal Management Temperature 1 field, then the command shall abort with a status code of Invalid
Field in Command.
5.1.25.1.10 Non-Operational Power State Config (Feature Identifier 11h)
This Feature configures non -operational power state settings for the controller. The settings are specified
in Command Dword 11.
If a Get Features command is submitted for this Feature, the values in Figure 399 are returned in Dword 0
of the completion queue entry for that command.
Figure 399: Non-Operational Power State Config – Command Dword 11
Bits Description
31:01 Reserved
