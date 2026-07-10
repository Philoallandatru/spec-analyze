---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 564
headings: ["8.1.18.2 Configuring Periodic Windows"]
---

# Source page 564

NVM Express® Base Specification, Revision 2.1

542
Figure 630: Typical and Reliable Estimate Example

The host configures the current window to be either DTWIN or NDWIN using a Set Features command with
the Predictable Latency Mode Window Feature. The host may use the reliable estimates provided in the
Predictable Latency Mode log page to ensure that the h ost transitions the NVM Set to the NDWIN prior to
any reliable estimates exceeding one of the typical or maximum values (e.g., DTWIN Reads Estimate = 0).
The reliable estimates provided shall have the following properties when in the Deterministic Window:
• The estimates shall be monotonically decreasing towards 0h for the entirety of the DTWIN,
depending on the attribute. For example, DTWIN Reads Reliable Estimate is monotonically
decreasing and thus does not increase without transitioning from the DTWIN to the NDWIN; and
• The estimates shall not change abruptly unless operating conditions have changed abruptly. The
estimate should be based on averaging or smoothing of data collected over some period of time.
 Configuring Periodic Windows
When using the NVM Set in Predictable Latency Mode, the host should transition the controller to NDWIN
for periodic maintenance. The maintenance is required in order for the NVM subsystem to reliably provide
the amount of time indicated for Deterministic Windows.
There are three static time based parameters reported in the Predictable Latency Per NVM Set log page
(refer to section 5.1.12.1.11) that may be used by the host to configure periodic windows. The values
provided are worst-case for the life of the NVM subsystem:
• NDWIN Time Minimum Low is the minimum time that the NVM Set remains in the Non-Deterministic
Window. The controller may delay completion of a Set Features command requesting a transition
to the Deterministic Window until this time is completed. This time d oes not account for additional
host activity in the Non-Deterministic Window;
• NDWIN Time Minimum High is the minimum time that the host should allow the NVM Set to remain
in the Non-Deterministic Window after the NVM Set remained in the previous Deterministic Window
for DTWIN Time Maximum. This time does not account for additional h ost activity in the Non -
Deterministic Window; and
• DTWIN Time Maximum is the maximum time that the NVM Set is able to stay in a Deterministic
Window.
