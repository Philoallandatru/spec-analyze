---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 27
headings: ["1.4.3 Byte, Word, and Dword Relationships", "1.5 Definitions", "1.5.1 admin label", "1.5.2 admin label ASCII"]
---

# Source page 27

NVM Express® Base Specification, Revision 2.1

5
 Byte, Word, and Dword Relationships
Figure 3 illustrates the relationship between bytes, words and dwords. A qword (quadruple word) is a unit
of data that is four times the size of a word; it is not illustrated due to space constraints. Unless otherwise
specified, this specification specifies data in a little endian format.
Figure 3: Byte, Word, and Dword Relationships

7

6

5

4

3

2

1

0


                                byte


                1
5
1
4
1
3
1
2
1
1
1
0
0
9
0
8
0
7
0
6
0
5
0
4
0
3
0
2
0
1
0
0


                                word



                byte 1 byte 0


3
1
3
0
2
9
2
8
2
7
2
6
2
5
2
4
2
3
2
2
2
1
2
0
1
9
1
8
1
7
1
6
1
5
1
4
1
3
1
2
1
1
1
0
0
9
0
8
0
7
0
6
0
5
0
4
0
3
0
2
0
1
0
0


                                dword



word 1 word 0



byte 3 byte 2 byte 1 byte 0

1.5 Definitions
 admin label
An admin label is an administratively configured ASCII or UTF-8 string (refer to section 1.4.2) that may be
used to help identify specific NVMe entities (i.e., Hosts, NVM subsystems and namespaces). An admin
label is capable of describing the entity’s physical location, DNS name or other information.
 admin label ASCII
An ASCII string. Refer to section 1.4.2 for ASCII string requirements. Refer to section 1.5.1 for admin label
usage.
