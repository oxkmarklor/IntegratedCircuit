While studying electronics, particularly computer microarchitecture, I came to understand how closely linked 
the functioning of hardware and software is.

This document explains the function of an integrated circuit that I designed myself after studying the IEEE-754 standard as well as 
floating point units (FPUs) and SIMD units.

The circuit diagram does not follow any specific order of magnitude.
I am not a professional, so it is possible that the circuit is not optimal in terms of its architecture or due to certain 
physical limitations that may not have been taken in count.

The diagram is based on NMOS and PMOS transistors (rather than high-level logic gate diagrams) so that the circuit is closer to reality in its 
architecture, mainly to highlight the compromises I had to make.
Furthermore, most of the circuits are CMOS technology, but for reasons of compromise, some circuits 
are transmission circuits.

My primary intention with this project is to make the complexity of the algorithms behind our computers, as well as our favorite software and video games, more concrete (more visual).

// to update

The file is divided into several sections, as follows:
- i. reminder of the IEEE-754 standard
- ii. the problem solved by the circuit

# i. Reminder of the IEEE-754 standard

IEEE is a consortium of professionals in the field of electronics. 
Its role is to propose standards that are widely respected by major companies in the small world of electronic engineering.

The IEEE-754 standard aims to describe a universal encoding of floating-point numbers, regardless of a computer's architecture or microarchitecture.
The standard defines an encoding that uses the same elements as binary scientific notation:
- a sign (encoded by one bit)
- a multiplicand (several bits to encode the value of the exponent)
- a truncated mantissa (several bits to encode the number in its normalized form)

Let's quickly define the role of each field.

The sign bit field replaces the sign (+ or -) of binary scientific notation with a bit set to 1 for a - and 0 for a +.

The exponent field uses biased encoding to store a number N representing the power to which the base 2 
of the multiplicand is raised.
Biased encoding uses a constant value B (the bias) that is added to any number X that can be encoded in unsigned binary on an n-bit field. 
The sum X+B forms the number represented.

As for the truncated mantissa field, it uses the mantissa of the number to be encoded (defined by its binary scientific notation) by 
normalizing it by masking the most significant bit being 1 (by definition, it is always behind the decimal point).
This masking operation is an optimization that allows one bit of precision to be gained on the encoding of each number.
This is why this field is called the truncated mantissa.

There are several floating point formats defined by the IEEE-754 standard.
The two main ones are single precision and double precision floats of 32 and 64 bits respectively.
However, there are many other floating point formats such as half precision with a width of 16 bits.
To my knowledge, half precision is mainly used in neural network applications within the vast field of artificial intelligence .


The circuit I designed is only capable of handling 16-bit floating point numbers, which is why, in this document, we 
will focus more specifically on the half precision format.

Here is what the half precision encoding of a decimal number such as 3.75 looks like:

    0 10000 1110000000

Starting from the left and moving to the right, we observe each element of the binary scientific notation of our example number.
We have 1 sign bit set to 0 because the number is positive, an exponent field with a value of 1 and a bias of -15 (i.e., 
0b10000-0b01111) followed by the truncated mantissa field with a width of 10 bits.
Together, these form a 16-bit binary field.

Now that we have briefly reviewed what IEEE-754 encoding is, let's move on to the next chapter by contextualizing the problem 
that the circuit aims to solve.

# ii. The problem solved by the circuit

The problem with IEEE-754 floating point numbers is that it is quite difficult to perform certain simple operations such as addition or 
subtraction.
This problem does not apply to signed integers because they are encoded in two's complement.

Two's complement encoding uses a range from -2^(n-1) to 2^(n-1)-1 to encode numbers, where n is the width of a given binary field.
A positive number is simply encoded in unsigned binary.
If we want to obtain a negative number, we must first write the absolute value of the underlying integer in unsigned binary, then we 
find the complement of the number.
The complement of a number is its opposite, and we obtain it by inverting all the bits of the number (one's complement) followed by an 
increment of 1. 

I mention this because addition and subtraction operations between signed integers can be performed with a simple adder circuit.
An adder coupled to a controllable inverter circuit transforms the adder into an adder-subtractor, a circuit capable of adding 
signed integers but also subtracting them.
The controllable inverter is a small network of XOR logic gates where the first input of each gate is connected to one of the i-th bits of the second 
operand of the adder, and the second is connected to a parameter bit.
When executing an instruction that produces this (+A)-(+B), i.e. A-B, (+A)-(-B) equivalent to A+B, or (-A)-(-B), which corresponds 
to (-A)+B, the instruction sequencer will set the parameter bit of the controllable inverter to 1 to obtain the one's complement of the second 
operand B.

Here is the truth table for an XOR gate:

    | Bit Operand | Bit Parameter | Output  |
    |_ _ _ _ _ _ _|_ _ _ _ _ _ _ _|_ _ _ _ _|
    |      0      |      0        |    0    |
    |      0      |      1        |    1    |
    |      1      |      0        |    1    |
    |      1      |      1        |    0    |

Note that with the parameter bit set to 1, the Bit Operand input is inverted, and with 0, it is copied to the output.
We could have used Boolean algebra, which tells us that 1 XOR A return the inverse of A, and 0 XOR A return A itself.

Furthermore, when the controllable inverter is activated (parameter bit at 1), the carry input of the first full adder must be 
initialized to 1 (necessary to obtain the two's complement of the second operand B).

    Note: In the case of an instruction performing the following operation (-A)-(+B), which is equal to (-A)-B, it is possible 
          to reverse the order of the two operands using a demultiplexer network.
          Then, perform the two's complement of the first operand and then of the result at the output of the adder.

All this is possible thanks to the two's complement encoding of signed integers.
However, the IEEE-754 encoding of floating-point numbers is unfortunately not as flexible, and it is not possible to reproduce the same calculations with 
floating-point operands as those shown above.
