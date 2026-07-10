---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 254
headings: []
---

# Source page 254

NVM Express® Base Specification, Revision 2.1

232
The host is expected to issue a Get Log Page command with the Action field set to 02h to release the
persistent event log reporting context after reading the persistent event log page data.
If the Persistent Event Log is not read with a single Get Log Page command, then host software should
read the Generation Number field in the Persistent Event Log header after establishing a reporting context
but before reading the remainder of the log and then re-read the Generation Number field after it has read
the entire log. If the generation numbers do not match, then:
• the reporting context may have been lost while reading the log;
• the Persistent Event Log contents read may be invalid; and
• host software should re-read the log.
