---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 560
headings: []
---

# Source page 560

NVM Express® Base Specification, Revision 2.1

538
The host specifies and enables the thermal management requirements by setting the Thermal Management
Temperature 1 field and/or Thermal Management Temperature 2 field ( refer to section 5.1.25.1.9) in a Set
Features command to a non -zero value. The supported range of values for the Thermal Management
Temperature 1 field and Thermal Management Temperature 2 field are indicated in the Identify Controller
data structure in Figure 312.
The Thermal Management Temperature 1 specifies that if the Composite Temperature (refer to Figure 207)
is:
a) greater than or equal to this value; and
b) less than the Thermal Management Temperature 2, if non-zero,
then the controller should start transitioning to lower power active power states or perform vendor specific
thermal management actions while minimizing the impact on performance in order to attempt to reduce the
Composite Temperature (e.g., transition to an active power state that performs light throttling).
The Thermal Management Temperature 2 field specifies that if the Composite Temperature is greater than
or equal to  this value, then the controller shall start transitioning to lower power active power states or
perform vendor specific thermal management actions regardless of the impact on performance in order to
attempt to reduce the Composite Temperature (e.g., transition to an active power state that performs heavy
throttling).
If the controller is currently in a lower power active power state or performing vendor specific thermal
management actions because of this feature (e.g., throttling performance) because the Composite
Temperature is:
a) greater than or equal to the current value of the Thermal Management Temperature 1 field; and
b) less than the current value of the Thermal Management Temperature 2 field,
and the Composite Temperature decreases to a value below the current value of the Thermal Management
Temperature 1 field, then the controller should return to the active power state that the controller was in
prior to going to a lower power active power state or stop performing vendor specific thermal management
actions because of this feature, the Composite Temperature and the current value of the Thermal
Management Temperature 1 field.
If the controller is currently in a lower power active power state or performing vendor specific thermal
management actions because the Composite Temperature is greater than or equal to the current value of
the Thermal Management Temperature 2 field and the Composite Temperature decreases to a value less
than the current value of the Thermal Management Temperature 1 field, then the controller should return
to the active power state that the controller was in prior to going to a lower power active power state or stop
performing vendor specific thermal management actions because of this feature, and the Composite
Temperature.
The temperature at which the controller stops being in a lower power active power state or performing
vendor specific thermal management actions because of this feature is vendor specific (i.e., hysteresis is
vendor specific).
Figure 627 shows examples of how the Composite Temperature may be affected by this feature.
