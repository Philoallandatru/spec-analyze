---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 140
headings: ["3.7.2 Controller Level Reset"]
---

# Source page 140

NVM Express® Base Specification, Revision 2.1

118
The occurrence of an NVM Subsystem Reset while power is applied to the domain is reported by the initial
value of the CSTS.NSSRO field following the NVM Subsystem Reset. This field may be used by host
software to determine if the sudden loss of communication with a controller was due to an NVM Subsystem
Reset or some other condition.
The ability for host software to initiate an NVM Subsystem Reset by writing to the NSSR.NSSRC field is an
optional capability of a controller indicated by the state of the CAP.NSSRS field. An implementation may
protect the domain from an inadvertent NVM Su bsystem Reset by not providing this capability to one or
more controllers that are in the domain.
 Controller Level Reset
The following methods initiate a Controller Level Reset:
• NVM Subsystem Reset;
• Controller Reset (i.e., host clears the CC.EN bit from ‘1’ to ‘0’); and
• Transport specific reset types (refer to the applicable NVMe Transport binding specification), if any.
A Controller Level Reset consists of the following actions:
• The controller stops processing any outstanding Admin or I/O commands;
• All I/O Submission Queues are deleted;
• All I/O Completion Queues are deleted;
• The controller is brought to an i dle state. When this is complete, the CSTS.RDY bit is cleared to
‘0’; and
• All NVMe controller properties defined in either section 3.1.4 or the applicable NVMe Transport
binding specification and all internal controller state are reset, with the following exceptions:
o for memory-based controllers:
▪ the following are not reset as part of a Controller Level Reset caused by a Controller Reset:
• Admin Queue properties (i.e., AQA, ASQ, and ACQ);
• Persistent Memory Region properties (i.e., PMRCAP, PMRCTL, PMRSTS, PMREBS,
PMRSWTP, PMRMSCU, and PMRMSCL); and
• The Controller Memory Buffer Memory Space Control property (CMBMSC)  (refer to
the NVM Express NVMe over PCIe Transport Specification);
and
▪ the following are not reset as part of a Controller Level Reset caused by a Function Level
Reset:
• the Controller Memory Buffer Memory Space Control property (CMBMSC);
and
o for message-based controllers:
▪ there are no exceptions.
In all Controller Level Reset cases except a Controller Reset, the controller properties defined by the
transport (e.g., the PCIe registers defined by the PCIe Base Specification) are reset as defined by the
applicable NVMe Transport binding specification (e.g., the NVM Express NVMe over PCIe Transport
Specification).
In all Controller Level Reset cases, if the media is not usable and an NVM Subsystem Shutdown that
includes the controller is neither reported as in progress nor reported as complete (i.e., the CSTS.ST bit is
cleared to ‘0’ or the CSTS.SHST field is cleare d to 00b), then the controller is permitted to initialize the
media for use upon completion of the Controller Level Reset.
To continue after a Controller Level Reset, the host should:
• update transport specific state and controller property state as appropriate;
