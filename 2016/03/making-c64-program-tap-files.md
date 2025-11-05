# Making C64 Program Tap Files

Sunday, March 20, 2016

---

Before I go ahead and tell you guys how to make a tap file that is loadable in a C64 emulator, I'm going to give you the link to the source of all my info: http://c64tapes.org/dokuwiki/doku.php?id=loaders:rom_loader and http://c64tapes.org/dokuwiki/doku.php?id=analyzing_loaders

This is where I learned how to make tap files (although it took me two weeks to find that page).

Enough chat, here is the structure of a C64 tap file. \
There are 3 different kinds of pulses:
* a long pulse lasting about 672 µS represented as ASCII char 86 or hex 56
* a medium pulse lasting about 512 µS represented as ASCII char 66 or hex 42
* a short pulse lasting about 352 µS represented as ASCII char 48 or hex 30

Physically, these pulses are square wave pulses running at 50% duty with a period of the specified µS.
I will refer to long as **L**, medium as **M**, and small as **S**. Data encoded using a combination of these pulses will be referred to as **LMS** format.

Now these pulses alone don't make up binary data (as I learned the hard way). \
Bits are represented as a pair of pulses, a 0 bit being **SM**, a 1 bit being **MS**, a start bit being **LM**, and a data end bit being a **LS**. \
A **LMS** byte is represented as a certain sequence of pulses which looks like this:
```
LM     xx    xx    xx    xx    xx    xx    xx    xx    nn
start  bit0  bit1  bit2  bit3  bit4  bit5  bit6  bit7  check
```
There is a start bit in front of every byte, but each byte doesn't have its own end bit. The check bit at the end is calculated with the bitwise formula:
```
1 xor bit0 xor bit1 xor bit2 xor bit3 xor bit4 xor bit5 xor bit6 xor bit7
```
Or more conveniently, if the byte has an even number of 1's, the check bit is 1, if the byte has an odd number of 1's, the check bit is 0.

Now that single bytes are covered, it's time to learn how whole data is encoded in a tap file.

Every C64 tap file starts off with 20 bytes of information, kind of like meta data. All information should be coded in hex, as this is raw data. \
Bytes 0 to 11 are C64-TAPE-RAW, byte 12 is the number 1, bytes 13 to 15 are just 0, and bytes 16 to 19 is the length of the actual tap data in least significant byte first (**LSBF**) format. The tap data length is usually added last, since we don't know how long our tap data is yet.

The rest of the bytes are the data. \
We start with 27136 small (S) pulses, this is the **leader block** used to calibrate the datasette speed. \
Then we start the header with 9 **sync bytes**, from 137 down to 129 (**LMS**). \
Then we write the **header file type**, in this case just the number 1 (**LMS**). \
Next is the **starting address** for the data, two bytes in LSBF format (**LMS**). \
Next is the **ending address** for the data plus 1, also two bytes in LSBF format (**LMS**). \
Then the bytes in PETSCII code for the **file name**, no more than 16 bytes (**LMS**). \
Then `16 - lengthOfFileName` bytes of the number 32 for **title padding** (**LMS**). \
Then write the number 32 for 171 times, this is the **header body**, not sure what it's for (**LMS**). \
Then a **checksum byte** for the header, this is all the bytes from the **header file type** (byte1) to the end of the **header body** (last byte) (**LMS**), use this bitwise formula to calculate: \
`0 xor byte1 xor byte2 xor ... xor last byte` \
Then a **data end bit** of **LS** to end the header. \
Then we write a small pulse 79 times as an **interblock gap**.

Now we write the header again, but with some changes. \
Write the 9 **sync bytes**, but this time from 9 to 1 (**LMS**). \
Then write the exact same **header data** from the **header file type** to the **data end bit** (**LMS**). \
Next write 78 small pulses as the **header end trailer**. \
Then write 5376 small pulses as the **interrecord gap**.

Now we can start writing our actual program data. \
Before we do that, you'll need to know how to make program data (useful link: https://www.c64-wiki.com/index.php/BASIC_token) or get it from RAM. (Or use my tapwriter.py program on GitHub) \
Start the program with 9 **sync bytes**, from 137 down to 129 (**LMS**). \
Next write the bytes of your program exactly how it would appear in ram from the **starting address** to the **ending address** (**LMS**). \
Then write your **checksum byte**, calculated just like on the header blocks, only from after the **sync bytes** all the way to before this **checksum byte** (**LMS**). \
Then write your **data end bit** of **LS** to end the header. \
Then write a small pulse 79 times as the **interblock gap**.

Now we write the program data again, just like with the header. \
Write the 9 **sync bytes**, but from 9 to 1 (**LMS**). \
Copy the program block from after the **sync bytes** all the way to the **data end bit**. \
Then write a small pulse 78 times for the **data end trailer**.

That's all there is to it, just remember to update the tap data length in bytes 16 to 19, this is only the length from byte 20 to the end.

This isn't the most efficient way to store data, since it takes about 10 seconds just to get to the header after the leader block, but it works and it's pretty cool to have a program load off a tap file that you made (forged with your own hands in the heart of a dying star).

If you understand how to create a tap file, and can actually get it to work (took me 3 days), then it wouldn't be hard to reverse the protocol to read the data off of the cassette or even directly from the C64, just as I will be doing for my C64 Arduino Internet project.

GitHub program files:
https://github.com/koppanyh/C64Net/blob/master/tapwriter.py

<sub>Posted by [koppanyh](https://github.com/koppanyh) on 2016/03/20 at 12:34 AM PDT</sub>
