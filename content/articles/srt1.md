---
title: "SHARE Square Root for the IBM 704"
date: 2024-07-06T00:00:00-00:00
math: true
draft: true
---
# Abstract

The IBM 704 SHARE library floating point square root routine `MISRT1` executes only two floating point instructions. The IBM 704 was introduced in 1954. It was the first commercial computer with core memory and hardware support for floating point. The first version of FORTRAN was developed for the 704 and some 704 features were influenced by the needs of FORTRAN. Here the implementation of `MISRT1` is described, illustrating of the techniques used in early programming.

# The IBM SHARE tape

In 1955 a group of IBM 704 users formed [SHARE](https://www.share.org/) to share programs and other information related to IBM computers. One of the founders was [Roy Nutt](https://history.computer.org/pioneers/nutt.html), who wrote the [SHARE Assembler Program (SAP)](https://sky-visions.com/ibm/704/uasap.pdf#page=2) that was used to assemble many of these programs and routines, including `MISRT1`. SHARE distributed programs on magnetic tape. [Paul Pierce's Computer Collection](https://piercefuller.com/collect/index.html) includes recovered contents of a number of early computer tapes, including the [IBM SHARE Library](https://www.piercefuller.com/library/share.html). The [Yale SHARE Tape 2 29-508](https://www.piercefuller.com/library/kyu2.html) contains the `MISRT1` square root routine described below.

The files on Paul Pierce's site are almost the raw tape contents. Some knowledge of magnetic tape conventions, reverse engineering and pre-processing not described in this description is required to convert `MISRT1` into the form shown here.

## MISRT1 header

Each routine on the SHARE tape has a header that contains meta-information about the routine. There is quite a bit of variability among the headers.

For `MISRT1` the header record is:
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
 - `0045` The last card, numbered from `0000`.

## MISRT1 source code

The complete SAP source code for the `MISRT1` square root is:
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

# Simplified description of the IBM 704

`MISRT1` will be described a few instructions at a time, with new operations defined as they are first encountered. A simplified description of the architecture provides the necessary background for understanding the operations. A detailed description, using 1954 terminology, can be found in [IBM 704 electronic data-processing machine](https://bitsavers.org/pdf/ibm/704/24-6661-2_704_Manual_1955.pdf). Some additional references are provided at the end of this section.

## Arithmetic

There are two special-purpose arithmetic registers:
 - `AC` is a 38 bit *accumulator* used as a source and destination for most operations,
 - `MQ` is a 36 bit *multiplier-quotient* register used for operations that require two registers, such as multiplication and division.

Bit positions in the arithmetic registers are labeled corresponding to their roles in the sign-magnitude representation of signed integers. In this representation, the left-most bit position is called `S` and contains the sign of the integer. The remaining bit positions hold the magnitude. The low 35 bit positions, from high to low, are `1` through `35`. The two additional bit positions in `AC` are `Q` and `P`:
```
AC: S Q P 1 2 3 4 ... 35
MQ: S     1 2 3 4 ... 35
```

When describing bit manipulation, the notation `R[S,1-11]` denotes the value in the high 12 bits of a 36 bit register `R`. When describing mathematical properties, \(R_{S,1-12}\) designates the same bits as an unsigned integer in the register \(R\).

Values were normally written in octal. The bit in the `S` position was either written as `+` or `-` for `0` and `1` respectively, or included in the high order octal digit, as in the four constants at the end of `MISRT1`. Here, binary will often be used because it makes it easier to see the effects of shifts.

## Memory

The IBM 704 memory (also called *registers*) comes in three configurations: 4096, 8192, or 32,768 36 bit words of memory. Bit positions in a word are the same as in `MQ`: `S` and `1` through `35`. For an address `A`, `C(A)` denotes the memory value at `A` masked down to 12, 13 or 15 bits according to the configured memory size. Because of this masking, all addresses correspond to valid memory on the IBM 704.

Three *index* registers hold 12, 13, or 15 bits, depending on the memory configuration. These registers are read/written in a somewhat unusual manner. A three bit *tag* serves as a bit mask to select a subset of the three registers. When writing, all the selected registers are written. When reading, the contents of the selected registers are ORed together. If the tag is 0, no registers are written and 0 is read. `X(T)` denotes the indexing for a tag `T`.

## Instructions

Each instruction is a full word. The `IC`, or *instruction counter*, contains the address of the next instruction to be executed. A clock cycle is 12 microseconds. Most instructions require two cycles, but some require more. The manual states that about 40,000 instructions/second are executed in normal use.

Bit positions `18-20` of a word are called the `tag` and positions `21-35` are called the `address`, even for words that do not contain an instruction. The tag and address are used to compute an effective address `Y` for *indexable* operations.

Even though the processor was directly implemented in logic, it will be easier to describe its operation in a C-like pseudo-code. The basic instruction loop is:

```
while(!halted) {
  IR = C(IC++);
  Y = IR[address] - X(IR[tag]);
  execute I[opcode];
}
```

Here `IR` is the instruction register, holding the instruction read from memory at the location of `IC`, and `Y` is the effective address, used by most operations. It is the address field of the instruction with the index of the tag field *subtracted*.

## Indicators

*Indicator* bits are set by some operations. They remain set until tested. The *accumulator overflow* and *multiplier-quotient overflow* indicators are used in `MISRT1`.

## Integer representation

The full word integer representation is sign-magnitude, not twos complement. Position `S` specifies the sign, `0` for `+` and `1` for `-`. The remaining bits specify the magnitude (absolute value). A 36 bit word \(W\) represents the integer \((1-2W_S)W_{1-35}\). An integer is 0 when the magnitude is 0, regardless of the sign. When no sign is written, `+` is assumed.

When words representing integers are written using octal notation, they may be written with a separate sign or with `S` folded into the first digit. The values \(-2\) through \(2\) follow, written using the signed and unsigned formats and are compared with the twos complement representation:
```
                                     Twos
         Signed      Unsigned      Complement
-2 : -000000000002 400000000002   777777777776
-1 : -000000000001 400000000001   777777777777
-0 : -000000000000 400000000000   000000000000
+0 : +000000000000 000000000000   000000000000
+1 : +000000000001 000000000001   000000000001
+2 : +000000000002 000000000002   000000000010
```

## Floating point representation

The floating point representation uses 36 bits containing three unsigned integer fields: a one bit *sign* \(s\) in the sign bit `S`, an eight bit *characteristic* \(c\) in bits 1 through 8, and a 27 bit *fraction* \(f\) in bits 9 through 35. The sign is used in the same way as with the integer representation. The characteristic is the binary exponent biased by 128 and the fraction is scaled by \(2^{-27}\).

The floating point value \(v\) associated with a sign of \(s\), characteristic \(c\), and fraction \(f\) is
$$
\begin{aligned}
v&=(1-2s) 2^{c-128}2^{-27}f\\
&=\hat{s}2^{\hat{c}}\hat{f}
\end{aligned}
$$
where
$$
\begin{aligned}
\hat{s}&=1-2s\\
\hat{c}&=c-128\\
\hat{f}&=2^{-27}f.
\end{aligned}
$$
The values without the hats are the value of the bit fields, the values with the hats are the values the bit fields represent.

A floating point value can have more than one representation because increasing the characteristic and shifting the fraction to the right does not change the represented value. For example, three representations of 1.0 are:
```
VALUE   S    C        F
1.0   | 0 | 201 | 400000000
1.0   | 0 | 202 | 200000000
1.0   | 0 | 203 | 100000000
```
The representation is called *normalized* when
 - \(f=0\) and \(c=0\) 
 - \(f \ge 2^{26}\), i.e. the first bit of the fraction, \(f_1\) is 1, so that \(\frac{1}{2}\le\hat{f}<1\).

When `AC` holds a normalized floating point representation, `Q` and `P` will be 0.

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
The floating point representation was designed so that the floating point and integer representations of 0 are the same and so that integer comparison can be used to compare normalized floating point values.

## Additional 704 information

 - [IBM 701](https://bitsavers.org/pdf/ibm/701/) Scanned 701 information. The 704 started as a revision to the 701.
 - [An Interview with GENE M. AMDAHL](https://conservancy.umn.edu/server/api/core/bitstreams/5f173919-6c38-4b0d-9812-24a3734fd35f/content) Architect of the 704.
 - [IBM 704](https://bitsavers.org/pdf/ibm/704/) Scanned 704 information.
   - [IBM 704 electronic data-processing machine](https://bitsavers.org/pdf/ibm/704/24-6661-2_704_Manual_1955.pdf).
 - Scanned information about follow-on processors in the same family:
   - [IBM 709](https://bitsavers.org/pdf/ibm/709/) faster, added additional capabilities, particularly in I/O.
   - [IBM 7090](https://bitsavers.org/pdf/ibm/7090/) faster with transistors.
   - [IBM 7094](https://bitsavers.org/pdf/ibm/7094/) even faster.

# SAP source format

Each line contains exactly 80 characters and corresponds to one 80 column IBM punched card. A typical card (produced by [mass:werk](https://www.masswerk.at/card-readpunch/) )would look something like:

![MISRT1009 Card](images/MISRT1009.png)

Cards are divided into fields based on column positions:
- [1-6] An optional label. When read, all blanks are removed, so `OLD X` and `OLDX` are the same.
- [7] Blank.
- [8-10] Three character opcode or pseudo-opcode.
- [11] Blank.
- [12-80] The *variable field*, up to the first blank. The variable field contains comma-separated parameters. Anything after the blank is a comment.

It was common to use columns on the right side of the card to add sequencing information. Since this was in the comment area it was ignored by the assembler. Here they will serve as a way to reference particular lines. When a deck of cards was accidentally dropped, a card sorter, a common piece of data processing equipment at the time, could be used to put them back in order. For the 46 cards in `MISRT1`, sorter setup time would have made sorting them by hand faster. 

## Using MISRT1 in a program

Understanding programs from the 1950s can be challenging. For example, even though \(\sqrt{}\) is a function, `MISRT1` is not a function in the current sense. A function returns to the instruction after its caller it completes, while invoking an IBM 704 subroutine is more like [continuation passing](https://en.wikipedia.org/wiki/Continuation). A subroutine is branched to and provided a pointer to a call vector containing arguments, space for return values, and branch targets. When the subroutine completes its work it will branch to one of the branch targets after setting the appropriate return values for that target. Although this may seem like a strange way to do things, it is similar to several IBM 704 instructions that have multiple return points. Even on modern architectures conditional branches have two return points and compilers use similar representations during their analysis.

SHARE and IBM established some conventions so that subroutines could be used reliably. One was that `X(4)` was always used for the call vector. Two more conventions were that the first argument and first result were passed in `AC` rather than the call vector.

Although there was no control stack and function frame, most subroutines still required temporary storage. With at most 32,768 words of memory, programs would often stash small temporary values in unused bits of instructions. When whole words were required, the SHARE convention was that a block of storage starting at a location named `COMMON` would be used. Subroutines would indicate how much space in `COMMON` they required, so the programmer could reserve space for the maximum size required by all of its subroutines.

All symbols in SAP have global scope. If subroutines used symbolic addresses to make the subroutine implementation easier to understand and maintain there would be name collisions between a program and all the subroutines it was using. Another convention was that subroutines should only use a label for their entry point and reference all subroutine addresses relative to the entry point or relative to the instruction counter.

The first few lines of `MISRT1` describe use of the routine:
```
       REM SQUARE ROOT, FLOATING POINT                                 MISRT1000
       REM   ACCURACY= MAX. ERROR OF HALF LAST BIT                     MISRT1002
       REM   TIME (AV.)= 102 CYCLES= 1.224 M.S.                        MISRT1001
       REM   SPACE= 37 + 2 COMMON CELLS                                MISRT1003
       REM   ALARM RETURN= NEGATIVE ARGUMENT                           MISRT1004
       REM   CALLING SEQUENCE= TSX SQRT,4                              MISRT1005
       REM                     (ALARM RETURN)                          MISRT1006
       REM                     (NORMAL RETURN)                         MISRT1007
```

`REM` is a pseudo-op for a *remark* or comment, so this entire block is documentation and will not take up any storage in the assembled program. It states that the subroutine takes up 37 words of storage and requires `COMMON` to contain at least two words.

The subroutine uses the first continuation when there is a negative argument and the second continuation when the computation is successful. The second continuation is at the end of the call vector and is just the next instruction to be executed. The first continuation would be a branch to error-handling code.

Following is a simple use of `MISRT1` to compute \(\sqrt{2}\):

```
       CLA TWO
       TSX SQRT,4
       TRA ERROR
       REM  DO SOMETHING WITH RESULT
       ...
TWO    DEC 2.0
```

```
// Clear and add
// Clear the accumulator and add the value at the effective address
CLA {
  AC = C(Y);
}
```
The first operand, `TWO` is the address. The second operand is the tag, which is 0 when omitted. `TWO` is defined at the end as a floating point 2.0 constant, using the `DEC` *decimal* pseudo-op. The result is that `AC` contains the floating point representation of 2.0.

The `TSX` instruction, *transfer and set index*, is not indexable. Pseudocode is:
```
// Transfer and set index
// Set the specified index to the twos complement of the location of the instruction and
// transfer to the specified address.
TSX {
  I = C(IC);          // Normal instruction execution
  X(I[tag]) = 1 - IC; // Twos complement of address of TSX instruction
  IC = I[address];    // Branch to address
}
```
Following the `TSX` instruction are the two return points, the first for *alarm return* when `AC` is negative, the second for normal return. The `TRA` is just an unconditional branch:
```
// Transfer
TRA {
  IC = C(Y);
}
```

An indexable branch to `1,4` will branch to the warning address, while a branch to `2,4` will branch to the normal continuation address.

# The MISRT1 routine

Given a normalized floating point representation \([s,c,f]\) for \(x\) in `AC`, the routine returns with a normalized floating point representation \([s, d, g]\) for \(y=\sqrt{x}\) in `AC` if \(x\not < 0\) and indicates an error if \(x < 0\).
 
## Special case for 0

The first instruction is the entry point, with the label `SQRT`:
```
  SQRT TZE 2,4               X ZERO NORMAL RETURN                      MISRT1009
```

```
// Test zero
// Branch to the effective address if the magnitude of AC is 0
TZE {
  if (0 == AC[Q-35]) then {
    IC = C(Y);
  }
}
```
The effective address is word 2 of the call vector, the normal return. Since `AC` has not been changed, the result is 0.

## Error for negative argument

```
       TMI 1,4               X NEGATIVE ALARM RETURN                   MISRT1010
```

```
// Test minus
// If the sign of AC is 1 branch to the effective address.
TMI {
  if (1 == AC[S]) then {
    IC = C(Y);
  }
}
```
The effective address is word 1 in the call vector, the alarm address. Since `AC` has not been changed, exception handling can take action based on the value of the argument.

## Taking advantage of the representation

At this point it is known that \(x>0\) and thus \(y > 0\).

Since \(x=y^2\), \(\hat{c}\) and \(\hat{f}\) can be computed from \(\hat{d}\) and \(\hat{g}\):
$$
2^{\hat{c}}\hat{f}=x=y^2=2^{2\hat{d}}\hat{g}^2.
$$
Since \(f\) must be in normalized form there are two cases for computing the exponent and fraction:
1) If \(\frac{1}{2}\le \hat{g}<\sqrt{\frac{1}{2}}\) then \(\frac{1}{4}\le\hat{g}^2<\frac{1}{2}\). Since \(\hat{g}^2\) is not in the range of a normalized fraction, the exponent must be reduced by 1 it and the fraction multiplied by 2 and to put the fraction in normalized form. Then \(\hat{c}=2\hat{d}-1\) and \(\hat{f}=2\hat{g}^2\).
2) If \(\sqrt{\frac{1}{2}}\le \hat{g} < 1\) then \(\frac{1}{2}\le\hat{g}^2<1\) and \(\hat{g}^2\) is normalized so \(\hat{c}=2\hat{d}\) and \(\hat{f}=\hat{g}^2\).

The two cases can be combined since \(c_8\) is 1 in the first case and 0 in the second:
$$
\begin{align*}
\hat{c}&=2\hat{d}-c_8\\
\hat{f}&=2^{c_8}\hat{g}^2.
\end{align*}
$$

For the square root \(\hat{d}\) and \(\hat{g}\) are needed.
$$
\begin{align*}
\hat{d}&=\frac{\hat{c}+c_8}{2}\\
d&=\frac{c+c_8}{2}+64\\
\hat{g}&=\sqrt{2^{-c_8}\hat{f}}.
\end{align*}
$$

## Setup

The following explanations include symbolic and actual binary results from [simh IBM 704 emulation](https://github.com/open-simh/simh) of the bit manipulations corresponding to the two fraction limits and one intermediate fraction for each of the two fraction ranges. Symbolic representations of the characteristic and fraction are in terms of the original characteristic bits `C` and fraction bits `F`. For example, if `C = 01101100` then `C[2-5]` is `1101` and `C[1-5]` is `01101`. Literals may be interspersed, as in `0 C[2,3] 1 = 0111`.

At the start of instruction `MISRT1011`, `AC` holds a positive normalized floating point value, such as the following:
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

The next instructions to be executed are:
```
       ANA SQRT+33           STORE 26 BIT X= Y*Y TO ALLOW ADD          MISRT1011
       STO COMMON              SHIFT AND ROUND FOR AVERAGES            MISRT1012
```
where `SQRT+33` is defined as an octal value by the pseudo-op `OCT`:
```
       OCT 777777777776      MASK FOR 27TH BIT CLIP                    MISRT1042
```
The operation definitions are:
```
// AND to accumulator
ANA {
  AC &= C(Y);
}

// STORE AC[S,1-35] in the effective address
STO {
  C(Y) = AC[S,1-35];
}
```
The two line comment is one sentence. By truncating the 27 bit fraction to 26 bits, the computation of the average in Heron's method (described below) will be able to be performed with integer arithmetic.

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
and `COMMON` contains
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
This time `AC` is ANDed with:
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

---

Both the even and odd exponent cases will be handled by the same instructions.

```
       ARS 1                                                           MISRT1014
```
```
// Accumulator right shift
// Shift AC[Q-35] right by the address field
ARS {
  AC[Q-35] >>= (IC[address] % 256);
}
```
AC (except for the sign) is shifted right by 1. Bit 9, the first bit of the fraction, now contains the low bit of the characteristic.
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

---

```
       ADD COMMON                                                      MISRT1015
```
```
// Add
// Add the value in the effective address to AC
ADD {
  AC += C(Y);
}
```
Since `COMMON` is normalized, the first bit of the fraction is 1. If `AC[9] = 1` (odd \(c\)) then adding `COMMON` to `AC` will add 1 to the first bit of the fraction, turning it to 0, and carry into the characteristic, adding 1 to it. This may generate a carry into `AC[P]`. It is simplest in this section to treat the characteristic as `AC[Q-8]`.  The characteristic is even and \(c+c_8\) and the fraction is \(\hat{f}-\frac{c_8}{2}\).
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

---

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

The computation of \(d\) is almost complete, but what is going on with the fraction? This is the beginning of the first linear approximation \(\hat{g}_0\) of \(\hat{g}\). 

Recall that there are two cases for the fraction, one for an odd exponent and one for an even exponent. In each case, \(\frac{1}{2}\le \hat{f}<1\), but with odd exponents \(\sqrt{\frac{f}{2}}\) must be computed and with the even exponent \(\sqrt{\hat{f}}\). These can be combined by computing \(\sqrt{s\hat{f}}\) where \(s\) is \(\frac{1}{2}\) for odd exponents and 1 for even exponents. The linear approximation must match \(\sqrt{s\hat{f}}\) when \(\hat{f}\) is \(\frac{1}{2}\) and 1. Then
$$
\begin{align*}
\hat{g}_0&=\sqrt{\frac{s\hat{f}}{2}}+\left(\hat{f}-\frac{1}{2}\right)\left(\sqrt{s\hat{f}}-\sqrt{\frac{sf}{2}}\right)\\
&=\sqrt{s}\left(\left(2-\sqrt{2}\right)\hat{f}+\sqrt{2}-1\right)\\
&=\begin{cases}
\hat{f}\left(\sqrt{2}-1\right)+1-\frac{\sqrt{2}}{2}&\text{for odd exponents }(s=\frac{1}{2})\\
\hat{f}\left(2-\sqrt{2}\right)+\sqrt{2}-1&\text{for even exponents }(s=1).
\end{cases}
\end{align*}
$$
Both cases involve a floating point multiplication (17 cycles) and addition (7 cycles). Since this is already an approximation, it can be approximated a little faster by using integer operations, in particular adds and shifts, which are 2 cycles each. Since \(16\sqrt{2}\approx 23\), \(\sqrt{2}\) can be replaced with \(\frac{23}{16}\) to get:
$$
\hat{g}_0=
\begin{cases}
\frac{7}{16}\hat{f} + \frac{9}{32}&\text{odd exponent}\\
\frac{9}{16}\hat{f}+\frac{7}{16}&\text{even exponent}.
\end{cases}
$$

---

```
       ALS 10                                                          MISRT1018
```
```
// Accumulator left shift
// Shift AC (except the sign) by the address
ALS {
  AC[Q-35] <<= IC[address];
}
```

`AC` is shifted left by 10 bits. If a 1 bit is shifted into or through `P` the overflow indicator is set.

The fraction is shifted so that its first bit is in `AC[Q]`. It will be simplest to treat `AC[Q-35]` as one field in this step:
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

---

```
       PBT                                                             MISRT1019
       COM                                                             MISRT1020
```
```
// P Bit Test
// Skip the next instruction if P is 1
PBT {
  if (1 == AC[P]) {
    IC++;
  }
}

// Complement magnitude
COM {
  AC[Q-35] = ~AC[Q-35];
}
```
If \(c_8=0\) the next instruction is skipped. The `COM` operation complements `AC[Q-35]`.
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

---

```
       ARS 13                                                          MISRT1021
```
The fraction had been shifted left by 10, and is now shifted right by 13, shifting the possibly complemented previous fraction right by 3:
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

---

Bits 4 and 5 of the fraction are cleared by:
```
       ANA SQRT+35                                                     MISRT1022
```
where `SQRT+35` is:
```
       OCT 000017777777      MASK FOR CORRECTION                       MISRT1044
```

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

For the odd case, subtract this from \((2^{22}-1)2^{-27}=\frac{1}{32}+2^{-27}\) to get $$-\frac{\hat{f}}{16}+\frac{1}{16}-2^{-27}.$$

---

```
       ADD COMMON+1                                                    MISRT1023
```
Recall that the fraction in `COMMON+1` is \((\hat{f}-c_8/2)/2\). In the even case,
$$
\begin{align*}
\frac{\hat{f}}{2}-\frac{c_8}{4}+\frac{f}{16}-\frac{1}{32}&=\frac{9}{16}f-\frac{1}{32}&c_8=0,\\
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

---

```
       ADD SQRT+36           FINISH CORRECTION AND ADJUST CHAR         MISRT1024
```
where `SQRT+36` is
```
      OCT 100360000001      LUMPED CONSTANT                           MISRT1045
```
This constant is:
```
S | CHAR     | FRACTION
0 | 01000000 | 011 110 000 000 000 000 000 000 001
```
The characteristic is 64, the amount still needed to add to the characteristic to get \(d\). The fraction is \(15/32+2^{-27}\). Adding to the previous fraction gives:
$$
\begin{align*}
\frac{9}{16}f-\frac{1}{32}+\frac{15}{32}+2^{-27}&=\frac{9}{16}f+\frac{7}{16}+2^{-27}&c_8=0,\\
\frac{7}{16}f-\frac{3}{16}-2^{-27}+\frac{15}{32}+2^{-27}&=\frac{7}{16}f+\frac{9}{32}&c_8=1.
\end{align*}
$$
Aside from the \(2^-{27}\) in the even case, these correspond to the linear approximation determined earlier.  Why the \(2^{-27}\)? In the even case a multiple of the fraction is to be subtracted by adding, which requires a twos complement. The 704 only has ones complement, which will be off by 1, or \(2^{-27}\). Including the \(2^{-27}\) in the lumped constant corrects the odd case, while leaving it out leaves the even case correct. When \(x=1\) the characteristic is odd so including the \(2^{-27}\) in the odd case makes the linear approximation exact.
```
 x  | S | QP | CHAR     | FRACTION
.25 | 0 | 00 | 10000000 | 100 000 000 000 000 000 000 000 000
.33 | 0 | 00 | 10000000 | 100 100 011 110 101 110 000 101 001
.49 | 0 | 00 | 10000000 | 101 110 000 000 000 000 000 000 000
.50 | 0 | 00 | 10000000 | 101 110 000 000 000 000 000 000 001
.70 | 0 | 00 | 10000000 | 110 101 001 100 110 011 001 100 111
.99 | 0 | 00 | 10000000 | 111 111 111 111 111 111 111 111 111
```

---

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

## Error of the linear approximation

The linear approximation will be less than \(\sqrt{s\hat{f}}\) between \(\frac{1}{2}\) and 1, so the error \(\epsilon_0\) will be:
$$
\begin{align*}
\epsilon_0&=\sqrt{s\hat{f}}-\sqrt{s}\left(\left(2-\sqrt{2}\right)\hat{f}+\sqrt{2}-1\right)\\
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
\epsilon_{n+1}&=y_{n+1}-\sqrt{x}\\
&=\frac{1}{2}\left(y_n+\frac{x}{y_n}\right)-\sqrt{x}\\
&=\frac{y_n^2+(y_n-e_n)^2}{2y_n}-(y_n-\epsilon_n)\\
&=\frac{2y_n^2-2y_n\epsilon_n+\epsilon_n^2-2y_n^2+2y_n\epsilon_n}{2y_n}\\
&=\frac{\epsilon_n^2}{2y_n}
\end{align*}
$$
Whether \(\epsilon_0\) is negative or positive, \(\epsilon_n\) is positive for \(n>0\). The linear approximation is exact for odd exponents when \(\hat{f}=1\) and otherwise less than \(\sqrt{x}\).

The error after \(n\) iterations can be estimated as:
$$
\begin{align*}
\epsilon_n&=\frac{1}{2}\frac{\epsilon_n^2}{\epsilon_n+\sqrt{x}}\\
&\approx \frac{\epsilon_n^2}{2\sqrt{x}}&\text{for }\epsilon_n\text{ small compared to }\sqrt{x}\\
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

---

```
       CLA COMMON            ITERATE Y(2)                              MISRT1026
```
`AC` now contains \(x\).

---

```
       FDP COMMON+1                                                    MISRT1027
```
```
// Float divide or proceed
FDP {
  MQ = AC/C(Y); // float divide
  AC -= MQ*C(Y);
}
```
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

---

In the next step the first approximation, `COMMON+1`, and the division result, `FDP`, must have the same exponent. Notice how this is true for all the examples. To see that it is true in general, the even and odd exponent cases must be examined. In both cases, \(\frac{1}{2}\le \hat{g}_0< 1\).

In the even case to prevent a change in exponent, ensure that \(\frac{1}{2}\le\frac{\hat{f}}{\hat{g}_0}<1\). For the lower bound,
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
\frac{\hat{f}}{\hat{g}_0}&<1\\
\frac{\hat{f}}{\frac{9}{16}\hat{f}+\frac{7}{16}+2^{-27}}&<1\\
\hat{f}&<\frac{9}{16}\hat{f}+\frac{7}{16}+2^{-27}\\
\frac{7}{16}\hat{f}<\frac{7}{16}+2^{-27}\\
\hat{f}<1 < 1+\frac{2^{-24}}{7}.
\end{align*}
$$
In the odd case, to ensure the exponent is increased by 1, ensure that \(1\le\frac{\hat{f}}{\hat{g}_0}<2\).

For the lower bound,
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

---

There is one other detail to be dealt with. The above is for exact arithmetic. For the `.99` case, `COMMON+1` is 27 1s. If we had not cleared the low bit of \(f\), it would also have been 27 1s and division would have result in a fraction of 1. This would have been shifted right one and the exponent increased. The `ANA` in instruction `MISRT1011` ensures that the value in `COMMON` is less than `COMMON+1` so that the division is not 1 and the exponent is not increased.

```
       CLA COMMON+1                                                    MISRT1028
       STQ COMMON+1                                                    MISRT1029
       ADD COMMON+1            AVERAGE                                 MISRT1030
```
```
// Store MQ
STQ {
  C(Y) = MQ
}
```

\(\frac{x}{y_0}\) and \(y_0\) must be averaged. Unfortunately, 704 data paths were limited and there isn't a way to add `MQ` and `AC` without going through memory.

But `ADD` is an integer add, not a floating point add. How can this work? The exponents of \(y_n\) and \(\frac{x}{y_n}\) will both be the same, \(d\), so the points are aligned and there is no need to shift fractions and adjust exponents during the addition. The value \(2d\) will be in `AC[P-8]`. This is even, so `AC[8]` will be 0 unless there is a carry from adding the fractions. Since fraction bit 1 of both `COMMON+1` and `FDP` are 1, there will be a carry and `AC[8]` will be 1. The result is the normalized floating point sum, shifted left by 1 and needs to be shifted right:

```
       LRS 1                                                           MISRT1031
```
```
// Long right shift
LRS {
  AC[Q-35]MQ[1-35] >>= (Y % 256);
  MQ[S] = AC[S];
}
```

---

```
       RND                     PREVENT CHAR DISAGREEMENT               MISRT1032
```
```
// Round
// Add MQ[1] to AC[Q-35]
RND {
  AC[Q-35] += MQ[1]'
}
```
Since this is immediately followed by a second application of Heron's formula, which will more than correct for an error in the low order bit of the fraction, this would seem to be an unnecessary operation, and it would be if floating point arithmetic were being used. But on closer inspection, notice how for the ".99" case, the combination of the `ANA` in `MISRT1011` and the `RND` in `MISRT1032` work together to ensure that the quotient will not increase the exponent and the final fraction at the end of this application of Heron's formula will be greater than `COMMON` for the next application of Heron's formula.
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

---

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
```
// Transfer on overflow
TOV {
  if (1 == overflow) {
    IC = Y;
    overflow = 0;
  }
}
```
Computing a square root should not result in an overflow, but the overflow indicator is always set in the `ALS 10` instruction at `MISRT1018`. This clears the overflow and continues execution at the normal (second) continuation.