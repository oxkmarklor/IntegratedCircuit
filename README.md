While studying electronics, and more specifically the components of a computer's microarchitecture—such as processors and different types of memory, for example—I realized that there was 
a real bridge between hardware and software.

The purpose of this document is to explain how a circuit I designed works after becoming interested in arithmetic calculation circuits—floating-point units—on floating-point numbers.  

I am not a specialist, and it is possible that some elements of my circuit are not optimal, whether in terms of complexity, architecture, or proximity of circuits, which could lead to 
constraints due to certain physical limitations.
There is no order of magnitude in the diagram.

The document is divided into four phases:
  i - a brief overview of the IEEE-754 standard
  ii - a second phase contextualizing the problem that the circuit aims to solve
  iii - a third phase explaining how the circuit works
  iiii - a fourth phase with the small mathematical demonstration that was used to conceptualize the circuit



  i - Overview of the IEEE-754 standard

To understand what I am going to talk about in this document, it will be necessary to have a few concepts in mind, such as the IEEE-754 standard, which uses a modified version of the binary scientific notation of a number.

IEEE is a consortium of professionals in the field of electronics. 
Its role is to propose standards that are widely respected by major companies in the small world of electronic engineering.

The IEEE-754 standard is one such standard.
It defines the unique way of representing a floating-point number in a computer, regardless of the architecture and microarchitecture of the computer.
The standard also deals with certain ambiguous situations, such as the management of calculations with NANs, which we will not discuss here.

Basically, IEEE-754 is based on the binary scientific notation of a number.
Let's take the number 3 in decimal form, which is encoded as 0b11 in unsigned binary. 
Please note that the prefix 0b simply means that the number that follows is binary, nothing more.
Here is the binary scientific notation for the number:

+ 1.1 * 2^1

There are many sources where you can learn about the IEEE-754 standard and scientific notation in general, so I will just give a quick reminder.
In the binary scientific notation of the number 3 above, we have three elements:
- the sign (+ or -)
- the multiplicand, which is the power N to which the binary base (2) is raised
- the significand (or mantissa) which contains a digit from the interval [1;2 [ as well as all real numbers between these limits

The standard explicitly formalizes the encoding of several floating point numbers of different sizes, the main ones being 32-bit (single precision) and 64-bit (double precision) floating point
numbers.
Between single and double precision floats, there are half precision floats that are encoded on 16 bits. 
These are the floats we will be interested in because the circuit only supports half precision floats.

Here is the number 3 encoded in a half precision float:

  0       10000       1000000000
  sign    exponent    truncated mantissa

We find the three elements of the scientific notation of this same number:
  - a sign, encoded on 1 bit
  - an exponent (multiplicand) of bias 15, encoded on 5 bits
  - a truncated mantissa (significand) that records the number in its normalized form

First, encoding bias uses the unsigned binary encoding of a number X to represent a number Y.
The difference between X-Y is the bias of the representation.
This bias is valid for all numbers that can be represented on the n encoding bits, basically X+1 = Y+1, etc.
To find the value Y of a given X, simply subtract the bias from X.

A normalized number is the significand of a number in binary scientific notation, from which we remove the most significant bit set to 1 that is located after the decimal point, hence 
the name truncated mantissa.
The reason behind this removal is that the bit in question will always be 1—except in rare cases defined by the standard—and that its removal allows for greater precision in the representation of 
numbers, among other things.
