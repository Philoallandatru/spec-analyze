---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 162
headings: []
---

# Source page 162

NVM Express® Base Specification, Revision 2.1

140
Figure 100: Completion Queue Entry: Status Field
Bits Description
31
Do Not Retry (DNR): If this bit is set to ‘1’, then this bit indicates that if the same command is re-submitted
to any controller in the NVM subsystem, then that re -submitted command is expected to fail. If this bit is
cleared to ‘0’, then this bit indicates that the same command may succeed if retried. If a command is
aborted due to time limited error recovery (refer to the Error Recovery section in the NVM Command Set
Specification), this bit should be cleared to ‘0’. If the SCT and SC fields are c leared to 0h, then this bit
should be cleared to ‘0’.
30
More (M): If this bit is set to ‘1’, then there is more status information for this command as part of the Error
Information log page that may be retrieved with the Get Log Page command. If this bit is cleared to ‘0’,
then there is no additional status information for this command. Refer to section 5.1.12.1.2.
29:28
Command Retry Delay (CRD):  If the DNR bit is cleared to ‘0’ and the host has set the Advanced
Command Retry Enable (ACRE) field to 1h in the Host Behavior Support feature (refer to section
5.1.25.1.14), then:
a) a 00b CRD value indicates a command retry delay time of zero (i.e., the host may retry the
command immediately); and
b) a non-zero CRD value selects a field in the Identify Controller data structure (refer to Figure 312)
that indicates the command retry delay time:
• a 01b CRD value selects the Command Retry Delay Time 1 (CRDT1) field;
• a 10b CRD value selects the Command Retry Delay Time 2 (CRDT2) field; and
• a 11b CRD value selects the Command Retry Delay Time 3 (CRDT3) field.
The host should not retry the command until at least the amount of time indicated by the selected field has
elapsed. It is not an error for the host to retry the command prior to that time.
If the DNR bit is set to’1’ in the Status field or the ACRE field is cleared to 0h in the Host Behavior Support
feature, then this field is reserved.
If the SCT and SC fields are cleared to 0h, then this field should be cleared to 00b.
27:25 Status Code Type (SCT): Indicates the status code type of the completion queue entry. This indicates the
type of status code the controller is returning.
24:17 Status Code (SC): Indicates a status code identifying any error or status information for the command
indicated.
Completion queue entries indicate a Status Code Type (SCT) for the type of completion being reported.
Figure 101 specifies the status code type values and descriptions.
Figure 101: Status Code – Status Code Type Values
Value Description Reference
0h
Generic Command Status: Indicates that the command specified by the Command and
Submission Queue identifiers in the completion queue entry has completed. These status
values are generic across all command types, and include such conditions as success,
opcode not supported, and invalid field.
4.2.3.1
1h
Command Specific Status: Indicates a status value that is specific to a particular
command opcode. These values may indicate additional processing is required. Status
values such as invalid firmware image or exceeded maximum number of queues is
reported with this type.
4.2.3.2
2h Media and Data Integrity Errors: Any media specific errors that occur in the NVM or data
integrity type errors shall be of this type. 4.2.3.3
3h
Path Related Status:  Indicates that the command specified by the Command and
Submission Queue identifier in the completion queue entry has completed. These status
values are generic across all command types. These values may indicate that additional
process is required and indicate a status value that is specific to:
a) the connection between the host and the controller processing the command; or
b) the characteristics that support Asymmetric Namespace Access Reporting (refer
to section 8.1.1), the characteristics of the relationship between the controller
processing the command and the specified namespace.
4.2.3.4
4h to 6h Reserved
