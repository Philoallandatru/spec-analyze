---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 706
headings: ["Annex C. Power Management and Consumption (Informative)"]
---

# Source page 706

NVM Express® Base Specification, Revision 2.1

684
Annex C. Power Management and Consumption (Informative)
NVM Express power management capabilities allow the host to manage power for a controller. Power
management includes both control and reporting mechanisms.
For information on transport power management (e.g., PCIe, RDMA), refer to the applicable NVM Express
transport specification.
The scope of NVM Express power management is the controller (refer to section 5.1.25.1.2).
NVM Express power management uses the following functionality:
a) Features:
• Power Management (refer to section 5.1.25.1.2 and section 8.1.17);
• Autonomous Power State Transition (refer to section 5.1.25.1.6 and section 8.1.17.2);
• Non-Operational Power State Configuration (refer to section 5.1.25.1.10 and section 8.1.17.1);
and
• Spinup Control (refer to section 5.1.25.1.18);
b) NVM subsystem workloads (refer to 8.1.17.3); and
c) Runtime D3 transitions (refer to section 8.1.17.4).
Controller thermal management may cause a transition to a lower power state, interacting with these
Features:
a) Temperature Threshold (refer to section 5.1.25.1.3);
b) Host Controlled Thermal Management (refer to section 5.1.25.1.9 and section 8.1.17.5); and
c) access to host memory buffer (refer to section  5.1.25.2.4) may be prohibited in non -operational
power state.
NVM Express power management uses these reporting mechanisms:
a) properties:
• Controller Power Scope (CAP.CPS) (refer to Figure 36);
b) fields in the Identify Controller data structure (refer to Figure 312):
• RTD3 Resume Latency (RTD3R);
• RTD3 Entry Latency (RTD3E);
• Non-Operational Power State Permissive Mode;
• Number of Power States Support (NPSS);
• Autonomous Power State Transition Attributes (APSTA); and
• Power State 0 Descriptor (PSD0) through Power State 31 Descriptor (PSD31) (refer to Figure
313);
c) Features:
• Power Management (refer to section 5.1.25.1.2);
• Temperature Threshold (refer to section 5.1.25.1.3);
• Autonomous Power State Transition (refer to section 5.1.25.1.6 and section 8.1.17.2);
• Non-Operational Power State Configuration (refer to section 5.1.25.1.10 and section 8.1.17.1);
• Host Controlled Thermal Management (refer to section 5.1.25.1.9 and section 8.1.17.5);
• Host Memory Buffer (refer to section5.1.25.2.4); and
• Spinup Control (refer to section 5.1.25.1.18);
and
d) log pages:
• SMART / Health Information log page fields (refer to section 5.1.12.1.3):
o Thermal Management Temperature [1-2] Transition Count; and
