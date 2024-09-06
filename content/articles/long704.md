---
title: "Detailed IBM 704 Description"
date: 2024-08-25T17:19:15-07:00
draft: true
---

# The IBM 704

## The word

The 704 had a 36 bit word size. Bit positions in a word, from high to low, were called `S` (sign) followed by `1` through `35`:

```
                 1   1   1   2   2   2   3   3 3
S 1  3   6   9   2   5   8   1   4   7   0   3 5
0 10 110 100 011 000 111 001 101 010 000 100 000
```

Bits 1 through 35 were usually written in octal. The `S` bit could be written as `+/-` for `0/1`, or as `0/1`, or combined with bits 1-2. Spaces could be added, usually in semantically important locations, to improve readability.

Some examples:
```
+2 64307 1 52040 (instruction format)
+264 307152040 (float format)
 264307152040 (36 bits)
```

For the word `W`, `W[S]` or \(W_S\) will designate the value of the `S` bit, `W[2]` or \(W_2\) is the value of bit 2, and `W[2-5]` or \(W_{[2-5]}\) is the value of bits 2 through 5.

We will usually use binary rather than octal because it is easier to see the results of shifting.

## The memory

There were three core memory options: One or two 4096 word core memory units, or one 32,768 word core memory unit. An address was 12, 13 or 15 bits, depending on the memory size, and referenced an entire word. Characters used a six bit encoding called *BCD*. Six characters would be packed in a word, but there was no way to directly address individual characters in a word. Most operations read or wrote an entire word of memory. A few could read particular subfields of a word, or modify the subfield leaving the remainder of the word intact. This was originally intended to make it easier for programs to modify themselves, but also found uses in relocating loaders, squeezing out a few more bits for small temporaries, and implementing Lisp.

Like dynamic RAM, reading a word from core memory was destructive, so the hardware had to write the value back after a read. Unlike dynamic RAM, core memory did not need to be periodically refreshed, and it retained its contents when powered off.

## Registers

Anything that could store a value was called a register, so each word was called a register. In addition, there were some word-sized and address-sized registers.

`SR`, the *storage register*, was a word-sized register used internally by the CPU.

`AC`, the *accumulator*, was an implicit source and/or destination for most instructions. The `AC` register has two additional bits positions, `Q` and `P`, between `S` and 1. When transferring from `AC` and memory, only `AC[S,1-35]` were transferred. When copying from memory to `AC`, data was copied to `AC[S,1-35]` and `AC[Q,P]` were set to 0.
```
AC
                    1   1   1   2   2   2   3   3 3
S Q P1  3   6   9   2   5   8   1   4   7   0   3 5
0 0 000 000 000 000 000 000 000 000 000 000 000 000
```

`MQ`, the *multiplier quotient*, was a 36 bit register used in some instructions. Some instructions combined `AC` and `MQ[1-35]`.
```
                 1   1   1   2   2   2   3   3 3
S 1  3   6   9   2   5   8   1   4   7   0   3 5
0 00 000 000 000 000 000 000 000 000 000 000 000
```

`IC`, the *instruction counter*, was an address-sized register indicating the next instruction to be executed.

`1`, `2` and `4`, were address-sized *index registers*.

## Instructions and the Effective Address

All instructions use one word. Most instructions are *indexable* and compute an effective address called `Y`. Instructions have a three bit field (18-20) called the *tag* and a 15 bit field (21-35) called the *address*. The tag bits are a bit mask that select index registers. The contents of the selected index registers are ORed together and subtracted from the address to obtain the effective address for the instruction. If the tag is 0, the effective address is the address. Some instructions contain a second address-sized field (3-17) called the *decrement*, which is subtracted from the selected index registers.

## Additional 704 information

