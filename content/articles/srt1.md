---
title: "SRT1"
date: 2024-03-21T20:41:38-07:00
math: true
draft: true
---
# IBM 704 Square Root

We describe the origins and execution of an IBM 704 floating point routine from the SHARE library that calculates \(y=\sqrt{x}\).

# The source

## Source of the source

In 1955 some IBM 704 users, including [Roy Nutt](https://history.computer.org/pioneers/nutt.html), formed a group called [SHARE](https://www.share.org/) to share programs and other information related to IBM computers. Programs were distributed on magnetic tape. [Paul Pierce's Computer Collection](https://piercefuller.com/collect/index.html) includes the contents of a number of early computer tapes, including the [IBM SHARE Library](https://www.piercefuller.com/library/share.html) on some tapes from Yale. The [Yale SHARE Tape 2 29-508](https://www.piercefuller.com/library/kyu2.html) contains the square root routine.

## Meta-information

Each routine in the library includes meta-information for the routine. In this case,
```
B4 MI SRT1 02-06-58 0399         SY 0045
```
 - `B4` is the classification, a root or power, as described in [SHARE Reference Manual, 03.10 -29](https://www.piercefuller.com/scan/share59.pdf#page=66).
 - `MI` indicates the contributor, MIT, as listed in [SHARE Reference Manual, 02.02 -14](https://www.piercefuller.com/scan/share59.pdf#page=27).
 - `SRT1` names this particular implementation of square root.
 - `02-06-58` is the date.
 - `0399` is not known. Perhaps related to the record size.
 - `SY` indicates *symbolic*, meaning source.
 - `0045` is the last line.

## The source program
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

The source is easily recognized as being written in an assembly language. The assembler is [SHARE Assembler Program (SAP)](https://sky-visions.com/ibm/704/uasap.pdf#page=2), written by [Roy Nutt](https://history.computer.org/pioneers/nutt.html). Each line corresponds to one 80 column IBM punched card. The punched card character set and the closely related six bit BCD encoding is limited to uppercase letters and some symbols. Character positions, numbered from 1 to 80, are called columns, not because of the columns formed in the source listing, but because of the column of punched holes that encodes the character on a card. Only columns 1 through 72 can be read by the IBM 709 but could be read if the cards were transferred to tape.

Columns 8 through 10 contain a three character operation which determine the syntax for the line.

## Comments

Any characters separated by at least one space from the last operand are comments. If the assembler reads the program from cards, characters in columns 73 through 80 will not be seen and will not show up in a listing, while they are available if the assembler is using tape input. The `REM` (remark) pseudo-op has no operands, giving additional room for a comment.
```
       REM THIS IS A REMARK COMMENT
       CLA X,2 THIS IS AN INSTRUCTION COMMENT
```

## Memory addressing

For the purposes of understanding the `SQRT` implementation, IBM 704 addresses are 15 bits and each address corresponds to a 36 bit word. The high order bit is designated `S`, followed by the *magnitude*, bits `1` through `35`, with `35` being the low order bit. For a word `W`, `W[S]` or \(W_S\) is the sign, `W[2]` or \(W_2\) is bit 2, and `W[2-5]` or \(W_{[2-5]}\) are bits 2 through 5. Words are written in octal while addresses are usually in decimal.

Two registers are used for arithmetic operations:
 - `AC` is a 38 bit *accumulator* register with bits `S`, `Q`, `P`, `1-35` from high to low. The `Q` and `P` bits extend the arithmetic range available for some intermediate results. When `AC` is written to memory, the `Q` and `P` bits are not written to memory, and when memory is written to `AC`, `Q` and `P` will be 0.
 - `MQ` is a 36 bit *multiplier quotient* register
Some instructions treat the pair of `AC` and `MQ` similar to a wide register.

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

Programs were organized somewhat differently than they are today. There was no call stack with stack frames containing return information and local storage. The main program was a collection of segments consisting of a mix of instructions and data connected together by conditional and unconditional branches. Subroutines were often inline. When not inline, they were branched to in such a way that when the routine was complete it could branch to the next segment to be executed. In some case, such as the square root, there might be more than one possible continuation segment, so a way was needed for the subroutine to determine where to branch to in each case. If there were arguments and/or results that could not be kept in registers, there also needed to be a way to tell the subroutine where to find the arguments and where to store the results.

One way this was done was to have a user of a subroutine first modify instructions in the subroutine to specify the proper return locations and addresses and then branch to the subroutine. Exit points in the subroutine would then be branches to the part of the program that was supposed to be executed next.

The 704 provides a simpler mechanism. There are three address-sized *index* registers, 1, 2, and 4. Most instructions have a three bit *tag* field which specifies how an index value is read from/written to the index registers. For reading, the specified register contents are ORed, while for writing the specified index registers are set. When the tag is 0 for a read, the index value is 0, and for a write no index register is written. An *indexable* operation computes its effective address by subtracting the index value specified by its tag from its address operand.

Stack frames let functions obtain storage while they are active. It would waste memory, which was a very limited resource, if every routine had to reserve fixed working storage. Instead, a region of memory designated `COMMON` lets routines share temporary storage. A routine that uses `COMMON` specifies how much storage it needs in `COMMON` and the program allocates enough space in `COMMON` for the maximum required size. When a routine is invoked, the caller must assume that the routine will overwrite the part of `COMMON` that it says it will use.

The `TSX` instruction has a tag field but is not indexable. It stores the twos complement of the instruction counter in the index and then branches to the instruction in its address. By convention, index register 4 is used for linkage.

## Invoking SQRT

A program would invoke `SQRT` by setting `AC` to the argument and calling
```
       TSX SQRT,4  SET INDEX REGISTER 4 TO PC, TRANSFER TO SQRT
       TRA ERROR   TRANSFER TO NEGATIVE ARGUMENT ERROR HANDLING
       ...         USE SQRT RESULT IN AC
```
Indexable instructions can refer to addresses relative to the `TSX` instruction by using the relative offset as their address and specifying the appropriate tag. For example, if a subroutine should branch to the normal return instruction after the `TSX` instruction that invoked it, it would branch to 2 with a tag of 4:
```
HELPER REM        A SUBROUTINE
       ...
       TRA 2,4    RETURN TO CALLER
```

## Integer format

A word \(W\) represents the integer \((1-2W_S)W_{1-35}\). A value is considered to be 0 when \(W_{1-35}=0\), regardless of \(W_S\), but \(W=0\) is the conventional representation for 0. The two zeros can be distinguished by including a sign. `AC` represents the integer \((2-\texttt{AC}_S)\texttt{AC}_{Q-35}\).

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

## Floating point format

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
The representation is normalized if when \(f=0\) then \(c=0\), and when \(f\ne 0\) then \(f \ge 2^{26}\), i.e. \(f_1=1\) and \(\frac{1}{2}\le\hat{f}<1\).

Some representations of normalized floating point values in octal and binary are:
```
VALUE      |      OCTAL      |                     BINARY
           | S  C  F         | S C        F     
-1.0       | 1 201 400000000 | 1 10000001 100 000 000 000 000 000 000 000 000
-0.33      | 1 177 521727024 | 1 01111111 101 010 001 111 010 111 000 010 100
-0.0       | 1 000 000000000 | 1 00000000 000 000 000 000 000 000 000 000 000
+0.0       | 0 000 000000000 | 0 00000000 000 000 000 000 000 000 000 000 000
+0.25      | 0 177 400000000 | 0 01111111 100 000 000 000 000 000 000 000 000
+0.33      | 0 177 521727024 | 0 01111111 101 010 001 111 010 111 000 010 100
+0.5       | 0 200 400000000 | 0 10000000 100 000 000 000 000 000 000 000 000
+0.7       | 0 200 546314631 | 0 10000000 101 100 110 011 001 100 110 011 001
+SQRT(0.5) | 0 200 552023631 | 0 10000000 101 101 010 000 010 011 110 011 001
+1.0       | 0 201 400000000 | 0 10000001 100 000 000 000 000 000 000 000 000
+SQRT(2.0) | 0 201 552023631 | 0 10000001 101 101 010 000 010 011 110 011 001
+2.0       | 0 202 400000000 | 0 10000010 100 000 000 000 000 000 000 000 000
```
Note that the floating point and integer representations of zero are the same. Floating point values also have the same order as their normalized representations when reinterpreted as integer representations, so the same comparison instructions were used for integer and normalized floating point representations.

When `AC` holds a normalized floating point representation, `Q` and `P` will be 0.

## Integers versus fixed point

The IBM 709 documentation refers to the integer format as *fixed point*. Multiplication on the 709 is defined as if the point is to the right of bit 35, i.e.
```
+000000000001 * +000000000001 = +000000000000 000000000001
```
Many computers at the time defined multiplication as if the point were to the left of bit 1, i.e. magnitudes are fractions less than 1:
```
+200000000000 * +200000000000 = +200000000000 000000000000
```
The fraction in floating point representation is in this second format where the point is to the left of the fraction.

# The SQRT routine

Given a normalized floating point representation \([s,c,f]\) for \(x\) in `AC`, the routine returns with a normalized floating point representation \([s, d, g]\) for \(y=\sqrt{x}\) in `AC` if \(x\not < 0\) and indicates an error if \(x < 0\).
 
## Special case for 0

```
  SQRT TZE 2,4               X ZERO NORMAL RETURN                      MISRT1009
```
The `TZE` instruction, *test zero*, transfers control to its effective address if `AC[Q-35] = 0`. This address is two words past the call point and is the address for a normal return. Since `AC` has not been changed, the result is 0.

## Error for negative argument

```
       TMI 1,4               X NEGATIVE ALARM RETURN                   MISRT1010
```
The `TMI` instruction, *test minus*, transfers control to its effective address if `AC[S] = 1`. This address is the alarm address, one word after the call point. Since `AC` has not been changed, exception handling can take action based on the value of the argument.

At this point we know \(x>0\) and \(y > 0\).

## Taking advantage of the representation

Since \(x=y^2\), we can compute \(\hat{c}\) and \(\hat{f}\) from \(\hat{d}\) and \(\hat{g}\):
$$
2^{\hat{c}}\hat{f}=x=y^2=2^{2\hat{d}}\hat{g}^2.
$$
We can't just say \(\hat{c}=2\hat{d}\) and \(\hat{f}=\hat{g}^2\) because \(f\) must be in normalized form. We have two cases:
1) Starting with the fraction, since \(\frac{1}{2}\le \hat{g} < 1\), we know that \(\frac{1}{4}\le\hat{g}^2<1\). We need \(\frac{1}{2}\le\hat{c}<1\), which is only true when \(\sqrt{\frac{1}{2}}\le\hat{g}\). Then \(\hat{c}=2\hat{d}\) and \(\hat{f}=\hat{g}^2\).
2) When \(\frac{1}{2}\le \hat{g}<\sqrt{\frac{1}{2}}\), \(\frac{1}{4}\le\hat{g}^2<\frac{1}{2}\) is not in the range of a normalized fraction, so we need to multiply by 2 to get the fraction in normalized form: \(\frac{1}{2}\le2\hat{g}^2<1\). The exponent will need to be reduced by 1 to cancel out the 2, so \(\hat{c}=2\hat{d}-1\) and \(\hat{f}=2\hat{g}^2\).

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
.50 | 0 | 00 | 10000000 | 100 000 000 000 000 000 000 000 000
.70 | 0 | 00 | 10000000 | 101 100 110 011 001 100 110 011 001
```

The next instructions is:
```
       ANA SQRT+33           STORE 26 BIT X= Y*Y TO ALLOW ADD          MISRT1011
```
The `ANA` instruction, *AND to Accumulator*, ands `AC` with the value at the effective address. This value in `SQRT+22` is defined with the pseudo-op `OCT` which specifies a raw octal value:
```
       OCT 777777777776      MASK FOR 27TH BIT CLIP                    MISRT1042
```
When the comment refers to `26TH BIT` it is referring to the bit in the fraction, not the word. The `ANA` will clear the low-order bit of the fraction, which won't have any effect on the result. `AC` now contains:
```
 x  | S | QP | CHAR     | FRACTION
    | 0 | 00 | C[1-8]   | F[1-26]0
.25 | 0 | 00 | 01111111 | 100 000 000 000 000 000 000 000 000
.33 | 0 | 00 | 01111111 | 101 010 001 111 010 111 000 010 100
.50 | 0 | 00 | 10000000 | 100 000 000 000 000 000 000 000 000
.70 | 0 | 00 | 10000000 | 101 100 110 011 001 100 110 011 000
```

Next,
```
       STO COMMON              SHIFT AND ROUND FOR AVERAGES            MISRT1012
```
The `STO` instruction, *Store*, stores `AC[S-35]` in the effective address, so `COMMON` now contains:
```
 x  | S | CHAR     | FRACTION
    | 0 | C[1-8]   | F[1-26]
.25 | 0 | 01111111 | 100 000 000 000 000 000 000 000 000
.33 | 0 | 01111111 | 101 010 001 111 010 111 000 010 100
.50 | 0 | 10000000 | 100 000 000 000 000 000 000 000 000
.70 | 0 | 10000000 | 101 100 110 011 001 100 110 011 000
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
.50 | 0 | 00 | 00000000 | 000 000 000 000 000 000 000 000 000
.70 | 0 | 00 | 00000000 | 000 000 000 000 000 000 000 000 000
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
.50 | 0 | 00 | 00000000 |    000 000 000 000 000 000 000 000 000
.70 | 0 | 00 | 00000000 |    000 000 000 000 000 000 000 000 000
```
Next,
```
       ADD COMMON                                                      MISRT1015
```
The `ADD` instruction sets `AC` to the integer addition of `AC` and the contents of the effective address.

Since `COMMON` is normalized, the first bit of the fraction is 1. If `AC[9] = 1` (odd \(c\)) then adding `COMMON` to `AC` will add 1 to the first bit of the fraction, turning it to 0, and carry into the characteristic, adding 1 to it. This may generate a carry into `AC[P]`. For this part, it will be simplest to treat the characteristic as `AC[Q-8]`. This gives an even characteristic of \(c+c_8\) and a fraction of \(\hat{f}-c_8/2\).
```
 x  | S |  CHAR       | FRACTION
    | 0 | C+C[8]      | (1-C[8]) F[2-26]0
.25 | 0 | 00 10000000 | 000 000 000 000 000 000 000 000 000
.33 | 0 | 00 10000000 | 001 010 001 111 010 111 000 010 100
.50 | 0 | 00 10000000 | 100 000 000 000 000 000 000 000 000
.70 | 0 | 00 10000000 | 101 100 110 011 001 100 110 011 000
```
Next,
```
       ARS 1                                                           MISRT1016
       STO COMMON+1                                                    MISRT1017
```
Shifting `AC` right by 1 halves the characteristic and makes `AC[P]=0` again. The characteristic is now \((c+c_8)/2=d-64\). Since the characteristic is even, a 0 is shifted into the fraction and the fraction is halved to \((\hat{f}-c_8/2)/2\). This value is stored in `COMMON+1`.
```
Stored in COMMON+1
 x  | S | QP | CHAR     | FRACTION
    | 0 | 00 | D-64     | 0(1-C[8])F[2-26]0
.25 | 0 | 00 | 01000000 | 000 000 000 000 000 000 000 000 000
.33 | 0 | 00 | 01000000 | 000 101 000 111 101 011 100 001 010
.50 | 0 | 00 | 01000000 | 010 000 000 000 000 000 000 000 000
.70 | 0 | 00 | 01000000 | 010 110 011 001 100 110 011 001 100
```

## Fraction

The computation of \(d\) is almost complete, but what is going on with the fraction? This is the beginning of a linear approximation to \(\hat{g}\), which we'll call \(\hat{g}_0\). Recall that there are two cases for the fraction, one for an even exponent and one for an odd exponent.

For an odd exponent, \(\frac{1}{4}\le \frac{\hat{f}}{2}<\frac{1}{2}\). The linear approximation in this region that matches on the endpoints would be:
$$
\begin{align*}
\hat{g}_0&=\left(\frac{\hat{f}}{2}-\frac{1}{4}\right)\frac{\sqrt{\frac{1}{2}}-\sqrt{\frac{1}{4}}}{\frac{1}{2}-\frac{1}{4}}+\sqrt{\frac{1}{4}}\\\\
&=\hat{f}\left(\sqrt{2}-1\right)+\left(1-\frac{\sqrt{2}}{2}\right).
\end{align*}
$$

For the even case, the exponent is 0 and \(\frac{1}{2}\le \hat{f} < 1\), so
$$
\begin{align*}
\hat{g}_0&=\left(\hat{f}-\frac{1}{2}\right)\frac{\sqrt{1}-\sqrt{\frac{1}{2}}}{1-\frac{1}{2}}+\sqrt{\frac{1}{2}}\\\\
&=\hat{f}\left(2-\sqrt{2}\right)+\left(\sqrt{2}-1\right).
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
The `ALS` instruction, *accumulator left shift*, shifts `AC` left by 10 bits. As with `ARS`, the `S` bit is not changed and `Q-35` are treated as a group with 0s shifted in from the right. If a 1 bit is shifted into or through `P` the overflow indicator is set. This will shift the fraction so that its first bit is in `AC[Q]`. It will be simplest to treat `AC[Q-35]` as one field in this step:
```
 x  | S | Q-35
    | 0 | 0(1-C[8])F[2-26]*2^10
.25 | 0 | 000 000 000 000 000 000 000 000 000 000 000 000 0
.33 | 0 | 000 101 000 111 101 011 100 001 010 000 000 000 0
.50 | 0 | 010 000 000 000 000 000 000 000 000 000 000 000 0
.70 | 0 | 010 110 011 001 100 110 011 001 100 000 000 000 0
```

Next,
```
       PBT                                                             MISRT1019
       COM                                                             MISRT1020
```
The `PBT` instruction, *P Bit test*, skips the next instruction if `AC[P] = 1`, i.e. if \(c_8=0\). The `COM` instruction complements `AC[Q-35]`.
```
 x  | S | Q-35
    | 0 | c8 ? (2^38-1)-00F[2-26]*2^10 : 01F[2-26]*2^10
.25 | 0 | 111 111 111 111 111 111 111 111 111 111 111 111 1
.33 | 0 | 111 010 111 000 010 100 011 110 101 111 111 111 1
.50 | 0 | 010 000 000 000 000 000 000 000 000 000 000 000 0
.70 | 0 | 010 110 011 001 100 110 011 001 100 000 000 000 0
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
.50 | 0 | 00 | 00000000 | 000 010 000 000 000 000 000 000 000
.70 | 0 | 00 | 00000000 | 000 010 110 011 001 100 110 011 001
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
.50 | 0 | 00 | 00000000 | 000 000 000 000 000 000 000 000 000
.70 | 0 | 00 | 00000000 | 000 000 110 011 001 100 110 011 001
```
Since `F[1] = 1`, a fraction of `0F[2-27]` is \(\hat{f}-1/2\). Here the `F[2-27]` has been shifted right by 4, which would give \(\hat{f}/16-1/32\), which is the fraction for the even case. 

For the odd case, we must subtract this from \((2^{22}-1)2^{-27}=1/32+2^{-27}\) to get \(-\hat{f}/16+1/16-2^{-27}\).

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
.50 | 0 | 00 | 01000000 | 010 000 000 000 000 000 000 000 000
.70 | 0 | 00 | 01000000 | 010 111 001 100 110 011 001 100 101
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
Aside from the \(2^-{27}\) in the even case, these correspond to the linear approximation determined earlier.  Why the \(2^{-27}\)? We want to subtract a multiple of the fraction in the even case by adding, which requires a twos complement, but the 709 only has ones complement, which will be off by 1, or \(2^{-27}\). Including the \(2^{-27}\) in the lumped constant will correct the odd case, while leaving it out would leave the even case correct. When \(x=1\) the characteristic is odd so including the \(2^{-27}\) in the odd case makes the linear approximation exact.
```
 x  | S | QP | CHAR     | FRACTION
.25 | 0 | 00 | 10000000 | 100 000 000 000 000 000 000 000 000
.33 | 0 | 00 | 10000000 | 100 100 011 110 101 110 000 101 001
.50 | 0 | 00 | 10000000 | 101 110 000 000 000 000 000 000 001
.70 | 0 | 00 | 10000000 | 110 101 001 100 110 011 001 100 110
```

Finally,
```
       STO COMMON+1          STORE Y(2)                                MISRT1025
```
replaces `COMMON+1` with the linear approximation.

### Heron's Square Root Formula

The square root routine is based on Heron's square root formula. Given an approximation \(y_n\approx \sqrt{x}\), Heron's formula gives a better approximation
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
&=\frac{y_n^2+x}{2y_n}-\sqrt{x}\\\\
&=\frac{y_n^2+x-2y_n\sqrt{x}}{2y_n}\\\\
&=\frac{\left(\sqrt{x}+\epsilon_n\right)^2+x-2\left(\epsilon_n+\sqrt{x}\right)\sqrt{x}}{2y_n}\\\\
&=\frac{x+2\epsilon_n\sqrt{x}+\epsilon_n^2+x-2\epsilon_n\sqrt{x}-2x}{2y_n}\\\\
&=\frac{\epsilon_n^2}{2y_n}\\\\
&=\frac{1}{2}\frac{\epsilon_n^2}{\epsilon_n+\sqrt{x}}\\\\
&\approx \frac{\epsilon_n^2}{2\sqrt{x}}&\text{for }\epsilon_n\text{ small compared to }\sqrt{x}.
\end{align*}
$$
Thus
$$
\epsilon_n\approx \epsilon_0 \left(\frac{\epsilon_0}{2\sqrt{x}}\right)^{2^n}.
$$
By picking a good initial approximation to the square root, only a few applications of Heron's method are needed.
