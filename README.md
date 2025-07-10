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