The book [IBM 704 electronic data-processing machine](https://bitsavers.org/pdf/ibm/704/24-6661-2_704_Manual_1955.pdf) provides a detailed description of the computer. Additional material can be found at 
 - [IBM 704](https://bitsavers.org/pdf/ibm/704/)
 - [IBM 709](https://bitsavers.org/pdf/ibm/709/) faster, added additional capabilities
 - [IBM 7090](https://bitsavers.org/pdf/ibm/7090/) faster and used transistors
 - [IBM 7094](https://bitsavers.org/pdf/ibm/7094/) even faster

## How programs were written

Today software is written using an editor on a computer to create one or more source files plus some additional files that describe how the source files are to be processed and combined. When problems occur, the appropriate files are modified and the process is repeated. In the 1950s this would have been considered an incredibly wasteful use of very limited computing resources.

At least according to books of the era, software development would start with a detailed off-line analysis of the problem to be solved. In many environments, a flow chart would be produced showing all the steps of the program. The flow chart would be carefully checked to ensure all cases were handled. Hand simulations would be performed. Then a coder would convert the flow chart to a program on paper. The program would again be carefully checked. In the case of the IBM 704, A key punch operator would use a card punch to create one [IBM punched card](https://en.wikipedia.org/wiki/Punched_card) for each line of the program:

## Punched cards

![Character set](images/charset.png)

IBM had been making mechanical and electromechanical punched card equipment years before they got involved with computers. The IBM cards in use at the time originally had 80 columns, numbered 1 through 80, and 10 rows, numbered 0 through 9. A hole punched in a row indicated that column contained the digit corresponding to the row. If there were no punches in a column, it was a blank. Cards were processed vertically. 80 wheels, one for each column, would turn as the card passed over sensors. When a hole was sensed in a column, the wheel for that column would stop, transferring the contents of the card to the wheels.

There was a demand for being able to store alphanumeric data on cards. There was room for two additional rows on top of the card. The top row was called 12, the second row 11. Rows 12, 11, and 0 (or 10) were called *zone* rows. The uppercase alphabet was encoded using two punches, one in a zone row and one in rows 1 through 9. It is possible that `S` was `0-2` rather than `0-1` to avoid having a series of `S` characters weaken the card.

Later there was a demand for some symbols. Some zone-only punches were used for symbols. Row 8 was also added as a row that could be punched in combination with a zone punch and a non-8 digit punch. Unlike the digits and the alphabet, there was some variation between sites and over time for how the symbols were encoded. There are ASCII characters, such as `^`, `{` and `}`, which have no punched card encoding. There are also a few punched card symbols which do not even have Unicode encodings. Following are the encodings in use at the end of the punched card era for characters that were also in ASCII.

The punched cards for a program formed a *deck*. If during handling the deck were dropped, the lines of the program would no longer be in the correct order. The assembler format left most of the right side of the card available for comments. In `MISRT1` we see an identifier of the routine, `MISRT1` and the card number within the source. Card sorting equipment or a person could use this to put the cards back in order. They will also serve as useful identifiers for the lines of the program.

## Tape


## Getting the program to the computer

![MISRT1009 Card](images/MISRT1009.png)

A punched card had 80 columns and twelve rows. Each character was represented by a unique combination of up to three holes in the column. Punched card equipment was mostly mechanical, which constrained how characters could be encoded. No punches in a column was a blank and a single punch in one of rows 0 through 9 was the corresponding digit. Alphabetic characters were later added by adding rows 11 and 12 at the top of the card and punching a pair of rows, one from rows 0, 11 or 12 and one from rows 1 through 9. As card applications began to require some symbols, additional encodings were added, although not consistently from site to site and over time. Since each card was 80 characters whether they were used or not, there was no "end of line" character.

The IBM 704 could read cards, or at least the first 72 columns, but it could not do much else while reading cards. IBM had equipment for transferring card directly to seven track magnetic tape which could be read much more quickly. Each character on tape used six bits, and the seventh track was a parity track. The transformation from holes punched in a column (Hollerith encoding) to a six bit value (BCD encoding) was simple.

Information on tapes was grouped into records with erased tape between records. To save space, a record would hold multiple cards.


