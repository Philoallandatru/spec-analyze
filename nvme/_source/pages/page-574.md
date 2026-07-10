---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 574
headings: ["8.1.21.2 RPMB Operations", "8.1.21.2.1 Authentication Key Programming"]
---

# Source page 574

NVM Express® Base Specification, Revision 2.1

552
generate a MAC using HMAC SHA -256 is the 256 -bit Authentication Key stored in the controller for the
selected RPMB target. The HMAC SHA -256 calculation takes as input a key and a message. Input to the
MAC calculation is the concatenation of the fields in the RPMB Data Frame (request or response) excluding
stuff bytes and the MAC itself – i.e., bytes [223:255] and Data of the frame in that order.
 RPMB Operations
The host sends a Request Message Type to the controller to request an operation by the controller or to
deliver data to be written into the RPMB memory block. To deliver a Request Message Type, the host uses
the Security Send command. If the data to be del ivered to the controller is more than reported in Identify
Controller data structure, the host sends multiple Security Send commands to transfer the entire data.
The host sends a Response Message Type to the controller to read the result of a previous operation
request, to read the Write Counter, or to read data from the RPMB memory block. To deliver a Response
Message Type, the host uses the Security Receive comma nd. If the data to be read from the controller is
more than reported in Identify Controller data structure, the host sends multiple Security Receive
commands to transfer the entire data.
8.1.21.2.1 Authentication Key Programming
Authentication Key programming is initiated by a Security Send command to program the Authentication
Key to the specified RPMB target, followed by a subsequent Security Send command to request the result,
and lastly, the host issues a Security Receive command to retrieve the result.
Figure 638: RPMB – Authentication Key Data Flow
Command Bytes in Command Field Name Value Objective
Security
Send 1
Data populated by the host and sent to the controller
Send
Authentication
Key to be
Programmed to
the controller
222-N:00 Stuff Bytes 0…00h
222:222-(N-1) MAC/Key Key to be
programmed
223 RPMB Target RPMB target to
access
239:224 Nonce 0…00h
243:240 Write Counter 00000000h
247:244 Address 00000000h
251:248 Sector Count 00000000h
253:252 Result 0000h
255:254 Request/Response 0001h (Request)
Security
Send 2
Data populated by the host and sent to the controller
Request Result of
Key
Programming
222-N:00 Stuff Bytes 0…00h
222:222-(N-1) MAC/Key 0…00h
223 RPMB Target RPMB target to
access
239:224 Nonce 0…00h
243:240 Write Counter 00000000h
247:244 Address 00000000h
251:248 Sector Count 00000000h
253:252 Result 0000h
255:254 Request/Response 0005h (Request)
Security
Receive 1
Data populated by the controller and returned to the host
Retrieve the Key
Programming
Result
222-N:00 Stuff Bytes 0…00h
222:222-(N-1) MAC/Key 0…00h
223
RPMB Target RPMB target
response was sent
from
239:224 Nonce 0…00h
243:240 Write Counter 00000000h
247:244 Address 00000000h
