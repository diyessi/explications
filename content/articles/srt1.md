---
title: "SHARE Square Root for the IBM 704"
date: 2024-07-06T00:00:00-00:00
math: true
draft: true
---
# Abstract

The `SRT1` square root routine from the IBM 704 SHARE library executes only two floating point instructions to compute a floating point square root. The 704 was introduced by IBM in 1954. It was the first commercial computer with core memory and hardware support for floating point. The first version of FORTRAN was co-developed with the 704. Here we describe the implementation of `SRT1`, illustrating some early programming techniques.

# The source

In 1955 some IBM 704 users formed a group called [SHARE](https://www.share.org/) to share programs and other information related to IBM computers. One of the founders was [Roy Nutt](https://history.computer.org/pioneers/nutt.html), who wrote the [SHARE Assembler Program (SAP)](https://sky-visions.com/ibm/704/uasap.pdf#page=2) that was used to assemble many of these programs and routines. SHARE distributed programs on magnetic tape. [Paul Pierce's Computer Collection](https://piercefuller.com/collect/index.html) includes recovered contents of a number of early computer tapes, including the [IBM SHARE Library](https://www.piercefuller.com/library/share.html). The [Yale SHARE Tape 2 29-508](https://www.piercefuller.com/library/kyu2.html) contains the `SRT1` square root routine.

# Software development

Today software is written using an editor on a computer to create one or more source files plus some additional files that describe how the source files are to be processed and combined. When problems occur, the appropriate files are modified and the process is repeated. In the 1950s this would have been considered an incredibly wasteful use of very limited computing resources.

At least according to books of the era, software development would start with a detailed off-line analysis of the problem to be solved. In many environments, a flow chart would be produced showing all the steps of the program. The flow chart would be carefully checked to ensure all cases were handled. Hand simulations would be performed. Then a coder would convert the flow chart to a program on paper forms. This would again be carefully checked. In the case of the IBM 704, A key punch operator would use a card punch to create one [IBM punched card](https://en.wikipedia.org/wiki/Punched_card) for each line of the program:

## Punched cards

IBM had been making mechanical and electromechanical punched card equipment years before they got involved with computers. The IBM cards in use at the time originally had 80 columns, numbered 1 through 80, and 10 rows, numbered 0 through 9. A hole punched in a row indicated that column contained the digit corresponding to the row. If there were no punches in a column, it was a blank. Cards were processed vertically. 80 wheels, one for each column, would turn as the card passed over sensors. When a hole was sensed in a column, the wheel for that column would stop, transferring the contents of the card to the wheels.

![Digits](images/digits.png)

There was a demand for being able to store alphanumeric data on cards. There was room for two additional rows on top of the card. The top row was called 12, the second row 11. Rows 12, 11, and 0 (or 10) were called *zone* rows. The uppercase alphabet was encoded using two punches, one in a zone row and one in rows 1 through 9:

![Alphabet](images/alpha.png)

It is possible that `S` was `0-2` rather than `0-1` to avoid having a series of `S` characters weaken the card.

Then there was a demand for some symbols. Some zone-only punches were used for symbols. Row 8 was also added as a row that could be punched in combination with a zone punch and a non-8 digit punch. Unlike the digits and the alphabet, there was some variation between sites and over time for how the symbols were encoded. There are ASCII characters, such as `^`, `{` and `}`, which have no punched card encoding. There are also a few punched card symbols which do not even have Unicode encodings. Following are the encodings in use at the end of the punched card era for characters that were also in ASCII:

![Symbols](images/symbols.png)



## Getting the program to the computer

![MISRT1009 Card](images/MISRT1009.png)

A punched card had 80 columns and twelve rows. Each character was represented by a unique combination of up to three holes in the column. Punched card equipment was mostly mechanical, which constrained how characters could be encoded. No punches in a column was a blank and a single punch in one of rows 0 through 9 was the corresponding digit. Alphabetic characters were later added by adding rows 11 and 12 at the top of the card and punching a pair of rows, one from rows 0, 11 or 12 and one from rows 1 through 9. As card applications began to require some symbols, additional encodings were added, although not consistently from site to site and over time. Since each card was 80 characters whether they were used or not, there was no "end of line" character.

The IBM 704 could read cards, or at least the first 72 columns, but it could not do much else while reading cards. IBM had equipment for transferring card directly to seven track magnetic tape which could be read much more quickly. Each character on tape used six bits, and the seventh track was a parity track. The transformation from holes punched in a column (Hollerith encoding) to a six bit value (BCD encoding) was simple.

Information on tapes was grouped into records with erased tape between records. To save space, a record would hold multiple cards.

# The SHARE library tape

The files on Paul Pierce's site are essentially the raw contents of the tapes and require some reverse engineering and pre-processing. Each record contains the contents of one or more cards, which may be in either a binary or a character (symbolic) format. Some tapes have records for 72 character cards and some have records for 80 character cards, possibly depending on their age. Here the BCD format has been converted to ASCII.

## Meta-information

Each routine on the SHARE tape is preceded by a header record that contains meta-information about the routine. There is quite a bit of variability among the headers.

For `SRT1` the header record is:
```
B4 MI SRT1 02-06-58 0399         SY 0045
```
Most of the information can be understood:
 - `B4` is the classification, a root or power, as described in [SHARE Reference Manual, 03.10 -29](https://www.piercefuller.com/scan/share59.pdf#page=66).
 - `MI` indicates the contributor, MIT, as listed in [SHARE Reference Manual, 02.02 -14](https://www.piercefuller.com/scan/share59.pdf#page=27).
 - `SRT1` names this particular implementation of square root.
 - `02-06-58` is the date.
 - `0399` might be related to the record size.
 - `SY` indicates *symbolic*, meaning source.
 - `0045` If cards are numbered from 0000, this is the number of the last card.

## The source code

The complete source code converted to ASCII is:
```
       REM SQUARE ROOT, FLOATING POINT                                 MISRT1000
       REM   TIME (AV.)= 102 CYCLES= 1.224 M.S.                        MISRT1001
       REM   ACCURACY= MAX. ERROR OF HALF LAST BIT                     MISRT1002
       REM   SPACE= 37 + 2 COMMON CELLS                                MISRT1003
       REM   ALARM RETURN= NEGATIVE ARGUMENT                           MISRT1004
       REM   CALLING SEQUENCE= TSX SQRT,4                              MISRT1005
       REM                     (ALARM RETURN)                          MISRT1006
       REM                     (NORMAL RETURN)                         MISRT1007
       REM                                                             MISRT1008
  SQRT TZE 2,4               X ZERO NORMAL RETURN                      MISRT1009
       TMI 1,4               X NEGATIVE ALARM RETURN                   MISRT1010
       ANA SQRT+33           STORE 26 BIT X= Y*Y TO ALLOW ADD          MISRT1011
       STO COMMON              SHIFT AND ROUND FOR AVERAGES            MISRT1012
       ANA SQRT+34           PREPARE Y(1) LESS CONSTANT                MISRT1013
       ARS 1                                                           MISRT1014
       ADD COMMON                                                      MISRT1015
       ARS 1                                                           MISRT1016
       STO COMMON+1                                                    MISRT1017
       ALS 10                                                          MISRT1018
       PBT                                                             MISRT1019
       COM                                                             MISRT1020
       ARS 13                                                          MISRT1021
       ANA SQRT+35                                                     MISRT1022
       ADD COMMON+1                                                    MISRT1023
       ADD SQRT+36           FINISH CORRECTION AND ADJUST CHAR         MISRT1024
       STO COMMON+1          STORE Y(2)                                MISRT1025
       CLA COMMON            ITERATE Y(2)                              MISRT1026
       FDP COMMON+1                                                    MISRT1027
       CLA COMMON+1                                                    MISRT1028
       STQ COMMON+1                                                    MISRT1029
       ADD COMMON+1            AVERAGE                                 MISRT1030
       LRS 1                                                           MISRT1031
       RND                     PREVENT CHAR DISAGREEMENT               MISRT1032
       STO COMMON+1          STORE Y(3)                                MISRT1033
       CLA COMMON            ITERATE Y(3)                              MISRT1034
       FDP COMMON+1                                                    MISRT1035
       CLA COMMON+1                                                    MISRT1036
       STQ COMMON+1                                                    MISRT1037
       ADD COMMON+1            AVERAGE                                 MISRT1038
       LRS 1                                                           MISRT1039
       RND                     IMPROVE ACCURACY                        MISRT1040
       TOV 2,4               NORMAL RETURN WITH Y(4)                   MISRT1041
       OCT 777777777776      MASK FOR 27TH BIT CLIP                    MISRT1042
       OCT 001000000000      MASK FOR CHAR MOD 1                       MISRT1043
       OCT 000017777777      MASK FOR CORRECTION                       MISRT1044
       OCT 100360000001      LUMPED CONSTANT                           MISRT1045
```

# Understanding the source

Each line contains exactly 80 characters and corresponds to one 80 column IBM punched card. Character positions, numbered from 1 to 80, are called columns, not because of the columns formed in the source listing, but because of the column of punched holes that encodes the character on a card. Since the IBM 704 could only read the first 72 columns, the last 8 characters were ignored.

![MISRT1009 Card](images/MISRT1009.png)

It was common to use columns on the right side of the card to add sequencing information that was ignored by the assembler. If the cards were accidentally dropped, a punched card sorter, a common piece of office equipment at the time, could be used to put them back in order, although for 46 cards it was probably faster to sort them by hand. Here they will serve as a way to reference particular lines.

## The opcode

Columns 8 through 10 contain a three character operation which determine the syntax for the line. Detailed descriptions of the operations can be found in [IBM 704 electronic data-processing machine](https://bitsavers.org/pdf/ibm/704/24-6661-2_704_Manual_1955.pdf) while pseudo-ops are described in SAP assembler documentation.

## Comments

Any characters separated by at least one space from the last operand are comments. If the assembler reads the program from cards, characters in columns 73 through 80 will not be seen and will not show up in a listing, while when read from tape they are available to the assembler. The `REM` (remark) pseudo-op is used for lines that only contain comments. Early versions of the assembler required two spaces between `REM` and the comment, but the first `REM` above indicates that this requirement was later dropped.
```
       REM  THIS IS A REMARK COMMENT
       CLA X,2 THIS IS AN INSTRUCTION COMMENT
```

## Memory

In the 704 documentation, a *register* is any place a value can be stored. For the purposes of understanding the `SQRT` implementation, IBM 704 addresses are 15 bits and each address corresponds to a 36 bit word. In a word, the high bit designated `S`, followed by bits `1` through `35`. Values are normally written in octal, but we will usually use binary. For the word `W`, `W[S]` or \(W_S\) will designate the value of the `S` bit, `W[2]` or \(W_2\) is the value of bit 2, and `W[2-5]` or \(W_{[2-5]}\) is the value of bits 2 through 5. Words are written in octal while addresses are usually written in decimal.
```
                 1   1   1   2   2   2   3   3 3
S 1  3   6   9   2   5   8   1   4   7   0   3 5
0 00 000 000 000 000 000 000 000 000 000 000 000
```

## Architecture

The 704 was organized like a basic calculator. Most instructions have as input a 38 bit *accumulator* register called `AC` and the contents of a memory location, with the result becoming the new value for `AC`. The `AC` register has two additional bits positions, `Q` and `P`, between `S` and 1:
```
                    1   1   1   2   2   2   3   3 3
S Q P1  3   6   9   2   5   8   1   4   7   0   3 5
0 0 000 000 000 000 000 000 000 000 000 000 000 000
```

For multiplication, division, and some shifting operations, a second 36 bit *multiplier quotient* register, `MQ`, is also used.

## Instructions

All instructions use one word. Most instructions contain an *address* in bits 21-35 and a *tag* in bits 18-20. Some instructions contain a 15 bit *decrement* in bits 3-17. In the original Lisp implementation, a `CONS` cell was one word, and `CAR` was the contents of the address register and `CDR` was the contents of the decrement register.

In assembly, the first six columns are an optional label for the address that will be associated with the instruction. Any spaces will be removed, so "OLD X" and "OLDX" are the same label. Column 7 must contain a blank. Columns 8 through 10 contain the opcode or pseudo-op and column 11 must be blank. For a regular operation, starting at column 12 are up to three comma-separated expressions containing no blanks followed by a blank. The three expressions are the address, tag, and decrement. If not specified, they default to 0.

## Effective Address

There are three 15 bit *index* registers, called 1, 2, and 4. Most instructions are *indexable* and compute an effective address called `Y`. If a tag is specified, it must be a value from 0 through 7, which specifies a bit mask for selecting from zero to three index registers. The contents of the selected index registers are ORed together and subtracted from the address field in the instruction to obtain `Y`, which is used as a source or destination, depending on the operation. If the tag is 0, nothing is subtracted from the instruction address.

As an example, to add the contents of addresses `X` and `Y` in memory and store the result in `Z`:
```
       CLA X     AC = X from memory
       ADD Y     AC += Y from memory
       STO Z     Z in memory = AC 
```
If instead index register 2 contains the twos complement of the address of a pair of numbers and the sum is to be stored in `Y`,
```
       CLA 0,2   AC = first value in pair
       ADD 1,2   AC += second value in pair
       STO Z     Z in memory = AC 
```

# Linkage

Programs were organized somewhat differently than they are today. There was no call stack with stack frames containing return information and local storage. The main program was a collection of segments consisting of a mix of instructions and data connected together by conditional and unconditional branches. Subroutines were often inline. When not inline, they were branched to in such a way that when the routine was complete it could branch to the next segment to be executed. In some case, such as the square root, there might be more than one possible continuation segment, so a way was needed for the subroutine to determine where to branch to in each case. If there were arguments and/or results that could not be kept in registers, there also needed to be a way to tell the subroutine where to find the arguments and where to store the results.

One way this was done was to have a user of a subroutine first modify instructions in the subroutine to specify the proper return locations and addresses and then branch to the subroutine. Exit points in the subroutine would then be branches to the part of the program that was supposed to be executed next.

The 704 provides a simpler mechanism. There are three address-sized *index* registers, 1, 2, and 4. Most operations have a three bit *tag* field which specifies how an index value is read from/written to the index registers. For reading, the specified register contents are ORed, while for writing the specified index registers are set. When the tag is 0 for a read, the index value is 0, and for a write no index register is written. An *indexable* operation computes its effective address by subtracting the index value specified by its tag from its address operand.

Stack frames let functions obtain storage while they are active. It would waste memory, which was a very limited resource, if every routine had to reserve fixed working storage. Instead, a region of memory designated `COMMON` lets routines share temporary storage. A routine that uses `COMMON` specifies how much storage it needs in `COMMON` and the program allocates enough space in `COMMON` for the maximum required size. When a routine is invoked, the caller must assume that the routine will overwrite the part of `COMMON` that it says it will use.

## Linkage for SQRT

The first few lines describe how the routine is to be used:
```
       REM   SPACE= 37 + 2 COMMON CELLS                                MISRT1003
       REM   ALARM RETURN= NEGATIVE ARGUMENT                           MISRT1004
       REM   CALLING SEQUENCE= TSX SQRT,4                              MISRT1005
       REM                     (ALARM RETURN)                          MISRT1006
       REM                     (NORMAL RETURN)                         MISRT1007
```
The complete `SQRT` routine uses 37 words.

The `TSX` operation, *Transfer and set Index*, has a tag field but is not indexable. It stores the twos complement of the instruction counter in the specified index register(s) and then branches to the instruction in its address. By convention, index register 4 is used for linkage.

`SQRT` has an error return and a normal return. The error return will be used when the argument is negative, and the normal return will be used otherwise.

## Invoking SQRT

A program would invoke `SQRT` by setting `AC` to the argument and calling
```
       TSX SQRT,4  SET INDEX REGISTER 4 TO PC, TRANSFER TO SQRT
       TRA ERROR   TRANSFER TO NEGATIVE ARGUMENT ERROR HANDLING
       ...         USE SQRT RESULT IN AC
```
Indexable operations can refer to addresses relative to the `TSX` operation by using the relative offset as their address and specifying the appropriate tag. For example, if a subroutine should branch to the normal return operation after the `TSX` operation that invoked it, it would branch to 2 with a tag of 4:
```
HELPER REM        A SUBROUTINE
       ...
       TRA 2,4    RETURN TO CALLER
```

# Number representations

## Fixed point/integer

In the fixed point representation, the sign bit specifies the sign, the magnitude bits specify the absolute value, and the programmer tracks a power of two the magnitude is scaled by to get the value. If the scale is 1, the values are integers. This is the case for all fixed point representations used in `SQRT`, so a word \(W\) represents the integer \((1-2W_S)W_{1-35}\). A value is considered to be 0 when \(W_{1-35}=0\), regardless of \(W_S\), but \(W=0\) is the conventional representation for 0. The two zeros can be distinguished by including a sign. `AC` represents the integer \((2-\texttt{AC}_S)\texttt{AC}_{Q-35}\). This is identical to how positive integers are represented today.

When words representing integers are written using octal notation, they may be written with a separate sign or with `S` folded into the first digit. For example, the values \(-2\) through \(2\) written using the signed and unsigned formats are:
```
         Signed      Unsigned
-2 : -000000000002 400000000002 
-1 : -000000000001 400000000001
-0 : -000000000000 400000000000
+0 : +000000000000 000000000000
+1 : +000000000001 000000000001
+2 : +000000000002 000000000002
```

## Floating point

The floating point representation uses 36 bits containing three unsigned integer fields: a one bit *sign* \(s\) in the sign bit `S`, an eight bit *characteristic* \(c\) in bits 1 through 8, and a 27 bit *fraction* \(f\) in bits 9 through 35. We will use the notation \([s|c|f]\). The sign is used in the same was as with the integer representation. The characteristic is the binary exponent biased by 128 and the fraction is the numerator of a fraction with denominator \(2^{27}\).

The floating point value \(v\) associated with a sign bit of \(s\), characteristic bits \(c\), and fraction bits \(f\) is
$$
\begin{aligned}
v&=(1-2s) 2^{c-128}2^{-27}f\\\\
&=\hat{s}2^{\hat{c}}\hat{f}
\end{aligned}
$$
where
$$
\begin{aligned}
\hat{s}&=1-2s\\\\
\hat{c}&=c-128\\\\
\hat{f}&=2^{-27}f.
\end{aligned}
$$
A value can have more than one representation because by increasing the exponent and shifting the fraction to the right. For example, three representations of 1.0 are:
```
VALUE   S    C        F
1.0   | 0 | 201 | 400000000
1.0   | 0 | 202 | 200000000
1.0   | 0 | 203 | 100000000
```
The representation is normalized if \(c=0\) when \(f=0\) or if when \(f\ne 0\), \(f \ge 2^{26}\). For non-zero values, \(f_1=1\) and \(\frac{1}{2}\le\hat{f}<1\).

Some representations of normalized floating point values in octal and binary are:
```
VALUE      |      OCTAL      |                     BINARY
           | S  C  F         | S CHAR     FRACTION
-1.0       | 1 201 400000000 | 1 10000001 100 000 000 000 000 000 000 000 000
-0.33      | 1 177 521727024 | 1 01111111 101 010 001 111 010 111 000 010 100
-0.0       | 1 000 000000000 | 1 00000000 000 000 000 000 000 000 000 000 000
+0.0       | 0 000 000000000 | 0 00000000 000 000 000 000 000 000 000 000 000
+0.25      | 0 177 400000000 | 0 01111111 100 000 000 000 000 000 000 000 000
+0.33      | 0 177 521727024 | 0 01111111 101 010 001 111 010 111 000 010 100
+.499...   | 0 177 777777777 | 0 01111111 111 111 111 111 111 111 111 111 111
+0.5       | 0 200 400000000 | 0 10000000 100 000 000 000 000 000 000 000 000
+0.7       | 0 200 546314632 | 0 10000000 101 100 110 011 001 100 110 011 001
+SQRT(0.5) | 0 200 552023632 | 0 10000000 101 101 010 000 010 011 110 011 010
+.99...    | 0 200 777777777 | 0 10000000 111 111 111 111 111 111 111 111 111
+1.0       | 0 201 400000000 | 0 10000001 100 000 000 000 000 000 000 000 000
+SQRT(2.0) | 0 201 552023632 | 0 10000001 101 101 010 000 010 011 110 011 010
+2.0       | 0 202 400000000 | 0 10000010 100 000 000 000 000 000 000 000 000
```
Note that the floating point and integer representations of zero are the same. Floating point values also have the same order as their normalized representations when reinterpreted as integer representations, so the same comparison operations were used for integer and normalized floating point representations.

When `AC` holds a normalized floating point representation, `Q` and `P` will be 0.

# The SQRT routine

Given a normalized floating point representation \([s,c,f]\) for \(x\) in `AC`, the routine returns with a normalized floating point representation \([s, d, g]\) for \(y=\sqrt{x}\) in `AC` if \(x\not < 0\) and indicates an error if \(x < 0\).
 
## Special case for 0

```
  SQRT TZE 2,4               X ZERO NORMAL RETURN                      MISRT1009
```
The `TZE` operation, *test zero*, transfers control to its effective address if `AC[Q-35] = 0`. This address is two words past the call point and is the address for a normal return. Since `AC` has not been changed, the result is 0.

## Error for negative argument

```
       TMI 1,4               X NEGATIVE ALARM RETURN                   MISRT1010
```
The `TMI` operation, *test minus*, transfers control to its effective address if `AC[S] = 1`. This address is the alarm address, one word after the call point. Since `AC` has not been changed, exception handling can take action based on the value of the argument.

At this point we know \(x>0\) and \(y > 0\).

## Taking advantage of the representation

Since \(x=y^2\), we can compute \(\hat{c}\) and \(\hat{f}\) from \(\hat{d}\) and \(\hat{g}\):
$$
2^{\hat{c}}\hat{f}=x=y^2=2^{2\hat{d}}\hat{g}^2.
$$
We can't just say \(\hat{c}=2\hat{d}\) and \(\hat{f}=\hat{g}^2\) because \(f\) must be in normalized form. We have two cases:
1) When \(\frac{1}{2}\le \hat{g}<\sqrt{\frac{1}{2}}\), \(\frac{1}{4}\le\hat{g}^2<\frac{1}{2}\) is not in the range of a normalized fraction, so we need to multiply by 2 to get the fraction in normalized form: \(\frac{1}{2}\le2\hat{g}^2<1\). The exponent will need to be reduced by 1 to cancel out the 2, so \(\hat{c}=2\hat{d}-1\) and \(\hat{f}=2\hat{g}^2\).
2) When \(\sqrt{\frac{1}{2}}\le \hat{g} < 1\), \(\frac{1}{2}\le\hat{g}^2<1\), so \(\hat{g}^2\) is normalized. Then \(\hat{c}=2\hat{d}\) and \(\hat{f}=\hat{g}^2\).

This gives us \(\hat{c}\) and \(\hat{f}\) in terms of \(\hat{d}\) and \(\hat{g}\), but we really want \(\hat{d}\) and \(\hat{g}\) in terms of \(\hat{c}\) and \(\hat{f}\). We can use the fact that in the first case \(\hat{c}\) is even and odd in the second to determine which case we are in. Since \(c=\hat{c}+128\), \(c_8=0\) if \(\hat{c}\) is even and 1 if odd. We can combine the two cases to:
$$
\begin{align*}
\hat{c}&=2\hat{d}-c_8\\\\
\hat{f}&=2^{c_8}\hat{g}^2.
\end{align*}
$$
Solving for \(\hat{d}\) and \(\hat{f}\) gives:
$$
\begin{align*}
\hat{d}&=\frac{\hat{c}+c_8}{2}\\\\
\hat{g}&=\sqrt{\frac{f}{2^{c_8}}}.
\end{align*}
$$
We really want \(d\) in terms of \(c\), so
$$
d=\frac{c+c_8}{2}+64.
$$

## Setup

It will be helpful to augment the instruction descriptions with symbolic and actual binary results of the bit manipulations corresponding to arguments of .25, .33, .5 and .7. A symbolic representation of the characteristic and fraction will reference the original characteristic bits as `C` and fraction bits as `F`. For exmaple, if `C = 01101100` then `C[2-5] = 1101` and `C[1-5] = 01101`. Literals may be interspersed, as in `0 C[2,3] 1 = 0111`.

In `AC` these we start with:
```
 x  | S | QP | CHAR     | FRACTION
    | 0 | 00 | C[1-8]   | F[1-27]
.25 | 0 | 00 | 01111111 | 100 000 000 000 000 000 000 000 000
.33 | 0 | 00 | 01111111 | 101 010 001 111 010 111 000 010 100
.49 | 0 | 00 | 01111111 | 111 111 111 111 111 111 111 111 111
.50 | 0 | 00 | 10000000 | 100 000 000 000 000 000 000 000 000
.70 | 0 | 00 | 10000000 | 101 100 110 011 001 100 110 011 010
.99 | 0 | 00 | 10000000 | 111 111 111 111 111 111 111 111 111
```

The next instructions is:
```
       ANA SQRT+33           STORE 26 BIT X= Y*Y TO ALLOW ADD          MISRT1011
```
The `ANA` operation, *AND to Accumulator*, ands `AC` to the value at the effective address. This value in `SQRT+22` is defined with the pseudo-op `OCT` which specifies a raw octal value:
```
       OCT 777777777776      MASK FOR 27TH BIT CLIP                    MISRT1042
```
When the comment refers to `27TH BIT` it is referring to the bit in the fraction, not the word. The `ANA` will clear the low-order bit of the fraction. This will lower the square root by less than half the low fraction bit, so it will not make a difference in the floating point representation of the square root. It will make a difference in the computation, as described later.

`AC` now contains:
```
 x  | S | QP | CHAR     | FRACTION
    | 0 | 00 | C[1-8]   | F[1-26]0
.25 | 0 | 00 | 01111111 | 100 000 000 000 000 000 000 000 000
.33 | 0 | 00 | 01111111 | 101 010 001 111 010 111 000 010 100
.49 | 0 | 00 | 01111111 | 111 111 111 111 111 111 111 111 110
.50 | 0 | 00 | 10000000 | 100 000 000 000 000 000 000 000 000
.70 | 0 | 00 | 10000000 | 101 100 110 011 001 100 110 011 010
.99 | 0 | 00 | 10000000 | 111 111 111 111 111 111 111 111 110
```

Next,
```
       STO COMMON              SHIFT AND ROUND FOR AVERAGES            MISRT1012
```
The `STO` operation, *Store*, stores `AC[S-35]` in the effective address, so `COMMON` now contains:
```
COMMON
 x  | S | CHAR     | FRACTION
    | 0 | C[1-8]   | F[1-26]
.25 | 0 | 01111111 | 100 000 000 000 000 000 000 000 000
.33 | 0 | 01111111 | 101 010 001 111 010 111 000 010 100
.49 | 0 | 01111111 | 111 111 111 111 111 111 111 111 110
.50 | 0 | 10000000 | 100 000 000 000 000 000 000 000 000
.70 | 0 | 10000000 | 101 100 110 011 001 100 110 011 010
.99 | 0 | 10000000 | 111 111 111 111 111 111 111 111 110
```

## Exponent

Now the bit tricks begin:
```
       ANA SQRT+34           PREPARE Y(1) LESS CONSTANT                MISRT1013
```
This time `AC` is anded with
```
       OCT 001000000000      MASK FOR CHAR MOD 1                       MISRT1043
```
Although the comment refers to `CHAR MOD 1` this actually leaves \(c \mod 2\) in the characteristic position and zeros out the rest of `AC`.
```
 x  | S | QP | CHAR     | FRACTION
    | 0 | 00 |     C[8] | 000 000 000 000 000 000 000 000 000
.25 | 0 | 00 | 00000001 | 000 000 000 000 000 000 000 000 000
.33 | 0 | 00 | 00000001 | 000 000 000 000 000 000 000 000 000
.49 | 0 | 00 | 00000001 | 000 000 000 000 000 000 000 000 000
.50 | 0 | 00 | 00000000 | 000 000 000 000 000 000 000 000 000
.70 | 0 | 00 | 00000000 | 000 000 000 000 000 000 000 000 000
.99 | 0 | 00 | 00000000 | 000 000 000 000 000 000 000 000 000
```
You might expect that `AC` would next be tested to see if it was 0 and then split the computation into two cases, one for even exponents and one for odd exponents. Instead, both cases will be handled at the same time.
```
       ARS 1                                                           MISRT1014
```
`ARS` is *accumulator right shift*, in this case by 1 bit. `ARS` shifts the bits in `Q-35`, shifting in 0s from the left and dropping bits on the right. The `S` bit is not changed. Bit 9, the first bit of the fraction, now contains the low bit of the characteristic.
```
 x  | S | QP | CHAR     | FRACTION
    | 0 | 00 | 00000000 | C[8]00 000 000 000 000 000 000 000 000
.25 | 0 | 00 | 00000000 |    100 000 000 000 000 000 000 000 000
.33 | 0 | 00 | 00000000 |    100 000 000 000 000 000 000 000 000
.49 | 0 | 00 | 00000000 |    100 000 000 000 000 000 000 000 000
.50 | 0 | 00 | 00000000 |    000 000 000 000 000 000 000 000 000
.70 | 0 | 00 | 00000000 |    000 000 000 000 000 000 000 000 000
.99 | 0 | 00 | 00000000 |    000 000 000 000 000 000 000 000 000
```
Next,
```
       ADD COMMON                                                      MISRT1015
```
The `ADD` operation sets `AC` to the integer addition of `AC` and the contents of the effective address.

Since `COMMON` is normalized, the first bit of the fraction is 1. If `AC[9] = 1` (odd \(c\)) then adding `COMMON` to `AC` will add 1 to the first bit of the fraction, turning it to 0, and carry into the characteristic, adding 1 to it. This may generate a carry into `AC[P]`. For this part, it will be simplest to treat the characteristic as `AC[Q-8]`. This gives an even characteristic of \(c+c_8\) and a fraction of \(\hat{f}-\frac{c_8}{2}\).
```
 x  | S |  CHAR       | FRACTION
    | 0 | C+C[8]      | (1-C[8]) F[2-26]0
.25 | 0 | 00 10000000 | 000 000 000 000 000 000 000 000 000
.33 | 0 | 00 10000000 | 001 010 001 111 010 111 000 010 100
.49 | 0 | 00 10000000 | 011 111 111 111 111 111 111 111 110
.50 | 0 | 00 10000000 | 100 000 000 000 000 000 000 000 000
.70 | 0 | 00 10000000 | 101 100 110 011 001 100 110 011 010
.99 | 0 | 00 10000000 | 111 111 111 111 111 111 111 111 110
```
Next,
```
       ARS 1                                                           MISRT1016
       STO COMMON+1                                                    MISRT1017
```
Shifting `AC` right by 1 halves the characteristic and makes `AC[P]=0` again. The characteristic is now $$\frac{c+c_8}{2}=d-64.$$ Since the characteristic is even, a 0 is shifted into the fraction and the fraction is halved to 
$$\frac{\hat{f}-\frac{c_8}{2}}{2}.$$ This value is stored in `COMMON+1`.
```
COMMON+1
 x  | S | QP | CHAR     | FRACTION
    | 0 | 00 | D-64     | 0(1-C[8])F[2-26]0
.25 | 0 | 00 | 01000000 | 000 000 000 000 000 000 000 000 000
.33 | 0 | 00 | 01000000 | 000 101 000 111 101 011 100 001 010
.49 | 0 | 00 | 01000000 | 001 111 111 111 111 111 111 111 111
.50 | 0 | 00 | 01000000 | 010 000 000 000 000 000 000 000 000
.70 | 0 | 00 | 01000000 | 010 110 011 001 100 110 011 001 101
.99 | 0 | 00 | 01000000 | 011 111 111 111 111 111 111 111 111
```

## Fraction

The computation of \(d\) is almost complete, but what is going on with the fraction? This is the beginning of a linear approximation to \(\hat{g}\), which we'll call \(\hat{g}_0\). 

Recall that there are two cases for the fraction, one for an odd exponent and one for an even exponent. In each case, \(\frac{1}{2}\le \hat{f}<1\), but with odd exponents we need to compute \(\sqrt{\frac{f}{2}}\) and with the even exponent \(\sqrt{\hat{f}}\). We can combine these by computing \(\sqrt{s\hat{f}}\) where \(s\) is \(\frac{1}{2}\) for odd exponents and 1 for even exponents. We want the linear approximation to match \(\sqrt{s\hat{f}}\) when \(\hat{f}\) is \(\frac{1}{2}\) and 1. Then
$$
\begin{align*}
\hat{g}_0&=\sqrt{\frac{s\hat{f}}{2}}+\left(\hat{f}-\frac{1}{2}\right)\left(\sqrt{s\hat{f}}-\sqrt{\frac{sf}{2}}\right)\\\\
&=\sqrt{s}\left(\left(2-\sqrt{2}\right)\hat{f}+\sqrt{2}-1\right)\\\\
&=\begin{cases}
\hat{f}\left(\sqrt{2}-1\right)+1-\frac{\sqrt{2}}{2}&\text{for odd exponents }(s=\frac{1}{2})\\\\
\hat{f}\left(2-\sqrt{2}\right)+\sqrt{2}-1&\text{for even exponents }(s=1).
\end{cases}
\end{align*}
$$
Both cases involve a floating point multiplication (17 cycles) and addition (7 cycles). Since this is already an approximation, we can approximate a little faster by using integer operations, in particular adds and shifts, which are 2 cycles each. We start with \(\sqrt{2}\). Since \(16\sqrt{2}\approx 23\), we can replace \(\sqrt{2}\) with \(\frac{23}{16}\) to get:
$$
\hat{g}_0=
\begin{cases}
\frac{7}{16}\hat{f} + \frac{9}{32}&\text{odd exponent}\\\\
\frac{9}{16}\hat{f}+\frac{7}{16}&\text{even exponent}.
\end{cases}
$$

```
       ALS 10                                                          MISRT1018
```
The `ALS` operation, *accumulator left shift*, shifts `AC` left by 10 bits. As with `ARS`, the `S` bit is not changed and `Q-35` are treated as a group with 0s shifted in from the right. If a 1 bit is shifted into or through `P` the overflow indicator is set.

This will shift the fraction so that its first bit is in `AC[Q]`. It will be simplest to treat `AC[Q-35]` as one field in this step:
```
 x  | S | Q-35
    | 0 | 0(1-C[8])F[2-26]*2^10
.25 | 0 | 0 000 000 000 000 000 000 000 000 000 000 000 000
.33 | 0 | 0 001 010 001 111 010 111 000 010 100 000 000 000
.49 | 0 | 0 011 111 111 111 111 111 111 111 110 000 000 000
.50 | 0 | 0 100 000 000 000 000 000 000 000 000 000 000 000
.70 | 0 | 0 101 100 110 011 001 100 110 011 010 000 000 000
.99 | 0 | 0 111 111 111 111 111 111 111 111 110 000 000 000
```
Using the 704 bit numbering for a word, the characteristic is in bits 1-8, so shifting left 10 bits moves the entire characteristic through `P`. Thus, if the characteristic is not 0, the overflow indicator will be set. If the characteristic is 0, it is even and a 1 will be shifted into `P`, again setting the overflow indicator.  The overflow indicator will remain set until it is explicitly cleared.

Next,
```
       PBT                                                             MISRT1019
       COM                                                             MISRT1020
```
The `PBT` operation, *P Bit test*, skips the next instruction if `AC[P] = 1`, i.e. if \(c_8=0\). The `COM` operation complements `AC[Q-35]`.
```
 x  | S | Q-35
    | 0 | c8 ? (2^38-1)-00F[2-26]*2^10 : 01F[2-26]*2^10
.25 | 0 | 1 111 111 111 111 111 111 111 111 111 111 111 111
.33 | 0 | 1 110 101 110 000 101 000 111 101 011 111 111 111
.49 | 0 | 1 100 000 000 000 000 000 000 000 001 111 111 111
.50 | 0 | 0 100 000 000 000 000 000 000 000 000 000 000 000
.70 | 0 | 0 101 100 110 011 001 100 110 011 010 000 000 000
.99 | 0 | 0 111 111 111 111 111 111 111 111 110 000 000 000
```
Next,
```
       ARS 13                                                          MISRT1021
```
The fraction had been shifted left by 10, and is now shifted right by 13, which shifts the previous fraction right by 3, possibly complemented.
```
 x  | S | QP | CHAR     | FRACTION
    | 0 | 00 | 00000000 | 000 c8 1 (c8 ? (2^23-1)-F[2-23] : F[2-23])
.25 | 0 | 00 | 00000000 | 000 111 111 111 111 111 111 111 111
.33 | 0 | 00 | 00000000 | 000 111 010 111 000 010 100 011 110
.49 | 0 | 00 | 00000000 | 000 110 000 000 000 000 000 000 000
.50 | 0 | 00 | 00000000 | 000 010 000 000 000 000 000 000 000
.70 | 0 | 00 | 00000000 | 000 010 110 011 001 100 110 011 001
.99 | 0 | 00 | 00000000 | 000 011 111 111 111 111 111 111 111
```

Now the bits 4 and 5 of the fraction are cleared:
```
       ANA SQRT+35                                                     MISRT1022
```
where `SQRT+35` is
```
       OCT 000017777777      MASK FOR CORRECTION                       MISRT1044
```
This results in:
```
 x  | S | QP | CHAR     | FRACTION
    | 0 | 00 | 00000000 | 000 00 (c8 ? (2^22-1)-F[2-23] : F[2-23])
.25 | 0 | 00 | 00000000 | 000 001 111 111 111 111 111 111 111
.33 | 0 | 00 | 00000000 | 000 001 010 111 000 010 100 011 110
.49 | 0 | 00 | 00000000 | 000 000 000 000 000 000 000 000 000
.50 | 0 | 00 | 00000000 | 000 000 000 000 000 000 000 000 000
.70 | 0 | 00 | 00000000 | 000 000 110 011 001 100 110 011 001
.99 | 0 | 00 | 00000000 | 000 001 111 111 111 111 111 111 111
```
Since `F[1] = 1`, a fraction of `0F[2-27]` is \(\hat{f}-1/2\). Here the `F[2-27]` has been shifted right by 4, which would give
$$\frac{\hat{f}}{16}-\frac{1}{32},$$ which is the fraction for the even case. 

For the odd case, we must subtract this from \((2^{22}-1)2^{-27}=\frac{1}{32}+2^{-27}\) to get $$-\frac{\hat{f}}{16}+\frac{1}{16}-2^{-27}.$$

Next,
```
       ADD COMMON+1                                                    MISRT1023
```
Recall that the fraction in `COMMON+1` is \((\hat{f}-c_8/2)/2\). In the even case, we have
$$
\begin{align*}
\frac{\hat{f}}{2}-\frac{c_8}{4}+\frac{f}{16}-\frac{1}{32}&=\frac{9}{16}f-\frac{1}{32}&c_8=0,\\\\
\frac{\hat{f}}{2}-\frac{c_8}{4}-\frac{\hat{f}}{16}+\frac{1}{16}-2^{-27}&=\frac{7}{16}f-\frac{3}{16}-2^{-27}&c_8=1.
\end{align*}
$$
```
 x  | S | QP | CHAR     | FRACTION
.25 | 0 | 00 | 01000000 | 000 001 111 111 111 111 111 111 111
.33 | 0 | 00 | 01000000 | 000 110 011 110 101 110 000 101 000
.49 | 0 | 01 | 00000000 | 001 111 111 111 111 111 111 111 111
.50 | 0 | 00 | 01000000 | 010 000 000 000 000 000 000 000 000
.70 | 0 | 00 | 01000000 | 010 111 001 100 110 011 001 100 110
.99 | 0 | 00 | 01000000 | 100 001 111 111 111 111 111 111 110
```

Next,
```
       ADD SQRT+36           FINISH CORRECTION AND ADJUST CHAR         MISRT1024
```
where `SQRT+36` is
```
      OCT 100360000001      LUMPED CONSTANT                           MISRT1045
```
This constant is
```
S | CHAR     | FRACTION
0 | 01000000 | 011 110 000 000 000 000 000 000 001
```
The characteristic is 64, the amount we still need to add to the characteristic to get \(d\). The fraction is \(15/32+2^{-27}\). Adding to the previous fraction gives:
$$
\begin{align*}
\frac{9}{16}f-\frac{1}{32}+\frac{15}{32}+2^{-27}&=\frac{9}{16}f+\frac{7}{16}+2^{-27}&c_8=0,\\\\
\frac{7}{16}f-\frac{3}{16}-2^{-27}+\frac{15}{32}+2^{-27}&=\frac{7}{16}f+\frac{9}{32}&c_8=1.
\end{align*}
$$
Aside from the \(2^-{27}\) in the even case, these correspond to the linear approximation determined earlier.  Why the \(2^{-27}\)? We want to subtract a multiple of the fraction in the even case by adding, which requires a twos complement, but the 704 only has ones complement, which will be off by 1, or \(2^{-27}\). Including the \(2^{-27}\) in the lumped constant will correct the odd case, while leaving it out would leave the even case correct. When \(x=1\) the characteristic is odd so including the \(2^{-27}\) in the odd case makes the linear approximation exact.
```
 x  | S | QP | CHAR     | FRACTION
.25 | 0 | 00 | 10000000 | 100 000 000 000 000 000 000 000 000
.33 | 0 | 00 | 10000000 | 100 100 011 110 101 110 000 101 001
.49 | 0 | 00 | 10000000 | 101 110 000 000 000 000 000 000 000
.50 | 0 | 00 | 10000000 | 101 110 000 000 000 000 000 000 001
.70 | 0 | 00 | 10000000 | 110 101 001 100 110 011 001 100 111
.99 | 0 | 00 | 10000000 | 111 111 111 111 111 111 111 111 111
```

Finally,
```
       STO COMMON+1          STORE Y(2)                                MISRT1025
```
replaces `COMMON+1` with the linear approximation.
```
COMMON+1
 x  | S | CHAR     | FRACTION
.25 | 0 | 10000000 | 100 000 000 000 000 000 000 000 000
.33 | 0 | 10000000 | 100 100 011 110 101 110 000 101 001
.49 | 0 | 10000000 | 101 110 000 000 000 000 000 000 000
.50 | 0 | 10000000 | 101 110 000 000 000 000 000 000 001
.70 | 0 | 10000000 | 110 101 001 100 110 011 001 100 111
.99 | 0 | 10000000 | 111 111 111 111 111 111 111 111 111
```

## Error of linear approximation

The linear approximation will be less than \(\sqrt{s\hat{f}}\) between \(\frac{1}{2}\) and 1, so the error \(\epsilon_0\) will be:
$$
\begin{align*}
\epsilon_0&=\sqrt{s\hat{f}}-\sqrt{s}\left(\left(2-\sqrt{2}\right)\hat{f}+\sqrt{2}-1\right)\\\\
&=\sqrt{s}\left(\sqrt{\hat{f}}-\left(2-\sqrt{2}\right)\hat{f}-\sqrt{2}+1\right).
\end{align*}
$$
The error is maximized when
$$
\hat{f}=\frac{3+2\sqrt{2}}{8}\approx .729,
$$
giving a maximum error of about .0126 when \(s=1\).

## Heron's Square Root Formula

The remainder of the square root routine is based on Heron's square root formula. Given an approximation \(y_n\approx \sqrt{x}\), Heron's formula gives a better approximation
$$
y_{n+1} = \frac{1}{2}\left(y_n+\frac{x}{y_n}\right).
$$

To see how quickly this converges, let the error \(\epsilon_n=y_n-\sqrt{x}\). If \(\epsilon_n=0\) then 
\(y_n=\sqrt{x}\) so \(\frac{x}{y_n}=y_n\), \(y_{n+1}=y_n\) and \(\epsilon_{n+1}=0.\)

Otherwise, one application of Heron's formula gives:
$$
\begin{align*}
\epsilon_{n+1}&=y_{n+1}-\sqrt{x}\\\\
&=\frac{1}{2}\left(y_n+\frac{x}{y_n}\right)-\sqrt{x}\\\\
&=\frac{y_n^2+(y_n-e_n)^2}{2y_n}-(y_n-\epsilon_n)\\\\
&=\frac{2y_n^2-2y_n\epsilon_n+\epsilon_n^2-2y_n^2+2y_n\epsilon_n}{2y_n}\\\\
&=\frac{\epsilon_n^2}{2y_n}
\end{align*}
$$
From this we see that whether \(\epsilon_0\) is negative or positive, \(\epsilon_n\) is positive for \(n>0\). The linear approximation is exact for odd exponents when \(\hat{f}=1\) and otherwise less than \(\sqrt{x}\).

We can estimate the error after \(n\) iterations:
$$
\begin{align*}
\epsilon_n&=\frac{1}{2}\frac{\epsilon_n^2}{\epsilon_n+\sqrt{x}}\\\\
&\approx \frac{\epsilon_n^2}{2\sqrt{x}}&\text{for }\epsilon_n\text{ small compared to }\sqrt{x}\\\\
\end{align*}
$$
Then,
$$
\begin{align*}
\epsilon_n&\approx \epsilon_0 \left(\frac{\epsilon_0}{2\sqrt{x}}\right)^{2^n}\\
&\approx 3.73\times 10^{-11}\\
&\approx 2^{-34.6}.
\end{align*}
$$
Thus, two applications of Heron's formula will be have more than the 27 bit precision of the floating point format.

## The first application

At this point, `COMMON` contains \(x\) with exponent \(c\) and fraction \(f\) and `COMMON+1` contains \(y_0\), the linear approximation, with exponent \(d\) and fraction \(g\).
```
       CLA COMMON            ITERATE Y(2)                              MISRT1026
```
The `CLA` operation, *Clear and Add*, clears `AC` and adds the contents of the effective address, so `AC` now contains \(x\).
```
       FDP COMMON+1                                                    MISRT1027
```
The `FDP` operation, *Float Divide or Proceed*, divides `AC` by the contents of the effective address, leaving the normalized quotient in `MQ` and the remainder in `AC`.
```
         | S | CHAR     | FRACTION
x=.25
COMMON   | 0 | 01111111 | 100 000 000 000 000 000 000 000 000
COMMON+1 | 0 | 10000000 | 100 000 000 000 000 000 000 000 000
FDP      | 0 | 10000000 | 100 000 000 000 000 000 000 000 000

x=.33
COMMON   | 0 | 01111111 | 101 010 001 111 010 111 000 010 100
COMMON+1 | 0 | 10000000 | 100 100 011 110 101 110 000 101 001
FDP      | 0 | 10000000 | 100 101 000 011 010 111 100 100 111

x=.49
COMMON   | 0 | 01111111 | 111 111 111 111 111 111 111 111 110
COMMON+1 | 0 | 10000000 | 101 110 000 000 000 000 000 000 000
FDP      | 0 | 10000000 | 101 100 100 001 011 001 000 010 100

x=.50
COMMON   | 0 | 10000000 | 100 000 000 000 000 000 000 000 000
COMMON+1 | 0 | 10000000 | 101 110 000 000 000 000 000 000 001
FDP      | 0 | 10000000 | 101 100 100 001 011 001 000 010 101

x=.70
COMMON   | 0 | 10000000 | 101 100 110 011 001 100 110 011 010
COMMON+1 | 0 | 10000000 | 110 101 001 100 110 011 001 100 111
FDP      | 0 | 10000000 | 110 101 111 001 010 000 110 101 111

x=.99
COMMON   | 0 | 10000000 | 111 111 111 111 111 111 111 111 110
COMMON+1 | 0 | 10000000 | 111 111 111 111 111 111 111 111 111
FDP      | 0 | 10000000 | 111 111 111 111 111 111 111 111 110
```
In the next step the first approximation, `COMMON+1`, and the division result, `FDP`, must have the same exponent. Notice how this is true for all the examples. To see that it is true in general, we need to look at the even and odd exponent cases. In both cases, \(\frac{1}{2}\le \hat{g}_0< 1\).

In the even case to prevent a change in exponent, we need \(\frac{1}{2}\le\frac{\hat{f}}{\hat{g}_0}<1\). For the lower bound,
$$
\begin{align*}
\frac{1}{2}&\le\frac{\hat{f}}{\hat{g}_0}\\
\frac{1}{2}&\le \frac{\hat{f}}{\frac{9}{16}\hat{f}+\frac{7}{16}+2^{-27}}\\
\frac{9}{32}\hat{f}+\frac{7}{32}+2^{-28}&\le\hat{f}\\
\frac{7}{32}+2^{-28}&\le\frac{23}{32}\hat{f}\\
\frac{7+2^{-23}}{23}<\frac{1}{2}&\le\hat{f}.
\end{align*}
$$
For the upper bound,
$$
\begin{align*}
\frac{\hat{f}}{\hat{g}_0&<1\\
\frac{\hat{f}}{\frac{9}{16}\hat{f}+\frac{7}{16}+2^{-27}}&<1\\
\hat{f}&<\frac{9}{16}\hat{f}+\frac{7}{16}+2^{-27}\\
\frac{7}{16}\hat{f}<\frac{7}{16}+2^{-27}\\
\hat{f}<1 < 1+\frac{2^{-24}}{7}.
\end{align*}
$$
In the odd case, to ensure the exponent is increased by 1, we need \(1\le\frac{\hat{f}}{\hat{g}_0<2\). For the lower bound,
$$
\begin{align*}
1&\le\frac{\hat{f}}{\frac{7}{16}\hat{f}+\frac{9}{32}}\\
\frac{7}{16}\hat{f}+\frac{9}{32}&\le\hat{f}\\
\frac{9}{32}\le\frac{9}{16}\hat{f}\\
\frac{1}{2}\le\hat{f}.
\end{align*}
$$
For the upper bound,
$$
\begin{align*}
\frac{\hat{f}}{\frac{7}{16}\hat{f}+\frac{9}{32}}&<2\\
\hat{f}<\frac{7}{8}\hat{f}+\frac{9}{16}\\
\frac{\hat{f}}{8}<\frac{9}{16}\\
\hat{f}<1<\frac{9}{2}.
\end{align*}
$$
There is one other detail to be dealt with. The above is for exact arithmetic. For the `.99` case, `COMMON+1` is 27 1s. If we had not cleared the low bit of \(f\), it would also have been 27 1s and division would have result in a fraction of 1. This would have been shifted right one and the exponent increased. The `ANA` in instruction `MISRT1011` ensures that the value in `COMMON` is less than `COMMON+1` so that the division is not 1 and the exponent is not increased.

```
       CLA COMMON+1                                                    MISRT1028
       STQ COMMON+1                                                    MISRT1029
       ADD COMMON+1            AVERAGE                                 MISRT1030
```
We want to average \(\frac{x}{y_0}\) and \(y_0\). Unfortunately, 704 data paths were limited and there isn't a way to add `MQ` and `AC` without going through memory. The `STQ` operation, *Store MQ*, copies `MQ` into the effective address. We then add the two values.

But `ADD` is an integer add, not a floating point add. How can this work? The exponents of \(y_n\) and \(\frac{x}{y_n}\) will both be the same, \(d\), so the points are aligned and there is no need to shift fractions and adjust exponents during the addition. The value \(2d\) will be in `AC[P-8]`. This is even, so `AC[8]` will be 0 unless there is a carry from adding the fractions. Since fraction bit 1 of both `COMMON+1` and `FDP` are 1, there will be a carry and `AC[8]` will be 1. The result is the normalized floating point sum, shifted left by 1. 
```
       LRS 1                                                           MISRT1031
```
The `LRS` operation, `Long Right Shift`, treats `AC[Q-35]MQ[1-35]` as one wide register and shifts the contents right 1, putting the low bit of the fraction in `MQ[1]`.
```
       RND                     PREVENT CHAR DISAGREEMENT               MISRT1032
```
The `RND` operation, *Round*, adds 1 to `AC` if `MQ[1]` is 1. Since this is immediately followed by a second application of Heron's formula, which will more than correct for an error in the low order bit of the fraction, this would seem to be an unnecessary operation, and it would be if floating point arithmetic were being used.

Notice how for the ".99" case, the combination of the `ANA` in `MISRT1011` and the `RND` in `MISRT1032` work together to ensure that the quotient will not increase the exponent and the final fraction at the end of this application of Heron's formula will be greater than `COMMON` for the next application of Heron's formula.
```
         | S | QP | CHAR     | FRACTION                            | MQ 1
x=.25
COMMON+1 | 0 |      10000000 | 100 000 000 000 000 000 000 000 000
FDP      | 0 | 00 | 10000000 | 100 000 000 000 000 000 000 000 000
ADD      | 0 | 01 | 00000001 | 000 000 000 000 000 000 000 000 000
LRS      | 0 | 00 | 10000000 | 100 000 000 000 000 000 000 000 000 | 0
RND      | 0 | 00 | 10000000 | 100 000 000 000 000 000 000 000 000 

x=.33
COMMON+1 | 0 |      10000000 | 100 100 011 110 101 110 000 101 001
FDP      | 0 | 00 | 10000000 | 100 101 000 011 010 111 100 100 111
ADD      | 0 | 01 | 00000001 | 001 001 100 010 000 101 101 010 000
LRS      | 0 | 00 | 10000000 | 100 100 110 001 000 010 110 101 000 | 0
RND      | 0 | 00 | 10000000 | 100 100 110 001 000 010 110 101 000

x=.49
COMMON+1 | 0 |      10000000 | 101 110 000 000 000 000 000 000 000
FDP      | 0 | 00 | 10000000 | 101 100 100 001 011 001 000 010 100
ADD      | 0 | 01 | 00000001 | 011 010 100 001 011 001 000 010 100
LRS      | 0 | 00 | 10000000 | 101 101 010 000 101 100 100 001 010 | 0
RND      | 0 | 00 | 10000000 | 101 101 010 000 101 100 100 001 010

x=.50
COMMON+1 | 0 |      10000000 | 101 110 000 000 000 000 000 000 001
FDP      | 0 | 00 | 10000000 | 101 100 100 001 011 001 000 010 101
ADD      | 0 | 01 | 00000001 | 011 010 100 001 011 001 000 010 110
LRS      | 0 | 00 | 10000000 | 101 101 010 000 101 100 100 001 011 | 0
RND      | 0 | 00 | 10000000 | 101 101 010 000 101 100 100 001 011

x=.70
COMMON+1 | 0 |      10000000 | 110 101 001 100 110 011 001 100 111
FDP      | 0 | 00 | 10000000 | 110 101 111 001 010 000 110 101 111
ADD      | 0 | 01 | 00000001 | 101 011 000 110 000 100 000 010 110
LRS      | 0 | 00 | 10000000 | 110 101 100 011 000 010 000 001 011 | 0
RND      | 0 | 00 | 10000000 | 110 101 100 011 000 010 000 001 011

x=.99
COMMON+1 | 0 |      10000000 | 111 111 111 111 111 111 111 111 111
FDP      | 0 | 00 | 10000000 | 111 111 111 111 111 111 111 111 110
ADD      | 0 | 01 | 00000001 | 111 111 111 111 111 111 111 111 101
LRS      | 0 | 00 | 10000000 | 111 111 111 111 111 111 111 111 110 | 1
RND      | 0 | 00 | 10000000 | 111 111 111 111 111 111 111 111 111
```


```
       STO COMMON+1          STORE Y(3)                                MISRT1033
```
`COMMON+1` is updated with the new approximation.

## Second application

The second application of Heron's formula is identical, except that there is no need to store the result in `COMMON+1`. Since an application of Heron's formula always results in a non-negative error, the first application ensures that the divisions will no increase the exponent, so the integer arithmetic will be safe.
```
       CLA COMMON            ITERATE Y(3)                              MISRT1034
       FDP COMMON+1                                                    MISRT1035
       CLA COMMON+1                                                    MISRT1036
       STQ COMMON+1                                                    MISRT1037
       ADD COMMON+1            AVERAGE                                 MISRT1038
       LRS 1                                                           MISRT1039
       RND                     IMPROVE ACCURACY                        MISRT1040
```
```
         | S | QP | CHAR     | FRACTION
x=.25
COMMON   | 0 |      01111111 | 100 000 000 000 000 000 000 000 000
COMMON+1 | 0 |      10000000 | 100 000 000 000 000 000 000 000 000 
FDP      | 0 |      10000000 | 100 000 000 000 000 000 000 000 000 
ADD      | 0 | 01 | 00000001 | 000 000 000 000 000 000 000 000 000
LRS      | 0 | 00 | 10000000 | 100 000 000 000 000 000 000 000 000 0
RND      | 0 | 00 | 10000000 | 100 000 000 000 000 000 000 000 000
               .5              400000000

x=.33
COMMON   | 0 |      01111111 | 101 010 001 111 010 111 000 010 100
COMMON+1 | 0 |      10000000 | 100 100 110 001 000 010 110 101 000
FDP      | 0 |      10000000 | 100 100 110 000 111 001 101 100 101
ADD      | 0 | 01 | 00000001 | 001 001 100 001 111 100 100 001 101
LRS      | 0 | 00 | 10000000 | 100 100 110 000 111 110 010 000 110 1
RND      | 0 | 00 | 10000000 | 100 100 110 000 111 110 010 000 111
               0.5744562670588493  446076207

x=.49
COMMON   | 0 |      01111111 | 111 111 111 111 111 111 111 111 110
COMMON+1 | 0 |      10000000 | 101 101 010 000 101 100 100 001 010
FDP      | 0 |      10000000 | 101 101 001 111 111 011 000 101 010
ADD      | 0 | 01 | 00000001 | 011 010 100 000 100 111 100 110 100
LRS      | 0 | 00 | 10000000 | 101 101 010 000 010 011 110 011 010 0
RND      | 0 | 00 | 10000000 | 101 101 010 000 010 011 110 011 010
               0.7071067839860916  552023631*

x=.50
COMMON   | 0 |      10000000 | 100 000 000 000 000 000 000 000 000
COMMON+1 | 0 |      10000000 | 101 101 010 000 101 100 100 001 011
FDP      | 0 |      10000000 | 101 101 001 111 111 011 000 101 010
ADD      | 0 | 01 | 00000001 | 011 010 100 000 100 111 100 110 101
LRS      | 0 | 00 | 10000000 | 101 101 010 000 010 011 110 011 010 1
RND      | 0 | 00 | 10000000 | 101 101 010 000 010 011 110 011 011
               0.7071067914366722  552023632*

x=.70
COMMON   | 0 |      10000000 | 101 100 110 011 001 100 110 011 010
COMMON+1 | 0 |      10000000 | 110 101 100 011 000 010 000 001 011
FDP      | 0 |      10000000 | 110 101 100 010 111 000 110 010 101
ADD      | 0 | 01 | 00000001 | 101 011 000 101 111 010 110 100 000
LRS      | 0 | 00 | 10000000 | 110 101 100 010 111 101 011 010 000 0
RND      | 0 | 00 | 10000000 | 110 101 100 010 111 101 011 010 000
               0.8366600275039673  654275320

x=.99
COMMON   | 0 |      10000000 | 111 111 111 111 111 111 111 111 110
COMMON+1 | 0 |      10000000 | 111 111 111 111 111 111 111 111 111
FDP      | 0 |      10000000 | 111 111 111 111 111 111 111 111 111
ADD      | 0 | 01 | 00000001 | 111 111 111 111 111 111 111 111 110
LRS      | 0 | 00 | 10000000 | 111 111 111 111 111 111 111 111 111 0
RND      | 0 | 00 | 10000000 | 111 111 111 111 111 111 111 111 111
               0.9999999925494194 777777777
```

## Return

```
       TOV 2,4               NORMAL RETURN WITH Y(4)                   MISRT1041
```
Computing a square root should not result in an overflow, but the overflow indicator was set in the `ALS 10` instruction.

The `TOV` operation, *Transfer on Overflow*, clears the overflow indicator and transfers to the effective address if the overflow indicator is on. Otherwise, it continues with the next instruction. Today it would be considered good practice to follow the `TOV` instruction with a branch to some kind of report of an assertion violated, but in 1958 using the `TOV` to both clear the overflow indicator and return was considered efficient.
