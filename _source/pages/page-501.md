---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 501
headings: ["8.1.1.6 ANA Inaccessible state", "8.1.1.7 ANA Persistent Loss state", "8.1.1.8 ANA Change state"]
---

# Source page 501

NVM Express® Base Specification, Revision 2.1

479
 ANA Inaccessible state
While the relationship between the controller and an ANA group is in this state, the characteristic of that
relationship to each namespace in that group is inaccessible. Commands processed by a controller that
reports this state for an ANA Group are not ab le to access user data of namespaces in that ANA Group.
The namespaces may become accessible through the controller reporting this state at a future time (i.e., a
subsequent ANA state transition may occur). Support for reporting this state is optional.
While in this state, accurate namespace related capacity information may not be available. As a result,
some namespace capacity information returned in the Identify Namespace data structure (e.g., the NUSE
field and the NVMCAP field), are cleared to 0h. For that namespace capacity information, hosts should use
the Identify Namespace data structure returned from a controller that reports the relationship between the
controller and the namespace to be in the ANA Optimized state or in the ANA Non -Optimized state.
A controller shall abort commands, other than those described in section 8.1.1.10, with a status code of
Asymmetric Access Inaccessible if those commands are submitted while the relationship between the
specified namespace and the controller processing the command is in this state.
While ANA Inaccessible state is reported by a controller for the namespace, the host should retry the
command on a different controller that is reporting ANA Optimized state or ANA Non-Optimized state. If no
controllers are reporting ANA Optimized state or  ANA Non -Optimized state, then a transition may be
occurring such that a controller reporting the Inaccessible state may become accessible and the host should
retry the command on the controller reporting Inaccessible state for at least ANATT seconds (refer to Figure
312). Refer to section 8.1.2.2.
 ANA Persistent Loss state
While the relationship between the controller and an ANA group is in this state, the characteristic of that
relationship to each namespace in that group is persistently inaccessible. Commands processed by a
controller that reports this state for an ANA Gro up are persistently not able to access user data of
namespaces in that ANA Group. The relationship between a controller and an ANA Group in this state shall
not transition to any other ANA state. Support for reporting this state is optional.
While in this state, accurate namespace related capacity information may not be available. As a result,
some namespace capacity information returned in the Identify Namespace data structure (e.g., the NUSE
field and the NVMCAP field), are cleared to 0h. For that namespace capacity information, hosts should use
the Identify Namespace data structure returned from a controller that reports the relationship between the
controller and the namespace to be in the ANA Optimized state or in the ANA Non -Optimized state.
A controller shall abort commands, other than those described in section 8.1.1.10, with a status code of
Asymmetric Access Persistent Loss if those commands are submitted while the relationship between the
specified namespace and the controller processing the command is in this state.
While ANA Persistent Loss state is reported by a controller for the namespace, the host should retry the
command on a different controller that is reporting ANA Optimized state or ANA Non-Optimized state. If no
controllers are reporting ANA Optimized state  or ANA Non -Optimized state, then a transition may be
occurring such that a controller reporting the Inaccessible state may become accessible and the host should
retry the command on the controller reporting Inaccessible state for at least ANATT seconds (refer to Figure
312).
 ANA Change state
The change from one asymmetric namespace access state to another asymmetric namespace access state
is called a transition. Transitions may occur in such a way that the ANA Change state is not visible to the
host (i.e., the ANA Change state may or may not be reported in the Asymmetric Namespace Access State
field in the Asymmetric Namespace Access log  page (refer to section 5.1.12.1.13)). Support for reporting
this state is optional.
A controller shall abort commands, other than those described in 8.1.1.10, with a status code of Asymmetric
Access Transition if those commands are submitted while the relationship between the specified
namespace and the controller processing the command is in this state.
