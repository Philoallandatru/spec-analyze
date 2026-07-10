---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 234
headings: []
---

# Source page 234

NVM Express® Base Specification, Revision 2.1

212
Figure 206: SMART / Health Information Log Page
Bytes Description
207:206 Temperature Sensor 4 (TSEN4): Contains the current temperature reported by temperature sensor 4.
This field is defined by Figure 207.
209:208 Temperature Sensor 5 (TSEN5): Contains the current temperature reported by temperature sensor 5.
This field is defined by Figure 207.
211:210 Temperature Sensor 6 (TSEN6): Contains the current temperature reported by temperature sensor 6.
This field is defined by Figure 207.
213:212 Temperature Sensor 7 (TSEN7): Contains the current temperature reported by temperature sensor 7.
This field is defined by Figure 207.
215:214 Temperature Sensor 8 (TSEN8): Contains the current temperature reported by temperature sensor 8.
This field is defined by Figure 207.
219:216
Thermal Management Temperature 1 Transition Count (TMT1TC): Contains the number of times the
controller transitioned to lower power active power states or performed vendor specific thermal
management actions while minimizing the impact on performance in order to attempt to reduce the
Composite Temperature because of the host controlled thermal management feature ( refer to section
8.1.17.5) (i.e., the Composite Temperature rose above the Thermal Management Temperature 1). This
counter shall not wrap once the value FFFFFFFFh is reached. A value of 0h, indicates that this transition
has never occurred or this field is not implemented.
223:220
Thermal Management Temperature 2 Transition Count (TMT2TC): Contains the number of times the
controller transitioned to lower power active power states or performed vendor specific thermal
management actions regardless of the impact on performance (e.g., heavy throttling) in order to attempt
to reduce the Composite Temperature because of the host controlled thermal management feature (refer
to section 8.1.17.5) (i.e., the Composite Temperature rose above the Thermal Management
Temperature 2). This counter shall not wrap once the value FFFFFFFFh is reached . A value of 0h,
indicates that this transition has never occurred or this field is not implemented.
227:224
Total Time For Thermal Management Temperature 1  (TTTMT1): Contains the number of seconds
that the controller had transitioned to lower power active power states or performed vendor specific
thermal management actions while minimizing the impact on performance in order to attempt to reduce
the Composite Temperatur e because of the host controlled thermal management feature ( refer to
section 8.1.17.5). This counter shall not wrap once the value FFFFFFFFh is reached . A value of 0h,
indicates that this transition has never occurred or this field is not implemented.
231:228
Total Time For Thermal Management Temperature 2  (TTTMT2): Contains the number of seconds
that the controller had transitioned to lower power active power states or performed vendor specific
thermal management actions regardless of the impact on performance (e.g., heavy throttling) in order
to attempt to reduce t he Composite Temperature because of the host controlled thermal management
feature (refer to section 8.1.17.5). This counter shall not wrap once the value FFFFFFFFh is reached. A
value of 0h, indicates that this transition has never occurred or this field is not implemented.
511:232 Reserved

Figure 207: Temperature Sensor Data Structure
Bits Description
15:00
Temperature Sensor Temperature (TST): Contains the current temperature in Kelvins reported by the
temperature sensor.
The physical point in the NVM subsystem whose temperature is reported by the temperature sensor
and the temperature accuracy is implementation specific. An implementation that does not implement
the temperature sensor reports a value of 0h. The temperature reported by a temperature sensor may
be used to trigger an asynchronous event (refer to section 5.1.25.1.3).
