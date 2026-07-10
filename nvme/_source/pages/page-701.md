---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 701
headings: ["B.4 Asynchronous Event Request Host Software Recommendations"]
---

# Source page 701

NVM Express® Base Specification, Revision 2.1

679
• CMD1.NSID is set to identify the appropriate namespace. This value shall be the same as
CMD0.NSID;
• If metadata is being used in a separate buffer, then the location of that buffer is specified in the
CMD1.MPTR field;
• The physical address of the first page of data to write is identified:
o If the command uses PRPs, then CMD1.PRP1 is set to the physical address of the first
page of the data to write and CMD1.PRP2 is set to the physical address of the PRP List.
The PRP List includes three entries; or
o If the command uses SGLs, CMD1.SGL1 is set to an appropriate SGL segment descriptor
depending on whether more than one descriptor is needed;
• CMD1.CDW10.SLBA is set to the first LBA to write. Note that this field also spans Command Dword
11. This value shall be the same as CMD0.CDW10.SLBA;
• CMD1.CDW12.LR is cleared to ‘0’ to indicate that the controller should apply all available error
recovery means to write the data to the NVM;
• CMD1.CDW12.FUA is cleared to ‘0’, indicating that the data may be written to any location,
including a volatile cache, in the NVM subsystem;
• CMD1.CDW12.PRINFO is cleared to 0h since end-to-end protection is not enabled;
• CMD1.CDW12.NLB is set to 3h, indicating that four logical blocks of a size of 4 KiB each are to be
compared against. This value shall be the same as CMD0.CDW12.NLB;
• CMD1.CDW14 is cleared to 0h since end-to-end protection is not enabled; and
• CMD1.CDW15 is cleared to 0h since end-to-end protection is not enabled.
Host software then completes a transport specific action in order to submit the command for processing.
Note that the transport specific submit action  shall indicate both commands have been submitted at one
time.
B.4 Asynchronous Event Request Host Software Recommendations
This section describes the recommended host software procedure for Asynchronous Event Requests.
The host sends n Asynchronous Event Request commands (refer to section  3.5.1, step 12). When an
Asynchronous Event Request completes (providing Event Type, Event Information, and Log Page details):
• If the event(s) in the reported Log Page may be disabled with the Asynchronous Event
Configuration feature (refer to section  5.1.25.1.5), then host software issues a Set Features
command for the Asynchronous Event Configuration feature specifying to disable reporting of all
events that utilize the Log Page reported. Host software should wait for the Set Features command
to complete;
• Host software issues a Get Log Page command requesting the Log Page reported as part of the
Asynchronous Event Command completion. Host software should wait for the Get Log Page
command to complete;
• Host software parses the returned Log Page. If the condition is not persistent, then host software
should re-enable all asynchronous events that utilize the Log Page. If the condition is persistent,
then host software should re -enable all asynchronous even ts that utilize the Log Page except for
the one(s) reported in the Log Page. The host re -enables events by issuing a Set Features
command for the Asynchronous Event Configuration feature;
• Host software should issue an Asynchronous Event Request command to the controller (restoring
to n the number of these commands outstanding); and
• If the reporting of event(s) was disabled, host software should enable reporting of the event(s) using
the Asynchronous Event Configuration feature. If the condition reported may persist, host software
should continue to monitor the event (e.g., spare belo w threshold) to determine if reporting of the
event should be re-enabled.
