---
title: "Reading IBM 70xx tapes from Paul Pierce's Computer Collection"
date: 2024-09-19T20:53:48-07:00
draft: true
---
# IBM 70xx tapes from Paul Pierce's Computer Collection

During the first several decades of computing, reels of magnetic tape were the primary means of bulk storage and data transfer. Old pictures and movies invariably depict computers as blinking lights and/or tape drives with reels of tape turning and stopping. Most of those tapes are now long gone, but [Paul Pierce's Computer Collection](https://piercefuller.com/collect/index.html) contains the raw data from a number of tapes. Here we describe how to understand the raw data obtained from early IBM computer tapes on Paul Pierce's site.

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