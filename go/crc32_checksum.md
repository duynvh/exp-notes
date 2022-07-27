# CRC32 Checksum - how it works
1. Definition
The cyclic redundancy check (CRC) is a technique used to detect errors in digital data. As a type of checksum, the CRC produces a fixed-length data set based on the build of a file or larger data set.

For IEEE802.3, CRC-32. Think of the entire message as a serial bit stream, append 32 zeros to the end of the message. Next, you MUST reverse the bits of EVERY byte of the message and do a 1's complement the first 32 bits. Now divide by the CRC-32 polynomial, 0x104C11DB7. Finally, you must 1's complement the 32-bit remainder of this division bit-reverse each of the 4 bytes of the remainder. This becomes the 32-bit CRC that is appended to the end of the message.

The reason for this strange procedure is that the first Ethernet implementations would serialize the message one byte at a time and transmit the least significant bit of every byte first. The serial bit stream then went through a serial CRC-32 shift register computation, which was simply complemented and sent out on the wire after the message was completed. The reason for complementing the first 32 bits of the message is so that you don't get an all zero CRC even if the message was all zeros.

2. How it works
-  The polynomial for CRC32 is:
> x32 + x26 + x23 + x22 + x16 + x12 + x11 + x10 + x8 + x7 + x5 + x4 + x2 + x + 1

Or in hex:
> 0x 01 04 C1 1D B7  

Or in binary:
> 1 0000 0100 1100 0001 0001 1101 1011 0111

- Note: Why this polynomial? Because there needs to be a standard given polynomial and the standard was set by IEEE 802.3. Also it is extremely difficult to find a polynomial that detects different bit errors effectively.

- How to calculate crc32:
```
calculate the CRC-32 hash for the 'ANSI' string 'abc':

inputs:
dividend: binary for 'abc': 0b011000010110001001100011 = 0x616263
polynomial: 0b100000100110000010001110110110111 = 0x104C11DB7

start with the 3 bytes 'abc':
61 62 63 (as hex)
01100001 01100010 01100011 (as bin)

reverse the bits in each byte:
10000110 01000110 11000110

append 32 0 bits:
10000110010001101100011000000000000000000000000000000000

XOR (exclusive or) the first 4 bytes with 0xFFFFFFFF:
(i.e. flip the first 32 bits:)
01111001101110010011100111111111000000000000000000000000

next we will perform 'CRC division':

a simple description of 'CRC division':
we put a 33-bit box around the start of a binary number,
start of process:
if the first bit is 1, we XOR the number with the polynomial,
if the first bit is 0, we do nothing,
we then move the 33-bit box right by 1 bit,
if we have reached the end of the number,
then the 33-bit box contains the 'remainder',
otherwise we go back to 'start of process'

note: every time we perform a XOR, the number begins with a 1 bit,
and the polynomial always begins with a 1 bit,
1 XORed with 1 gives 0, so the resulting number will always begin with a 0 bit

'CRC division':
'divide' by the polynomial 0x104C11DB7:
01111001101110010011100111111111000000000000000000000000
 100000100110000010001110110110111
 ---------------------------------
  111000100010010111111010010010110
  100000100110000010001110110110111
  ---------------------------------
   110000001000101011101001001000010
   100000100110000010001110110110111
   ---------------------------------
    100001011101010011001111111101010
    100000100110000010001110110110111
    ---------------------------------
         111101101000100000100101110100000
         100000100110000010001110110110111
         ---------------------------------
          111010011101000101010110000101110
          100000100110000010001110110110111
          ---------------------------------
           110101110110001110110001100110010
           100000100110000010001110110110111
           ---------------------------------
            101010100000011001111110100001010
            100000100110000010001110110110111
            ---------------------------------
              101000011001101111000001011110100
              100000100110000010001110110110111
              ---------------------------------
                100011111110110100111110100001100
                100000100110000010001110110110111
                ---------------------------------
                    110110001101101100000101110110000
                    100000100110000010001110110110111
                    ---------------------------------
                     101101010111011100010110000001110
                     100000100110000010001110110110111
                     ---------------------------------
                       110111000101111001100011011100100
                       100000100110000010001110110110111
                       ---------------------------------
                        10111100011111011101101101010011

we obtain the 32-bit remainder:
0b10111100011111011101101101010011 = 0xBC7DDB53

note: the remainder is a 32-bit number, it may start with a 1 bit or a 0 bit

XOR the remainder with 0xFFFFFFFF:
(i.e. flip the 32 bits:)
0b01000011100000100010010010101100 = 0x438224AC

reverse bits:
bit-reverse the 4 bytes (32 bits), treating them as one entity:
(e.g. 'abcdefgh ijklmnop qrstuvwx yzABCDEF'
to 'FEDCBAzy xwvutsrq ponmlkji hgfedcba':)
0b00110101001001000100000111000010 = 0x352441C2

thus the CRC-32 hash for the 'ANSI' string 'abc' is: 0x352441C2
```


3. References:
- https://stackoverflow.com/questions/2587766/how-is-a-crc32-checksum-calculated
- https://vi.wikipedia.org/wiki/Cyclic_Redundancy_Check
- https://en.wikipedia.org/wiki/Cyclic_redundancy_check
- https://www.techopedia.com/definition/1793/cyclic-redundancy-check-crc
