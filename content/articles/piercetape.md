---
title: "Reading IBM 70xx tapes from Paul Pierce's Computer Collection"
date: 2024-09-19T20:53:48-07:00
draft: true
math: true
---
# Old IBM computer data

Before IBM made computers they were a major provider of electromechanical punched card data processing equipment. The construction and use of this equipment influenced early computer hardware and software. Input and output were performed by slightly modified or reconfigured punched card equipment. Since computers could run significantly faster than punched card equipment, faster magnetic tape drives were developed that could store data more compactly. Humans put data on punched cards and the contents of the cards were transferred to magnetic tape. Most of those tapes long gone, but Paul Pierce's Computer Collection[^PaulPierce] contains the raw data from a number of tapes. Here we describe how to understand the raw data from early IBM computer tapes on Paul Pierce's site.

# The tape

For years media depicted computers as a collection of punched card equipment, spinning tapes on drives, blinking lights and printers. Today, many people would not recognize the depictions as a computer. Paul Pierce includes photographs of the recovered tape reels on his site:

![Tape](https://piercefuller.com/library/p0002133.full.jpg)
The tape reel is about 11" in diameter. Tape is 1/2" wide and up to 2400' long. The amount of data that could be stored on a tape depended on the record size. One tape could hold as much data as 11,000 punched cards.[^IBM704] Converted to bytes, this is the less impressive 4MB, but massive compared to the maximum sized 144K of memory.

To mount the tape on a tape drive, the white rim is removed by unsnapping the black fastener shown at the bottom of the picture, exposing the tape. The reel is attached to the left side of a drive, unwound a bit and run past the heads and wound onto an empty take-up reel on the right:

![Tape drive](https://upload.wikimedia.org/wikipedia/commons/3/3b/IBM_729_restored.jpg)  
[__Tape drive__](https://commons.wikimedia.org/wiki/File:IBM_729_restored.jpg)

In later tape drives the drive the tape only had to be mounted on the left and the drive would handle the rest.

In the early days, one tape held on tape file. As data always expands to fill the available storage, the format was later extended to support multi-volume files.

# Format of the recovered tape files

Data was written on the tape 7 bits at a time and grouped into records. The recovered data uses one byte for each seven bits written, setting the high bit to indicate the start of a record.

For example, for the tape shown above, the `adc00037.data` starts as:
```
E3 0A 50 65 71 50 0A 41 41 0A 50 50 50 50... C0 40 40 40 40 40...
```
In `E3`, the high bit is set, indicating the beginning of a record. If we use `[` and `]` to group records, the 7 bit values are:
```
[63 0A 50 65 71 50 0A 41 41 0A 50 50 50...][40 40 40 40 40 40...]
```

If you are familiar with ASCII hex codes, the values look ASCII-like, but this is just coincidence. ASCII didn't exist when the tape was written. Text was encoded in a family of formats called *BCD*, which was based on punched cards, the predominant form of IBM data storage when tape was introduced.

# Punched cards

The idea of punched cards may have gotten its start in the 9th century with the Banū Mūsā brothers, who built an organ that played music according to pins on a cylinder.[^WikiIngenious].

## Bouchon Loom

Much later in 1725 France, Bouchon, the son of an organ maker, produced an automatic loom whose patterns were programmed on a paper tape with holes that caused threads to be lifted.[^WikiBouchon]

![Bouchon Loom](https://upload.wikimedia.org/wikipedia/commons/5/56/Basile_Bouchon_1725_loom.jpg)  
[__Bouchon Loom__](https://commons.wikimedia.org/wiki/File:Basile_Bouchon_1725_loom.jpg)

## Jacquard Loom

 Correcting mistakes and repairing the paper tape was difficult. In 1805, Jacquard replaced the tape with a chain of punched cards sewn together, allowing mistakes and repairs to be limited in scope to a single card.[^WikiJacquard]

 ![Jacquard Loom](https://upload.wikimedia.org/wikipedia/commons/0/09/Jacquard.loom.cards.jpg)  
 [__Jacquard Loom__](https://commons.wikimedia.org/wiki/File:Jacquard.loom.cards.jpg)

## Hollerith

In 1884, Hollerith combined the idea of recording data by punching holes in paper, such as railroad tickets, with the sensing of holes to control a loom to produce a tabulating machine.[^WikiHollerith] His company was one of several companies that combined and became IBM. The original tabulating machines just counted occurrences of holes in particular positions.

![Hollerith Machine](https://upload.wikimedia.org/wikipedia/commons/4/4e/HollerithMachine.CHM.jpg)  
[__Hollerith Machine__](https://commons.wikimedia.org/wiki/File:HollerithMachine.CHM.jpg)


Newer equipment was developed that could treat the position of a hole in a column as a value. Groups of columns in a deck of cards could be summed and printed. In 1928, IBM introduced the once familiar punched card with 80 columns and 10 rows:[^WikiPunchedCard]

![Blank Card](images/blank.png)

Each column can hold one digit, and columns with no punches are blank.

![Digits Card](images/digits.png)

Punched card equipment read cards by passing them over read brushes that sensed the holes in one row or column at a time, synchronously with a rotary switch that acted as a clock to control the action of other parts of the equipment. For example, if the equipment was reading horizontally from the bottom of the card, the machine would go through 9-time, 8-time, etc. A machine that printed what was on the cards would use type bars, a horizontal bar with characters 0 through 9 and a blank. All type bars would start lowered, in the 9 position. At 9 time, the type bars for any columns with a 9 punched would stop moving, leaving them positioned to print a 9. The other bars would continue moving. At 8 time, the type bars for columns with an 8 punched would stop moving. Any bars that were still moving after 0 time would be in a blank position. Then hammers would force the bars against an ink ribbon inking the characters. All the type bars would reset and the next card would move across the brushes to be printed.

Plugboards were developed so that signals from the brush columns could be sent to different print columns.

![Plugboard](https://upload.wikimedia.org/wikipedia/commons/b/b7/IBM402plugboard.Shrigley.wireside.jpg)  
[__Plugboard__](https://commons.wikimedia.org/wiki/File:IBM402plugboard.Shrigley.wireside.jpg)

Plugboards could get quite complicated for tabulating machines.

There was demand for keeping alphabetic information on punched cards. Two unmarked rows at the top and the `0` row were used as *zone* rows. When only a digit row is punched, the column is a digit. When one digit, `1` through `9`, and a zone row is punched, the column is alphabetic:

![Alpha Card](images/alpha.png)

For alphanumeric cards (called *alphmeric*) the cards were passed over two sets of brushes. The first set of brushes determined the zone, 12, 11, 0, or none. The type bars were four times as long. During the zone times, the type bars were moved to the section of the type bar with the characters for the zone. The card then moved to the second set of brushes reading the digit positions (so 0 is read twice) and the type bars moved to the appropriate position for the character within the zone. There was no need to electrically remember the zone between the two readings since the type bar position served as the memory.


We can put the character encodings in a table with one row for each zone:

|Hollerith|0  |1  |2  |3  |4  |5  |6  |7  |8  |9  |
|:-------:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|**12**   |   | A | B | C | D | E | F | G | H | I |
|**11**   |   | J | K | L | M | N | O | P | Q | R |
|**0**    |   |   | S | T | U | V | W | X | Y | Z |
|         | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 6 | 9 |



The BCD coding is a 6 bit encoding where the two high bits specify the zone and the low 4 bits specify the digits:

|BCD   |0000|0001|0010|0011|0100|0101|0110|0111|1000|1001|
|:----:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
|**00**| 0  | 1  | 2  | 3  | 4  | 5  | 6  | 7  | 6  | 9  |
|**01**|    |    | S  | T  | U  | V  | W  | X  | Y  | Z  |
|**10**|    | J  | K  | L  | M  | N  | O  | P  | Q  | R  |
|**11**|    | A  | B  | C  | D  | E  | F  | G  | H  | I  |

Four possible encodings remained, which could be used for symbols. Data processing customers didn't usually exchange data with each other, so different sites could use different symbols as long as they kept their printers and key caps consistent internally.

In the BCD encoding, there is room for additional symbols by extending the digits. To save space, we'll switch to hexadecimal:

|BCD   |0  |1  |2  |3  |4  |5  |6  |7  |8  |9  |A  |B  |C  |D  |E  |F  |
|:----:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|**0**| 0  | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 6 | 9 |   |   |   |   |   |   |
|**1**|    |   | S | T | U | V | W | X | Y | Z |   |   |   |   |   |   |
|**2**|    | J | K | L | M | N | O | P | Q | R |   |   |   |   |   |   |
|**3**|    | A | B | C | D | E | F | G | H | I |   |   |   |   |   |   |

For punched cards, the `A` through `F` columns were specified by punching two digit rows, `8` and a digit from `2` through `7`, so that the two digits were added or ORed in the low four bits:

|Hollerith|0  |1  |2  |3  |4  |5  |6  |7  |8  |9  |8-2|8-3|8-4|8-5|8-6|8-7|
|:-------:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|         | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 6 | 9 |   |   |   |   |   |   |
|**0**    |   |   | S | T | U | V | W | X | Y | Z |   |   |   |   |   |   |
|**11**   |   | J | K | L | M | N | O | P | Q | R |   |   |   |   |   |   |
|**12**   |   | A | B | C | D | E | F | G | H | I |   |   |   |   |   |   |


Recall that on tape, `000000` could not be used in BCD mode since with even parity it was `0000000` which has no `1` to indicate data was written. 
When writing to tape, `0` was moved to `0A` and moved back during reads:

|TAPE |0  |1  |2  |3  |4  |5  |6  |7  |8  |9  |A  |B  |C  |D  |E  |F  |
|:---:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|**0**|   | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 6 | 9 | 0 |   |   |   |   |   |
|**1**|   |   | S | T | U | V | W | X | Y | Z |   |   |   |   |   |   |
|**2**|   | J | K | L | M | N | O | P | Q | R |   |   |   |   |   |   |
|**3**|   | A | B | C | D | E | F | G | H | I |   |   |   |   |   |   |

The symbols use for the IBM 704 were:
|TAPE 704|0    |1  |2  |3  |4  |5  |6  |7  |8  |9  |A  |B  |C  |D  |E  |F  |
|:---:|:---:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|**0**|     | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 6 | 9 | 0 | # | @ |   |   |   |
|**1**|blank| / | S | T | U | V | W | X | Y | Z | ‡ | , | % |   |   |   |
|**2**| _   | J | K | L | M | N | O | P | Q | R | Ô | $ | * |   |   |   |
|**3**| &   | A | B | C | D | E | F | G | H | I | Ó | . | ¤ |   |   |   |

Not all the symbols are available in Unicode. The `Ô` substitutes for a `0` with a `-` on top, and `Ó` is a `0` with a `+` on top.

The IBM 704 is the computer that first supported FORTRAN, which requires a number of symbols not listed.

|TAPE FORTRAN|0  |1  |2  |3  |4  |5  |6  |7  |8  |9  |A  |B  |C  |D  |E  |F  |
|:----------:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|**0**       |   | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 6 | 9 | 0 | = | ' | : | > |   |
|**1**       |   | / | S | T | U | V | W | X | Y | Z |   | , | ( |   |   | " |
|**2**       | - | J | K | L | M | N | O | P | Q | R | ! | $ | * |   | ; |   |
|**3**       | + | A | B | C | D | E | F | G | H | I | ? | . | ) |   | < |   |


By the mid-60s, the symbol assignments had settled on:

![029](images/card029.png)



![All Card](images/charset.png)


![KeyPunch](https://upload.wikimedia.org/wikipedia/commons/b/b9/IBM_026_from_above.mw.jpg)  
[__IBM 026 Keypunch__](https://commons.wikimedia.org/wiki/File:IBM_026_from_above.mw.jpg)




Although the goal is to store bits, bits must be stored as some type of analog signal. A process called *modulation* produces an analog signal from a stream of bits. Likewise, *demodulation* is the process of converting an analog signal back into a stream of bits. Demodulation has two components: determining when there are bits to be added to the output bit stream and determining the value of those bits. In some cases, determining when there are bits to add to the output stream comes from a source external to the signal, such as a pulse generated by the media's position or movement. When the signal is magnetic only changes in the signal, i.e. its derivative, can be sensed.

**Magnetic write and read signals**
```
WRITE: 0  0  0  1  1  1  0  1  1  0  0  0
                |--------|  |-----|
       ---------|        |--|     |-------

READ:           |           |
       ---------|--------|--|-----|-------
                         |        |
       0  0  0 +1  0  0 -1 +1  0 -1  0  0
```

In the tapes recovered by Paul Pierce, the demodulation circuits in tape drives were used to recover the bit stream. In some more recent restoration efforts, the actual analog signals are captured and digitally processed to obtain the bit streams.

The IBM 70x tapes used NRZI modulation. Abstractly, there is one 7 bit register containing the current signal values and one 7 bit register containing the next bits to be written. When the write process begins for a stream of bits, the current signal value register is initialized to 0 and the first seven bits to be written are stored in the register holding the next bits to be written. The tape begins movement at a constant rate. Seven tape heads are set to magnetize according to the values in the current signal value register, say south if their bit is 0, north if their bit is 1. A periodic clock pulse the current signal value to be XORed into the current signal register, any 1 bits cause the polarity to reverse. The next seven bits to be written must be set in the register of next bits to be written before the next clock signal occurs. If this does not happen, the current signal value register is set to 0 and the tape continues moving for a short distance, called the inter-record gap. If new data still has not arrived, the tape backs up enough that it will have time to come up to speed within the gap, and stops; otherwise the process resumes.

In the following illustration, `CLOCK` is the clock pulse for writing a bit. `DATA` is the value of the incoming data register. When `DATA` is written, `CH DATA` is set, and cleared on the next `CLOCK`. `NO DATA` is set then `CH DATA` is 0 on a clock pulse, meaning no new bit value was received. `SIGNAL` is the value written to tape.
**NRZI write modulation (1 bit)**
```
CLOCK:   0 0 0 1 0 0 0 1 0 0 0 1 0 0 0 1 0 0 0 1 0 0 0 1 0 0 0 1 ...
DATA:    0 0 0 0 0 0 1 1 1 1 0 0 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 ...
CH DATA: 1 1 1 1 0 0 1 1 0 0 1 1 0 0 1 1 0 0 0 0 0 0 0 0 0 0 0 0 ...
NO DATA: 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 1 1 1 1 1 1 1 ...
SIGNAL:  0 0 0 0 0 0 0 1 1 1 1 1 1 1 1 0 0 0 0 1 1 1 1 0 0 0 0 0 ...
```

The inter-record gaps give the computer a chance to perform processing to produce more data or to process data that has been read, but they also reduce the density of data on the tape. The IBM tape drives allowed for arbitrary record sizes, while the first UNIVAC tape drives used a different modulation and a fixed block size. By setting the current signal value to 0, the last seven bits written formed an even checksum of each of the seven tracks.

For NRZI demodulation, we need to determine where in the write signal the clock pulses occurred and what the value of the seven bits were. We also need to determine when an inter-record gap is reached. When a read is initiated, the tape starts moving. The seven tape heads produce the derivative of the signal that was written to the tape. This could be -1, 0, or +1, depending on whether the modulation changed from 1 to 0, was steady at 0 or 1, or changed from 0 to 1. The signal on a track only changes when a 1 bit was written, so both -1 and +1 in the input signal correspond to a 1, while 0 corresponds to a 0. Changes in the write signal only occur on a clock pulse, so a change in any of the seven tracks indicate that a clock pulse had occurred and there seven bits to add to the output stream. But what if all seven bits were 0? Then there would be no way to detect where the clock pulse was. A requirement on the input stream is that at least one bit of the seven written bits is always 1. When no 1s are seen for a long enough period of time, an inter-record gap is assumed and end of record processing is triggered.



# Tape modes

IBM had decades of experience with punched card data processing and used modified versions of punched card equipment for reading and writing punched cards as well as for printing. With magnetic tape, Whirlwind[^WhirlwindTape1951] and UNIVAC[^UNIVACUniservo] were also publicly experimenting, but no one had any real world experience and things that seem obvious today hadn't been thought of yet. Mistakes were made and then had to be worked around. Moving through the formats chronologically helps to make sense of some of the craziness.

## Before tape

When IBM began developing computers they had a well-established business of producing equipment for punched card data processing. A deck of cards served as a table in a database. Machines configured by plugboards could sort, collate and tabulate decks of cards. Modified versions of the punched card equipment could be used to copy the contents of a card to a virtual card in computer memory as well as punch new cards and print from virtual cards in computer memory.

## IBM 701 - Binary mode

The IBM 701[^IBM701] was the first IBM computer to use magnetic tape. The processor had a 36 bit word and was designed for *scientific computing*, i.e. calculating. To print, a program needed to use 24 words of memory, corresponding to 72 columns and 12 rows of a virtual punched card. The program would set 1 bits where holes would be punched in the card and then send the 24 words to the printer the same way a card reader would.

A `WRITE` instruction sent 36 bits, which were split into 6 groups of 6 bits in big-endian order. An odd parity bit was computed for each six bits and served as the high bit. By using odd parity, every seven bits would have at least one `1` bit. As long as `WRITE` instructions were executed in time for the next 6 bits to be written, they went into the same record. When a `WRITE` did not occur in time, the record ended with an inter-record gap and the tape stopped until additional writes occurred. A `WRITE EF` instruction wrote an end of file marker at the end of a file.



During reads, the seven bits are added to the stream by storing them in a register. If the register is not read before the next clock pulse is found in the signal, demodulation continues to the end of the record and the program is notified of the timeout. Whether the inter-record gap was reached by producing all the bits or a timeout happened, if a new read is initiated before the tape is stopped, reading continues, otherwise the tape stops.

Although the tape device write 7 bits at a time, the computer provides one word, or 36 bits, at a time. The words are written 6 bits at a time in big-endian order. Two output modes are available, *binary* and *BCD*. The output mode determines how the seven bits to be written are determined from the 6 bits. There is no overlap between the two modes and changing the value of any bit will result in a seven bit value from the other mode.

As with writing, reading a record is associated with a mode that determines how the seven bits are transformed into six bits. If the seven bits are not valid for the mode, an error flag is set that the program can check.

A tape contains exactly one file, which may consist of any number of records. An *end of file*, or *EOF*, can be added after the last record. At a low level, this is a record with one clock pulse and a special EOF marker value for the bits. Since all normal records will have a multiple of six clock pulses, an EOF can not be confused with a normal record. A later change added a separate marker to indicate the file was continued on the next tape so that file size was unlimited.



# Record modes

Since logic was big and expensive, encodings were kept simple. In binary mode, the 7th bit is the odd parity of the lower six bits, while in BCD mode bit 7 is the even parity value of the lower 6 bits. The mode is per-record, so all the bytes will have the same kind of parity unless a 1 bit error has occurred. We can look at the parity of the bytes in the first record:
```
HEX:    63         0A         50         65         71        ...
BINARY: 1 10 0011  0 00 1010  1 01 0000  1 10 0101  1 11 0001 ...
EVEN:   1          0          1          1          1
ODD:    0          1          0          0          0
```
It can be seen that even parity is being used, so this is BCD data. BCD is used for characters. In the binary mode, the low six bits are the actual six bits. With odd parity, every seven bit value will contain at least one 1. This is because if the low six bits are all 0, the parity will be 1, and if the low six bits are not all 0 at least one of them is a 1. With even parity, if the low six bits are 0 then the parity is 0 resulting in all seven bits being 0, which is forbidden. In the BCD encoding, a blank (space) is encoded as all 0s, but in BCD mode it is converted to `0x10`, or `0x50` with bit 7.

Making this transformation, we have
```
[23 0A 00 25 31 00 0A 01 01 0A 00 00 00 00 ...]
 L  0     N  A     0  1  1  0
```
The characters will be explained in the next section. This happens to be an identification record for the next record. The `NA` indicates it was a contribution from North American Aviation, a major contributor to 70x software.

For the second record, the value `0x40` is `1 00 0000`, which is odd parity, i.e. binary, so the six bits are simply `00 0000`. Six of these make one 36-bit word whose value is `0x000000000` or `0o000000000000` in octal.

# Hollerith and BCD

Before computers, there was a data processing industry based on processing punched cards with electromechanical equipment. Early computers used modified versions of this equipment for punched card input and output and printing. Computers were significantly faster and more expensive than the punched card equipment, so magnetic tape was developed for faster less bulky input and output. Although the IBM 701 and 704 computers and magnetic tape both used BCD encodings, magnetic tape could only be written using a computer or connecting a punched card reader to a tape drive and copying a deck of cards to tape. Anything not computed had to originate on punched cards. Cards could be directly punched from the computer, punched by transferring tape contents to cards, or manually produced using a *key punch*, a human-operated device with a keyboard.

IBM punched cards had 80 columns and 12 rows where holes could be punched.

![Blank Card](images/blank.png)

Rows `0` through `9` are labeled, while the top two rows, `12` (top) and `11` (above `0`) are not. Each column could hold one character.
The early punched card equipment just counted occurrences of holes at particular locations. Newer equipment was developed that could treat the position of a hole in a column as a value. Groups of columns in a deck of cards could be summed and printed.

Card reading equipment passed the card over sensors from bottom to top. Originally, when a hole was sensed in a column, a counter wheel would start moving and stop when row 0 was reached. Thus, a hole in column 6 would move the counter six positions. Later, plug boards were introduced so that as a hole was sensed in a row some row-specific action for that column could be taken.

![Digits Card](images/digits.png)

A deck of cards was like a table in a database. The simpler relational database operations, like joins, could be performed on decks of cards. If cards could also store alphabetic information, invoices, inventory lists, etc. could be printed. Today we would easily accommodate additional characters by treating the 12 rows as a 12 bit field. With electromechanical devices, positional systems were much simpler to implement. Rows `12`, `11` and `0` were used as a kind of four way shift when one of the rows `1` through `9` was punched. Otherwise, the column was still treated as a digit. These three rows specified the *zone*, which had one of four values, `12`, `11`, `0` or no zone punch. A lone `0` was treated as no zone.

![Alpha Card](images/alpha.png)


In the 1950s, multiple encodings were still in effect. Part of interpreting tape contents is determining which particular encoding was in use. At the time, programmers just had to just make mental substitutions between what the assembler thought the encodings were and how their equipment had been configured for the types of applications it would be running.

|       | 12| 11| 0 |   |
|:-----:|:-:|:-:|:-:|:-:|
| **0** | & | - |   | 0 |
| **1** | A | J | / | 1 |
| **2** | B | K | S | 2 |
| **3** | C | L | T | 3 |
| **4** | D | M | U | 4 |
| **5** | E | N | V | 5 |
| **6** | F | O | W | 6 |
| **7** | G | P | X | 7 |
| **8** | H | Q | Y | 8 |
| **9** | I | R | Z | 9 |
|**8-2**| ¢ | ! |   | : |
|**8-3**| . | $ | , | # |
|**8-4**| < | * | % | @ |
|**8-5**| ( | ) | _ | \ |
|**8-6**| + | ; | > | = |
|**8-7**|\| | ¬ | ? | " | 

Here the columns correspond to which zone is punched, `12`, `11`, `0` or no punch, and the rows correspond to the digit punches. For example `F` has zone `12` and digit `6`, while `#` has no zone punch and digits `8` and `3`.

In the BCD encoding, the top 2 bits specify the zone encoding, `11` for `12`, `10` for `11`, `01` for `0` and `00` for no zone punch, while the low four bits are the row.

|        | 11| 10| 01| 00|
|:------:|:-:|:-:|:-:|:-:|
|**0000**| & | - |   | 0 |
|**0001**| A | J | / | 1 |
|**0010**| B | K | S | 2 |
|**0011**| C | L | T | 3 |
|**0100**| D | M | U | 4 |
|**0101**| E | N | V | 5 |
|**0110**| F | O | W | 6 |
|**0111**| G | P | X | 7 |
|**1000**| H | Q | Y | 8 |
|**1001**| I | R | Z | 9 |
|**1010**| ¢ | ! |   | : |
|**1011**| . | $ | , | # |
|**1100**| < | * | % | @ |
|**1101**| ( | ) | _ | \ |
|**1110**| + | ; | > | = |
|**1111**|\| | ¬ | ? | " |

But then `0` is moved from `010000` to `0010101`.

As electronic calculating equipment was developed, various 4 bit digit encodings were developed in conjunction with their arithmetic implementations. IBM used binary, except that `0` was encoded as `1010` to avoid `0000` on tape. As a card was read, a counter could be started. For characters, two more bits were added for the zone. For no zone (i.e. just a single digit) the zone bits were `00`. For zone `12` the zone bits were `11`, for `11` the bits were `10` and for zone `0/10` they were `01`. When `8` was punched along with another digit, the 8 bit was set in the low four bits. A blank column was `010000`.


Prior to computers, punched cards were the dominant storage media for data processing. Punched card equipment used idea from looms that could sense holes in cards, and railroad conductors punching holes in railroad tickets to indicate the occurrence of various events. A hole in a punched card at a particular position indicated some property that was to be counted or collated, and special equipment incremented counters when a card passed by with a hole in the selected position, or cards were separated based on a hole in a position.

As technology improved, cards were changed to have 80 columns with ten row positions, called `0` through `9` for holes. Cards were read from top to bottom, and, as read, counter wheels turned until a hole was sensed, adding that value to a running total. The system was enhanced by adding two *zone* rows at the top, `12` and `11`. If a column only had one hole punched in `0` through `9` it encoded that digit, while if rows `12`, `11` or `0` and another digit were punched, the character was an uppercase letter or a symbol. A hole in row `8` could also be punched in some circumstances. The card encoding for characters was called *Hollerith*.

Each Hollerith character had a 12 bit encoding on the card, chosen to simplify the electromechanical devices that processed punched cards. But there were less than 64 characters, so only 6 bits were needed. The 6 bit encoding was called BCD.

It is simple to convert between Hollerith and BCD. The low 4 bits are 



In the data processing industry, a 12 bit encoding was used with 80 column punched cards that simplified electromechanical sorting, collating and tabulating operations. A keypunch, a combination of a keyboard, card punch and optional printing mechanism, was used to create the cards, which were then stored until needed.

In telegraphy, teleprinters electrically combined a keyboard and printing mechanism that could communicate with remote teleprinters using five or six bit encodings. Many teleprinters included a paper tape punch that could also be used for output and paper tape reader that could be used as an input. The keyboard could be used to encode messages on paper tapes which would be sent when a line was available to remote teleprinters. With teleprinters, the encoding was developed with the goal of minimizing line use and equipment wear.


 



# Machine-readable information

Today most information stored in a human-readable form is also machine-readable, but most machine-readable media cannot be read directly by humans and even most of the information on human-readable media passed through a computer, if not generated by a computer. Here we look at the steps along the way to today's written media. 

## Media to control machines

One of the earliest examples of machine-readable media are cylinders with pins. In Baghdad in the 9th century the Banu Musa brothers built a water-powered organ controlled by interchangeable cylinders with pins. As the cylinder rotated, the pins would push against parts that would allow specific notes to play.[^WikiMusicBox]

In 1725, Basile Bouchon, coincidentally the son of an organ maker, brought machine-readable media to weaving. In a loom, warp threads run the length of the cloth and weft threads are woven between the warp threads. Patterns are made by the weft thread going over and under groups of warp threads. Basile used a paper tape whose length corresponded to the warp and whose width was partitioned into columns, one per warp thread. At each step of weaving, the loom raised the warp threads whose columns had holes punched in the tape. A weft thread was sent across, going between the raised and lowered threads. The tape was shifted to the next position, causing a different set of warp threads to be raised, and the process repeated.[^WikiBouchon]

In 1804, Joseph Marie Jacquard improved on Bouchon's loom. It was difficult to correct a mistake on the paper tape or to repair the tape when it tore from wear. Jacquard made a tape by sewing together thicker cards which were less likeley to tear. If a mistake were made or a tear did occur, only one card had to be replaced.[^WikiJacquard]

Railroad personnel would punch holes in railroad tickets to indicate various information for humans. Herman Hollerith combined the idea of recording information with punched holes and using a machine to act upon the holes to tabulate the information. There had been some earlier suggestions to do the same thing, but they did not lead to the creation of a data processing industry and IBM and become synonymous with computers for a few decades.

## Machine-readable text

Émile Baudot developed a five bit character code in 1870 for use with telegraphs in Europe that evolved into ITA2, standardized in 1930. It was actually two character codes, where each of the two codes had a switching character that switched to the other code. This code was used by teletype machines. Teletypes combined a keyboard, printing mechanism, paper tape punch and reader and wire connection. The keyboard, paper tape reader and wire can all serve as inputs, while the paper tape punch, wire and printer can all serve as outputs. A typical mode of use was to type messages on the keyboard, printing them and punching a paper tape at the same time. When a communications line was available, the paper tapes would be rapidly read and sent over the line to another teletype, which could either punch tapes for later off-line printing, or print directly.  A group of teletypes could also be configured so that whatever was typed on any of the keyboards printed on all the teletypes.




Computers used punched cards, punched paper tape and magnetic tape for input, output and data storage. Punched paper tape was the oldest machine-readable storage medium, used for controlling looms. In weaving, paper tape was replaced by tapes made by sewing together cards, which were stronger and made it easier to make corrections and repairs. Paper tape was again used in telegraphy as a way to better utilize the network. Messages could be written to tape by partitioning the tape into channels, with a line of punches in the channels perpendicular to the tape length encoding a character. The tape could be used to mechanically send the message more rapidly than a human could transmit Morse code. Some combination of the cards used to make the tapes for looms and railroad conductors punching holes in train tickets led to using punched cards to encode data which could be mechanically tabulated. Later they were refactored into representing a group of fields each made up of one or more characters, with each character encoded by a set of holes punched in a column.

Encodings were chosen for practical reasons, such as minimizing the number of tape punches needed to encode the common letters to make the punches last longer, or using character representations on cards that simplified data processing operations such as summing fields, sorting and collating. The punched paper tape and card equipment was adapted to be used with the computers. Teletype machines could punch paper tape from a keyboard and then send the information over a line or send it directly, as well as receive data and punch a paper tape and/or print the received data. By connecting the line to a computer, the computer could receive keyboard input and print output, provided it used the teletype encoding. Even the early computers were much faster than paper equipment, so magnetic tape was developed as a faster means to get data to/from the computer and equipment was developed to transfer between paper storage and magnetic storage without tying up the computer.


Various forms of tape have been used to store machine-readable information for about 200 years, starting Bouchon's 1725 loom, which used holes punched in tape to control a loom. Punching mistakes were difficult to correct and tears were hard to repair. Jacquard solved both problems by sewing cards together. Correcting a mistake or a tear only required making a new card, not an entire new tape. Hollerith adapted the tape of cards controlling needles to individual cards where each position on the card was associated with a possible selection of a value and a hole in the position indicating the value was selected.

A *line* on a tape is the set of tape positions read/written at some particular instant as the tape travels past the sensors or writing mechanism. In older equipment, a line is perpendicular to the long edges of the tape. Modern high-density magnetic tape uses diagonal lines and an off-axis rotating mechanism to pack more information into the same region without the need for densely packed sensors.

Piano rolls and music boxes assign discrete positions on a line to notes and have a continuous time axis. When a hole passes over the sensor for a note, the note is played. Information uses a discrete time axis.


# Encodings

When you send a message to someone you expect them to be able to read it, even if they have a different kind of computer or phone. You can copy a text message into an email. You can print it without being concerned about what kind of printer you use. The only time you need to have any concern is if you save it into a text file, which is the last vestige of a time not so long ago when things were much different.

The earliest telegraphy used humans for encoding and decoding. To make more effective use of the wires, encodings were developed that could be read and produced by machines.

 a new fixed-length encoding was developed using holes punched in paper tape. Humans could put the messages on paper tape and then machines could read the tapes and transmit their contents to other machines that could produce new paper tapes that could be decoded.



# Using magnetic tape

An LTO 9 tape cartridge is about 4"x4"x.85", holds 18TB on a 3400 foot half inch tape, which averages about 440MB/inch, and has a data transfer rate of 400MB/s. It is mainly used for archiving and backup. Cartridges cost about $140, tape drives start in the low $4,000s. With much faster 18TB disk drives costing at about twice the price of a cartridge and not requiring a separate drive, few, if any, programs use tape today, other than some backup and archival software.

For a number of reasons, magnetic tape was very important in the initial decades of computing:
- Additional storage could be added at the cost of a reel of tape.
- Data transfer between the CPU and tape was much faster than between the CPU and I/O devices, but data transfer was simple enough that the slower I/O devices could directly read/write tape off-line, freeing the CPU for other work.
- Unlike drum and early disk storage, tape reels for one program could be removed and replaced with other tape reels for other programs.
- In the days before networks, tape reels could be used on different computers, and, to some extent, on different kinds of computers. Even in the earlier days of networks a joke used to be, "What's the fastest way to transfer data? A truck full of tapes."

## Physical medium

Open reel computer tape used reels of tape that were up to 10.5" in diameter. The tape was coated with a powder that could be magnetized and was .5" wide and wound around the hub of its reel in lengths of 1200' or 2400'. When used, the outer end of the tape was threaded through the drive past a line of tape heads and onto a second reel. Tape could be moved rapidly past the heads but could also be stopped and started quickly since vacuum columns maintains slack between the light section of the tape that was stopped and started and the heavier reels of tape that held the bulk of the tape. Reflective markers near the beginning and end of the tape kept the tape from moving past a point where it would come free of the reel. The beginning marker also serves to position the tape where data operations will begin after loading or rewinding the tape.

Other than the reflective markers that denote the two ends of the tape, a tape starts with no physical location information. Tape can expand and contract with changes in temperature and humidity and stretch with use, and different drives may move the tape at slightly different speeds, so all location information must be derived from the signals from the heads for each track as the tape passes them. During writing, a tape head will magnetize the region of tape passing it in a way that depends on the strength and direction of the current. During reading, a tape head will produce electrical current proportional to the rate of change of the magnetic field as it passes the head. Thus, a section of tape with a constant magnetic field, no matter what the value of the field, will produce no signal during a read.

Since writes affect a small region of tape, they must be spaced far enough apart on the tape so that they do not interfere with each other. If you could see the magnetism on the tape you would see periodic locations where the magnetic field changed. These are called *lines*. Since there are no physical cues for finding the lines, the combination of signals in the tracks during a read needs to be able to identify where a line has occurred. Particularly in early computers, a line was smaller than the smallest number of bits that could be written/read in a single operation. A *frame* was a group of lines that made up this smallest quantity of bits. Early computers were not capable of sustained reading or writing a tape. Instead, the tape contents were organized as blocks of data called *records* separated by enough tape without lines, *called an inter-record gap*, that the tape could be stopped and then restarted.

A tape application did not always know how many records would be on the tape. In one tape format, a special grouping of lines smaller than a frame was used as an *end of file*, or EOF, to indicate that there was no more data on the tape. A tape held exactly one file. Later, this scheme was adapted so that a file could span multiple tapes. All but the last tape indicated there was another tape, and the last tape had the EOF.

The `tar` file is a remnant of the days when tape was used for transferring files. `tar` stands for *tape archive* and creates a file containing the names and contents of many files into a single file that can be put on tape, or creates the files with the names and contents in such a file.

## Whirlind

Whirlwind adapted a specially designed Raytheon 6 track half inch magnetic tape drive. While others found ways to make vacuum tubes last longer when used in a computer, the Whirlwind group determined what was causing the vacuum tubes to fail and found ways to make better vacuum tubes. Likewise, the group analyzed what caused data loss on tape and determined dust and uneven coatings of magnetic material on the tape were the problem. Both caused small regions of the tape to not be magnetized. The regions were smaller than the size of two adjacent tracks, so tracks were paired such that every pair had two other tracks between the pair, i.e. 0-3, 1-4 and 2-5. Tape drop-outs were rare and two drop-outs in the same line were extremely rare, so if either head in the pair detected a change in the magnetic field it was assumed to be real. For this to work, there needed to be no magnetic field between lines, since, otherwise, a drop-out would be seen as a change in the field from some value to 0. Lines were marked using the pair 0-3, which was written for every line. Two bits of data were stored using the pairs 1-3 and 2-5. A frame was 8 lines, or 16 bits, the word size. Records had one frame. Unlike may later tape formats, records not at the end of the tape file could be modified in place.[^WhirlwindTape1951]

Block mark is first line of a block, recorded automatically.
6.3.1 1000' x .5" 100 lines/inch
6.3.1.2 6 digit binary character code recorded serially, 4 lines. First line is just index mark. 4 index pulses (0 for rest?) stops printing.

The 0-3 pair was used as a timing marker and the other two pairs were used for data. Each three bit write was called a line. The density for lines was about 100 lines per inch, which is about 25 bytes/inch. Tape traveled at 30 inches per second, giving a transfer rate of 750B/s. Tape could be read/written in either direction but needed to be read in the same direction it was written. There are 12 msec of blank tape at the start of a record, followed by a block marker and then a sequence of 16 bit words. Words in a block could be read or written individually or as a block transfer, but the reads or writes needed to be able to keep up with the speed of the tape[^WhirlwindTrainingMaterial].

## UNIVAC UNISERVO

A UNISERVO tape had 7 tracks, 6 for data and one for odd parity. Data was stored as a sequence of 60 word records, where each word was 72 bits. The UNIVAC I supported up to 9 UNISERVO tape drives and had a 60 word buffer for use with tape.[^UNIVACUniservo]
Tape instructions were:
- Read the next block on a tape into the buffer. The tape is positioned at the end of the block.
- Read the previous block on a tape into the buffer. The tape is positioned at the beginning of the block.
- Transfer the entire tape buffer to a block of memory and optionally start a forward or backward read. There is an alignment requirement for the starting memory address.
- Fill the entire tape buffer from a block of memory.
- Write the buffer as the next block on a tape, making any data that followed the initial tape position inaccessible.
- Write the buffer at low density as the next block on a tape. Low density was used with off-line tape equipment.
- Rewind the tape, optionally unmounting the tape. If unmounted, the tape cannot be accessed without physically remounting it.[^UNIVACFactronic] [^UNIVACIBasicProgramming] [^UNIVACIIBasicProgramming]

## Univac

In 1951 UNIVAC introduced the first commercial tape system, UNISERVO. The metal tape was 1200 feet by a half inch on an 8" reel that together weighed about 25 pounds. A tape could hold about 1.5MB, which is about 16 bytes per inch. The transfer rate was about 9.6KB/s. Today, one 4"x4"x.85" LTO 9 cartridge costs about $140, holds 18TB on a 3400 foot half inch tape, which averages about 440MB/inch. Data is transferred at 400MB/s.

Tape was introduced as an input/output medium with throughput closer to the speeds of the computers at the time than equipment based on punched cards and paper tape. Cards could be transferred to tape off-line and tapes could be printed off-line while the computer was kept busy with other tasks. Tape was also used to transfer data between sites, and, once standardized, between different types of computers. For many years, tape also served as a cheaper extension to the limited expensive memory found on a computer. In the first several decades of computing, knowing how to use tape was a necessary part of programming. In media, computers were depicted with flashing lights, punched cards, printers and spinning tapes. Today tape is mainly used for backing up and archiving data and most current programmers have never even seen tape, let alone know how it was used.

Although tape is no longer used in most programs, it still influences programming. Files are stored on random access devices but are treated as though they are a linear sequence of bytes. When random access devices such as drums and disks were first introduced, they did not have file systems. Programs accessed fixed-size blocks of data by its physical location on the disk or drum and performant programs took advantage of the geometry to minimize latency. Tape had files. It was only later that file systems were added so disks and drums could conveniently cache frequently used tape files. Originally operating systems supported a variety of file types. In Multics, a file was used by mapping its contents to memory. It was UNIX that treated everything as linear sequences of bytes, just like tape.

## IBM

Before magnetic tape was developed for data storage, punched cards were used to store most machine-readable data. Key punches could be used to record information on cards. Sorting equipment could organize a deck of cards into multiple decks based on how a particular column on a card was punched. Collators could merge sorted decks of cards based on groups of columns. Tabulators could print values from cards while performing simple calculations. Early computers used modified versions of card printing and reading equipment for input/output, but reading and writing cards was much slower than the computer. Tape was much faster and the same equipment could be used to transfer the data on a deck of cards off-line while the computer was performing other tasks. Likewise, the computer could quickly write results to tape that could be used for off-line printing while the computer was working on other tasks.

Memory was very expensive. Computers had memory, I/O devices did not. Data was written to tape in packets called records, with a section of erased tape in between the records. While writing a record, the computer had to be sending data to the tape drive as fast as it was writing. When reading, the computer had to accept data as fast as the tape drive was producing it. Between records the tape could be stopped to let the computer perform processing, but stopping and starting the tape was slow compared to the computer, and the inter-record gaps could take a lot of space on the tape. Programs would choose a record size that was not too large because the computer's memory was limited, but large enough that tape could be used efficiently.

Tape is linear, so the only way to get from point A to point B is to pass through all the intervening points. At normal tape reading speed this would be a slow way to skip to another part of the tape. The inter-record gap was chosen long enough that it could be detected while running the tape at a high speed, and then the tape could be stopped ready to read the next record. A special one character marker record was used to denote an end of file. The end of file marker could also be detected at high speed, so that the tape drive could be told to skip to the next file. A physical marker near the beginning of the tape was used to end a high speed rewind to the first record on the tape.



# Storing data

Data storage is information transfer where the sink is somewhat later than the source. Memory is the oldest form and predates humans. Animals store information when they mark territory. Cave paintings transfer stories from humans long ago to later humans. Written language encodes even more information.

Information can also be transferred from machines. An hourglass knocked over by a door stores the time when the door was opened. Storing information for a machine to use seems to be a more recent invention. It is easy to make mistakes weaving complicated patterns on looms. In 1725, Bouchon had the idea of using holes in a spool of paper to indicate which vertical threads should be raised for each horizontal thread[^WikiBouchon]. Unfortunately, it was difficult to make corrections to mistakes made when punching the holes, and repairs to the paper when tears occurred were also difficult. Jacquard used thicker more durable cards tied together, so repairs and corrections could be made by replacing a card[^WikiPunchedTape].

Computer tape is still used. One 4"x4"x.85" LTO 9 cartridge holds 18TB, transfers data at 400MB/s and costs about $140. In 1951, the first commercial UNIVAC UNISERVO tapes used an 8" reel of .5" metal that weighed 25 pounds and could hold about 1.5MB, with a transfer rate of about 9.6KB/s. Today tape is primarily used for backup with cloud storage a strong competitor. Few people today have even seen a tape, let alone have any idea how to use one. The same was true in 1951, but they were developed by necessity. Knuth has chapters of tape algorithms. Once disks could much more conveniently handle the storage needs, tape was primarily used for data transfer, archival and backup, all of which only required running commands. As networks matured, all that was left was archival and backup. But remnants from the days when tape was central remain, such as the `tar` file format, where "`tar`" stands for "`tape archive`."

Text.

## Machine-readable data

Human-readable data storage literally goes back to the dawn of recorded history. One of the earliest forms of machine-readable data storage was Bouchon's 1725 loom.[^WikiBouchon] A paper tape wide enough to have enough holes for every vertical thread allowed needles to pass through the holds that would cause the threads to be lifted so that when the horizontal thread passed through it made the correct pattern, and the tape would move to the next row of holes. Paper thin enough to bend through the mechanism had a tendency to tear and was difficult to repair. Jacquard found that by tying together stiffer cards they acted like a hinge at the boundary, tearing was reduced and repairs were simpler.[^WikiPunchedTape] In 1842 punched holes in a paper tape, a piano roll, were used to drive the keys on a player piano. Since the holes could be mechanically sensed, they could be mass-produced with a machine that used the holes in a master to control the punching of holes in a new piano roll.[^WikiPlayerPiano] A variant was the music box, which used a disc with raised pins to pluck tuned teeth as it rotated by them.[^WikiMusicBox]








In the early days of information processing, data was stored on punched cards. In theory, one IBM card can hold up to 120 bytes, while in practice cards that made use of all possible bit combinations would tend to fall apart and 80 six bit characters was the usual limit. It is no coincidence that 80 is often the default page width. Memory was very expensive. During processing, decks of cards were put in hoppers and cards physically moved through equipment that sorted, collated, summarized, printed and produced new cards. This equipment was adapted so that it could be used as input and output devices for early computers.

Punched paper tape actually preceded punched cards with Bouchon's 1725 loom[^WikiBouchon]. The paper was difficult to make and it would tear. Jacquard switched to threaded together cards for his looms[^WikiPunchedTape]. Piano rolls for player pianos were another early use of paper tape. Teletypes used a narrower tape with five holes per character.




Although software engineers think of computers as binary devices manipulating 0s and 1s, the digital world is implemented by hardware engineers in the non-digital real world. The values 0 and 1 are each implemented with a range of some physical value such as a voltage or current. Noise, unpredictable variations in the values, can change a value that is in the 0 range into one that is in the 1 range, and vice versa. Small variations are more likely than large variations, so signals are amplified to keep the 0s and 1s as far apart as practical to reduce the chance of a change in value.

Data storage is accomplished by making use of devices with two stable states. Static RAM uses a device that only allows current to flow in one of two ways until disrupted. Dynamic RAM stores electrons in an insulated region. In order to tell if a 0 or a 1 is stored, the device opens a hole in the insulation. If electrons flow out, a 1 had been stored. Once tested, the device needs to be refilled or re-emptied with electrons. Since insulators are not perfect, the device must also be periodically refreshed by reading and then rewriting.

Data can also be stored in a magnetic material. At each point in space, a magnetic field has a direction and intensity. If a magnetic field is made strong enough for long enough near a magnetic material, the magnetic field in the material will shift to the same direction as the external magnetic field and remain in that direction until changed again by a stronger magnetic field. Depending on the intensity of the external field, the intensity of the magnet's field may also be set, up to a limit dependent on the material.

A changing electric field will create a magnetic field. If an electron passes through a region, the electric field in the region changes, creating a small magnetic field. Current flowing through a wire moves many electrons, creating a larger magnetic field. If current passes through a coil of wire, each loop adds to the magnetic field. Reversing the current reverses the field, so current can be used to make magnetic fields to change the direction of the field of a magnet. If we move the coil over a magnetic material, or move the material past the coil, we can magnetize different parts of the material in each of two directions, storing data.

Stored data isn't of much use if it can't be read back. To read it back we need to know find the correction region and determine the direction and possibly intensity of the magnetic field. The easiest way to do this is with a changing magnetic field. Just as a changing electric field creates a magnetic field, a changing magnetic field creates an electric field. If this changing magnetic field passes through a coil, each loop of the coil adds to the electric field, making it easier to detect. If the magnetic field is not changing, there will be no electric field.

If we are able to accurately position the magnetic material at a point with a known value, we can track changes in the field as we track a path to the region we are interested in. This tracking is called integration, and there are integrator circuits.




Definitely [^Ledley1960] pg 737.

See [^IBMTapeUnits].

Although software engineers think of digital computers as digital devices manipulating 0s and 1s, we live in an analog world and those 0s and 1s are represented and manipulated in the non-digital physical world and subject to its rules. During processing, 0 and 1 are represented by two extremes of voltage or current and anything close enough to the extreme will be treated as though it were the extreme.

Magnets may seem to have two natural extremes, north and south, but these are no more extremes than positive and negative are extremes for numbers. Magnets produce a magnetic field. Each point in the field has a magnitude and a direction in three dimensions. We call the direction north. The magnetic field comes from a changing electric field, caused by an alignment of the movement of the electrons in the magnet's atoms, and that magnetic field keeps the electron movements aligned.

Another way to create a magnetic field is by moving electrons through a wire. As each electron moves past a point it causes a change in the electric field at that point, creating a magnetic field. By coiling the wire, the magnetic fields from each loop add to each other making a stronger field perpendicular to the coil. Reversing the current reverses the magnetic field. With a strong enough external magnetic field, the alignment of the electron movements in a magnet can be changed, resulting in a new stable state that remains after the external field is removed. But if the external field is not strong enough, the magnet's field will realign its electron movements to the original orientation.





Traditional magnetic tape for computers are a non-magnetic base material coated with magnetic iron oxide, cut into long strips half an inch wide and wound on a reel. The tape moves over a group of heads spaced across the half inch strip. The section of tape that passes a particular head is called a track or channel. When reading, a head can only detect if there is a change in the magnetism of the tape. When writing, a head leave the section of tape passing by it magnetized as north or south. If a tape were magnetized only north or only south, there would be no change in the magnetic field and nothing would be read.


## Univac Uniservo

The [UNIVAC UNISERVO](https://www.computer.org/csdl/proceedings-article/afips/1952/50410047/12OmNzxgHya) is the first computer tape drive. It can be seen in [Computer History: The Amazing Design of the UNIVAC's UNISERVO METAL TAPE DRIVES 1951](https://www.youtube.com/watch?v=-KuoZ6cades&t=8s). The tapes are made of metal coated with iron oxide. A reel of tape weighs about 25 pounds.

The UNISERVO uses phase encoding, also called Manchester encoding. When writing, a head is magnetizing the tape in a particular direction, either north or south. It doesn't matter which, so we can assume south. When it is time to write a bit,
- For a 0, south continues to be written briefly, then north briefly, then back to south,
- For a 1, north is written briefly, then back to south.
In both cases, there is a brief switch to north and then a return to south. During a read, the switch to north will be detected. If it comes at the position on the tape where the bit write occurred, then a 1 is read, while if it comes a brief time later, a 0 is read.

The read circuitry needs to know the position on the tape where the write occurred so it can determine whether there that was where the change to north happened, or if the change was delayed. The UNISERVO used a timing track to indicate these positions. Whenever it was time for a head to write a bit, the head for the timing track wrote a 1.

```
            _      _
TIMING: ___| |____| |____

             _     _
DATA:   ____| |___| |____

           0       1
```

The UNISERVO has eight tracks
- The timing track,
- An parity track,
- Six data tracks. The UNIVAC used six bit characters.
The parity for a character is odd parity for the character, or even for the entire eight bits being written since the timing track is always 1.

Data was written in packets (called records) of 720 characters, with erased space (no change in magnetism) between the records. A record used 5.6 inches of tapes, and the records were separated by 2.4 inches of blank tape.

## IBM improvements

The timing track was not really necessary since with odd parity there will be at least one 1 in every character, so whenever any track indicated a change in polarity, all tracks that changed polarity at the same time would be 1s and all that had a slight delay would be 0s.

NRZI allowed data to be denser.

# Data organization on magnetic tape

IBM 70xx tapes store data in variable-sized packets called *records*. The data in a record is a dense sequence of 6 bit values, called *characters*, with one parity bit for each character. The character encoding, described below, ensures that every character, including repeats, results in a detectable change on the tape. The last character of the record is immediately followed by a longitudinal checksum value that makes the number of 1 bits in each position in the record even, and then about \\(\frac{3}{4}\\) inches of tape with no detectable signal on the tape.  A special *end of file* record has a single 001111 character followed by the longitudinal checksum, and \\(3\frac{3}{4}\\) inches of tape with no signal.

For historic reasons, the CPU used a different encoding for text than what was used on tape, so it was necessary to distinguish binary data from text, so binary data used odd parity and text used even parity. Technically a single bit error in a record can be corrected by using the character parity to determine the character position of the error and the longitudinal parity to determine the bit position within the character.

# Storing data on magnetic tape

Tape was made by coating large rolls of a flexible material, originally cellulose acetate and later mylar, with iron oxide. After coating, the rolls were sliced into half inch ribbons up to 2400 feet in length. The early tapes could hold up to 1.6 million 6 bit characters (1.2MB). Although barely worth mentioning today when an LTO-9 tape cartridge holds 18TB, in terms of the primary data storage of the time, this is equivalent to 20,000 punched cards, which would be a stack about 12 feet high.

An IBM 704 could have up to 10 IBM 727 tape drives. Each tape drive weighed 911 pounds, was 29 1/4" deep, 28 1/2" wide and 69" tall, and averaged 1148 watts. The follow-on 729 can be seen in [Debugging the 1959 IBM 729 Vacuum Column Tape Drive at the Computer History Museum](https://www.youtube.com/watch?v=7Lh4CMz_Z6M).

Seven tape heads were spaced evenly across the 1/2" width of the tape.

The tape drive had seven tape heads spaced evenly across the 1/2" width of the tape that passed over the heads from left to right. Each head was a torus of a material that conducted magnetism but did not retain magnetism. Coils wrapped around the torus would induce a magnetic field. A small air gap where the tape passed over the head forced the field to spread out from the ring into the tape so that the tape would be magnetized as it passed over the head. Reversing the direction of the current reversed the direction of the field. While writing, the current in one direction or the other was always applied to the coils.

```
<--- Tape Movement ---<
>-----><-----<>------>
```

If `>` were used for 0 and `<` for 1 then the data could be read back by sensing the orientation of the field on the tape at the locations corresponding to the writes, marked by `|`:
```
--> Tape Movement -->
Field: >>>>><<<<>>>>>>>>><<<<<<
Write:  |  |  |  |  |  |  |  |
        0  0  1  0  0  0  1  1
```
The value written would be `11000100`, reversed from the figure since the tape is moving left to right.

Using this scheme, the data could be read from the tape by sensing the field orientation at the points where the writes had occurred. Unfortunately, there were two problems with this approach:
- Finding the places where the orientation should be measured. Tape could stretch, clock frequencies could vary slightly between drives or even the same drive.
- At the time, there wasn't a way to sense the orientation of the field on a moving tape.

There was a way to sense a change in the orientation of the magnetic field since this would induce a current whose direction depended on whether the change was `>` to `<` or `<` to `>`. These changes could be used to track the orientation, but there was still the problem of determining where the writes had occurred.

IBM came up with a method called NRZI, *non return to zero inverted*. During writes, whenever a 1 was written the current was reversed, reversing the field. Thus during reads, whenever there was a current, in either direction, there was a 1 at that location. Recall that there are seven heads, so there are seven tracks of data written to the tape. The data is written 6 bits at a time and the seventh track is a parity track. The tracks are called `C`, `B`, `A`, `8`, `4`, `2` and `1`.

When writing binary data, tracks `B` through `1` hold the six bits of data and `C` is chosen so that the number of bits is odd (odd parity). If the octal values 01, 02, 14, 27 were written the tracks would look like:
```
--> Tape Movement -->
C >>><<<<<>>>>>>>>>>>>
B >>>>>>>>>>>>>>>>>>>>
A <<<>>>>>>>>>>>>>>>>>
8 <<<<<<<<>>>>>>>>>>>>
4 >>><<<<<>>>>>>>>>>>>
2 <<<>>>>>>>>>>>>>>>>>
1 >>><<<<<<<<<<<<<<<>>
    |    |    |    | 
   27   14   02   01
```
With odd parity, every value will cause the change in at least one track's orientation. To see this, if there were no change in orientation `B` through `1` would have to all be 0 which would force `C` to be 1. This means that whenever any of the tracks change orientation the head is at a write point.


## Magnetism and current

Magnetic storage used two properties of electricity and magnetism:
 - Current flowing in one direction produces a magnetic field perpendicular to the current. Reversing the current flow reverses the direction of the magnetic field.  A coil of wire stacks the current flow, increasing the magnetic field strength.
 - A changing magnetic field causes a change in the current in a wire in the field. Again, a coil stacks the magnetic field resulting in greater current changes.

The tape head contained seven small coils that were evenly spaced across the half inch tape. If current flowed one direction, it would magnetize the tape as north, and, flowing the other direction, south. A simple way to record data would be to record seven bits at a time, using one polarization for 0 and the other for 1. To read the tape back, you would just need to move the tape past sensors that read the polarization values at each location where the bits were written. There are several problems with that approach:
 - Reliably determining where on the tape each 7 bit value is located. One approach, used on some drums and disks, would be to have a timing track.
 - Sensing the polarization. At the time, the easiest way to sense polarization was to try to change it to a specific polarization and sense whether the polarization changed since a current would be induced by a change. But that would make reads destructive, so the tape would need to be remagnetized.

One way to solve these two problems would be to put a demagnetized gap between each magnetized seven bits. This can be done by rapidly switching current direction between the writes, leaving a mixture of north and south that cancel out. Then during reading as a coil passes from neutral to north or south a current is induced and the direction of the current can be used to determine whether the value is a 0 or a 1, solving both problems at the expense of halving the bit density because of the neutral regions.

IBM came up with a method called NRZI, *non return to zero inverted*. In NRZI, the write coils are always writing. When it is time to write the next seven bits, the bits that are one reverse the current direction. During reads, as long as the polarity is constant, no current is induced, resulting in a 0 value. When the polarity reverses, a positive or negative current is briefly induced, with either being treated as a 1. As long as at least one of the seven bits written is a 1, any 1 pulse means a seven bit value has been read.

Even though there were seven tracks of bits, only six were data. The seventh was parity, either odd or even. IBM used odd parity when recording binary data. With odd parity, the parity bit is set to a value that makes an odd number of 1 bits in the seven bits. There is no way to have seven zero bits with odd parity. Character data used even parity. With even parity, the parity bit is chosen to make the number of 1 bits even. If some character were encoded as 0, the six bits would be 0 and the parity bit would then be 0 and no value would be detected during a read. The space character's encoding was 0, but there were less than 64 characters so when a space was seen it was remapped to an unused value, preventing a 0 from being written.

A tape held far more data than could fit in the memory of a computer. A way was needed a way to break the data up into packets called records in such a way that it was easy to detect where the records ended. When writing data, the computer needed to supply the data for a record as fast as it was being written. When the data for a record was complete, the polarization in all the heads was set to the same value resulting in a final seven bit value that was a checksum of all the values in the record. The writes would continue for 3 3/4 inches and then the next record would start being written if there was more data, or the tape would stop until more data was available. These inter-record gaps were easily detected during fast tape movement since all the read heads would sense 0.

## Reference

- [Reference Manual: IBM Magnetic Tape Units (1961)](https://bitsavers.org/pdf/ibm/magtape/A22-6589-1_magTapeReference_Jun62.pdf)
- [727 magnetic tape unit and tester (1956)](https://bitsavers.org/pdf/ibm/magtape/727/22-6681-0_IBM_727_Magnetic_Tape_Unit_and_Tester_CE_MOI_1956.pdf)
- [IBM 729 files](https://bitsavers.org/pdf/ibm/magtape/729/)

[^IBM701]: [Principals of Operation Type 701 and associated equipment](https://bitsavers.org/pdf/ibm/701/24-6042-1_701_PrincOps.pdf)

[^IBM702]: [IBM Electronic Data-Processing Machines Type 702](https://bitsavers.org/pdf/ibm/702/22-6173-1_702prelim_Feb56.pdf)

[^IBM704]: [IBM 704 electronic data-processing machine manual of operation](https://bitsavers.org/pdf/ibm/704/24-6661-2_704_Manual_1955.pdf)

[^IBMTapeUnits]: [Reference Manual: IBM Magnetic Tape Units (1961)](https://bitsavers.org/pdf/ibm/magtape/A22-6589-1_magTapeReference_Jun62.pdf).

[^Ledley1960]: Ledley, Robert Steven, *Digital Computer and Control Engineering*, McGraw-Hill Book Company, Inc., 1960.

[^PaulPierce]: [Paul Pierce's Computer Collection](https://piercefuller.com/collect/index.html).

[^UNIVACFactronic]: [Programming for the UNIVAC FACTRONIC system](https://bitsavers.org/pdf/univac/univac1/UNIVAC_Programming_Jan53.pdf)

[^UNIVACIBasicProgramming]: [Basic Programming UNIVAC I](https://bitsavers.org/pdf/univac/univac1/UNIVAC1_Programming_1959.pdf)

[^UNIVACIIBasicProgramming]: [UNIVAC II Basic Programming](https://bitsavers.org/pdf/univac/univac2/UnivacII_Programming_1957.pdf)

[^UNIVACUniservo]: [The Uniservo - Tape Reader and Recorder](https://www.computer.org/csdl/proceedings-article/afips/1952/50410047/12OmNzxgHya)

[^WhirlwindTape1951]: [Summary Report No. 28, Fourth quarter 1951](https://bitsavers.org/pdf/mit/whirlwind/Whirlwind_Tape_1951.pdf).

[^WhirlwindTrainingMaterial]: [Whirlwind Training Program Material](https://bitsavers.org/pdf/mit/whirlwind/Whirlwind_Training_Program_Material.pdf).

[^WikiIngenious]: [Book of Ingenious Devices: Mechanical musical machines](https://en.wikipedia.org/wiki/Book_of_Ingenious_Devices#Mechanical_musical_machines)

[^WikiBouchon]: [Basile Bouchon](https://en.wikipedia.org/wiki/Basile_Bouchon)

[^WikiBouchonLoom]: [Bouchon Loom](https://commons.wikimedia.org/wiki/File:Basile_Bouchon_1725_loom.jpg)

[^WikiHollerith]: [Herman Hollerith](https://en.wikipedia.org/wiki/Herman_Hollerith)

[^WikiJacquard]: [Jacquard machine](https://en.wikipedia.org/wiki/Jacquard_machine)

[^WikiMusicBox]: [Music Box](https://en.wikipedia.org/wiki/Music_box)

[^WikiPlayerPiano]: [Player Piano](https://en.wikipedia.org/wiki/Player_piano)

[^WikiPlugboard]: [Plugboard](https://en.wikipedia.org/wiki/Plugboard)

[^WikiPunchedCard]: [Punched Card](https://en.wikipedia.org/wiki/Punched_card)

[^WikiPunchedTape]: [Punched Tape](https://en.wikipedia.org/wiki/Punched_tape)
