---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 576
headings: []
---

# Source page 576

NVM Express® Base Specification, Revision 2.1

554
If the MAC in the request and the calculated MAC are equal, then the controller compares the Write Counter
in the request with the Write Counter stored in the controller. If the counters are different, then the controller
sets the result to 0003h (counter failure) and no data is written to the RPMB data area.
If the MAC and Write Counter comparisons are successful, then the write request is authenticated. The
Data from the request is written to the Address indicated in the request and the Write Counter is
incremented by one.
If the write fails, then the returned result is 0005h (write failure). If another error occurs during the write
procedure, then the returned result is 0001h (general failure).
The controller returns a successful completion for the Security Send command when the Authenticated
Data Write operation is completed regardless of whether the Authenticated Data Write was successful or
not.
The success of programming the data should be checked by the host by reading the result property of the
RPMB:
1) The host initiates the Authenticated Data Write verification process by issuing a Security Send
command with delivery of a RPMB data frame containing the Request Message Type = 0005h;
2) The controller returns a successful completion of the Security Send command when the verification
result is ready for retrieval;
3) The host should then retrieve the verification result by issuing a Security Receive command; and
4) The controller returns a successful completion of the Security Receive command and returns the
RPMB data frame containing the Response Message Type = 0300h, the incremented counter
value, the data address, the MAC and result of the data programming operati on.
