---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 25
headings: ["1.4.1.4 optional", "1.4.1.5 R", "1.4.1.6 reserved", "1.4.1.7 shall", "1.4.1.8 should", "1.4.2 Numerical Descriptions"]
---

# Source page 25

NVM Express® Base Specification, Revision 2.1

3
 optional
A keyword that describes features that are not required by this specification. However, if any optional
feature defined by the specification is implemented, the feature shall be implemented in the way defined by
the specification.
 R
“R” is used as an abbreviation for “reserved” when the figure or table does not provide sufficient space for
the full word “reserved”.
 reserved
A keyword referring to bits, bytes, words, fields, and opcode values that are set -aside for future
standardization. Their use and interpretation may be specified by future extensions to this or other
specifications. A reserved bit, byte, word, field, property, or register shall be cleared to 0h, or in accordance
with a future extension to this specification. The recipient of a command or a register write is not required
to check reserved bits, bytes, words, or fields. Receipt of reserved coded values in defined fields in
commands shall be reported as an error. Writing a reserved coded value into a controller property field
produces undefined results.
 shall
A keyword indicating a mandatory requirement. Designers are required to implement all such mandatory
requirements to ensure interoperability with other products that conform to the specification.
 should
A keyword indicating flexibility of choice with a strongly preferred alternative. Equivalent to the phrase “it is
recommended”.
 Numerical Descriptions
A 0’s based value is a numbering scheme in which the number 0h represents a value of 1h, 1h represents
2h, 2h represents 3h, etc. In this numbering scheme, there is no method to represent the value of 0h.
Values in this specification are 1 -based (i.e., the  number 1h represents a value of 1h, 2h represents 2h,
etc.) unless otherwise specified.
Size values are shown in binary units or decimal units. The symbols used to represent these values are as
shown in Figure 2.
Figure 2: Decimal and Binary Units
Decimal Binary
Symbol Power
(base-10) Symbol Power
(base-2)
kilo / k 103 kibi / Ki 210
mega / M 106 mebi / Mi 220
giga / G 109 gibi / Gi 230
tera / T 1012 tebi / Ti 240
peta / P 1015 pebi / Pi 250
exa / E 1018 exbi / Ei 260
zetta / Z 1021 zebi / Zi 270
yotta / Y 1024 yobi / Yi 280
The ^ operator is used to denote the power to which that number, symbol, or expression is to be raised.
Some parameters are defined as an ASCII string. ASCII strings shall contain only code values  (i.e., byte
values or octet values)  20h through 7Eh. For the string “Copyright”, the character “C” is the first byte, the
character “o” is the second byte, etc. ASCII strings are left justified. If padding is necessary, then the string
