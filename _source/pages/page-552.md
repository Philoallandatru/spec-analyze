---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 552
headings: ["8.1.15.2 Namespace Deletion Asynchronous Event Reporting"]
---

# Source page 552

NVM Express® Base Specification, Revision 2.1

530
6. if Allocated Namespace Attribute Notices are enabled on the controller, then the host requests the
Identify Namespace data structures (refer to section 1.5.49) for the new namespace to determine
all attributes of the namespace upon receiving an Allocated Namespace Attribute Changed
asynchronous event from that controller.
The following is an example of how a host attaches a namespace:
1. the host issues the Namespace Attachment command specifying the Controller Attach operation
to attach the specified namespace to one or more controllers;
2. the host receives an Attached Namespace Attribute Changed asynchronous event from each
controller that has a namespace newly attached to that controller if Attached Namespace Attribute
Notices are enabled on that controller; and
3. the host receives an Allocated Namespace Attribute Changed asynchronous event from each
controller that has Allocated Namespace Attribute Notices enabled.
The following is an example of how a host detaches a namespace:
1. the host issues the Namespace Attachment command specifying the Controller Detach operation
to detach the specified namespace from one or more controllers;
2. the host receives an Attached Namespace Attribute Changed asynchronous event from each
controller that has a namespace newly detached from that controller  if Attached Namespace
Attribute Notices are enabled on that controller; and
3. the host receives an Allocated Namespace Attribute Changed asynchronous event from each
controller that has Allocated Namespace Attribute Notices enabled.
The following is an example of how a host deletes a namespace:
1. the host should detach the namespace from all controllers;
2. the host issues the Namespace Management command specifying the Delete operation for the
specified namespace. On successful completion of the command, the namespace has been
deleted;
3. if the namespace was attached to any controller(s) when deleted:
a. the host receives an Attached Namespace Attribute Changed asynchronous event from
each of those controllers that has Attached Namespace Attribute Notices enabled as
described in section 8.1.15.2; and
b. the host receives an Allocated Namespace Attribute Changed asynchronous event from
each of those controllers that has Allocated Namespace Attribute Notices enabled as
described in section 8.1.15.2;
and
4. if the namespace was not attached to any controller when deleted, the host receives an Allocated
Namespace Attribute Changed asynchronous event from each controller that has Allocated
Namespace Attribute Notices enabled as described in section 8.1.15.2.
 Namespace Deletion Asynchronous Event Reporting
If a delete operation is requested for a namespace:
• via a command (e.g., a Namespace Management command or Capacity Management command)
received on the Admin Submission Queue (e.g., not via a Management Endpoint, refer to the NVM
Express Management Interface Specification), then:
o each controller, to which the namespace is attached, that has  Attached Namespace
Attribute Notices enabled (refer to Figure 391) other than the controller processing that
delete operation, shall issue an  Attached Namespace Attribute  Changed asynchronous
event as part of the delete operation to indicate a namespace change; and
o each controller that has Allocated Namespace Attribute Notices enabled (refer to Figure
391) other than the controller processing that delete operation, shall issue an Allocated
Namespace Attribute Changed asynchronous event as part of the delete operation to
indicate a namespace change;
