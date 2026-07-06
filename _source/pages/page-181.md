---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 181
headings: ["4.3.3 Metadata Region (MR)", "4.4 Feature Values"]
---

# Source page 181

NVM Express® Base Specification, Revision 2.1

159
 Metadata Region (MR)
The definition for the Metadata Region is command set specific. Refer to each I/O Command Set
specification for applicability and additional details, if any.
4.4 Feature Values
The Get Features command (refer to section  5.1.11) and Set Features command (refer to section  5.1.25)
may be used to read and modify operating parameters of the controller. The operating parameters are
grouped and identified by Feature Identifiers. Each Feature Identifier contains one or more attributes that
may affect the behavior of the Feature.
If the Save and Select Feature Support (SSFS) bit is set to ‘1’ in the Optional NVM Command Support
(ONCS) field of the Identify Controller data structure in Figure 312, then for each Feature, there are three
settings: default, saved, and current. If the SSFS bit is cleared to ‘0’, then the controller only supports a
current and default value for each Feature. In this case, the current value may be persistent across power
cycles and resets based on the information specified in Figure 385.
If the SSFS bit is set to ‘1’, then e ach Feature has supported capabilities (refer to Figure 195), which are
discovered using the Supported Capabilities value in the Select field in Get Features (refer to Figure 192).
The default value for each Feature is vendor spec ific and set by the manufacturer unless otherwise
specified. The default value is not changeable.
The current value for a Feature is the value in active use by the controller for that Feature.
A Set Features command uses the value specified by the command to set:
a) the current value for that Feature; or
b) the current value for that Feature and the saved value for that Feature, if that Feature is saveable.
A Feature may be saveable. If a Feature is saveable (i.e., the controller supports the Save field in the Set
Features command and the Select field in the Get Features command), then any Feature Identifier that is
namespace specific may be saved on a per namespace basis.
If a Feature is not saveable, or does not have a saved value, then a Get Features command to read the
saved value returns the default value.
If a Feature is not saveable and is persistent as specified in Figure 385, then the current value of that
Feature shall be persistent across power cycles and resets.
The current value for a Feature, as a result of a Controller Level Reset, is indicated in Figure 124 for an
NVM subsystem that supports only a single controller (i.e., the Multiple Ports bit of the CMIC field (refer to
Figure 311) is cleared to ‘0’).
The current value for a Feature, as a result of an NVM Subsystem Reset, is indicated in Figure 124, for an
NVM subsystem:
• that is able to support two or more controllers and does not support multiple domains ; or
• supports multiple domains where the NVM Subsystem Reset results in a reset of the entire NVM
subsystem.
