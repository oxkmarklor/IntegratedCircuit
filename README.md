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

As mentioned in the previous chapter, encoding floating point numbers in IEEE-754 requires scientific notation of a number to represent it within a computer.
Furthermore, we use the binary form of a decimal number to convert it to its binary scientific notation, which goes without saying.
A floating point number written in binary is obtained by converting that same number from its decimal form.
Let's take the value 3.75 as an example.

The integer part of 3.75, i.e. 3, is encoded in unsigned binary, while the fractional part 0.75 is also encoded in unsigned binary but with 
negative powers of 2.
As we can see, the sign of the number is not taken in count during conversion.
This is why it must be added using an additional bit set to 1 for the sign - and 0 for the +.
The result of the conversion is 0 11.11. However, I should point out that the `.` in the result is only there to make it easier to read.

In fact, without the sign bit, there is no difference between two floats X and -X. For example, 3.75 and -3.75 are encoded in the same way 
except for the sign bit, which differs from one number to another.
In other words, it is as if each floating point number were encoded as an absolute value, so we add a bit as a sign 
to distinguish between the number in positive and negative form.

The sign bit is located in place of the MSB (most significant bit) of each floating point number, and the weight of this bit has an unjustified impact on the value 
of the floating point number.
This is why floating-point units (FPUs) only work with the absolute value of floating-point numbers (without taking in count 
the sign bit), unlike an adder-subtractor for signed integers.

Consequently, when an instruction that operates on floating-point numbers is executed, there are sometimes complications due to the signs 
of each operand.
Let's take an operation that does not pose a problem to start with, let's say (+A)+(+B), i.e. A+B.
This calculation will require an adder that will calculate the sum |A|+|B| and add the common sign bit of the two operands to the result.
Now let's change the sign of operand B so that it becomes -B.
The operation will then be (+A)+(-B), which is equivalent to A-B, and the calculation will be performed in a subtractor circuit.

Note: There is no adder-subtractor circuit for floating-point operations, given that IEEE-754 encoding is not compatible 
with this calculation logic.

The floating-point subtractor in question shares some similarities with integer subtractors.
When a subtractor circuit performs a subtraction A-B of two unsigned integers, the circuit will only generate a valid result 
if |B|<=|A|. 
Since the floating-point subtractor works with the absolute values (without signs) of its operands, it is constrained in the same way as a 
calculation circuit that allows subtraction between two unsigned integers.

There is a universal solution to this problem: inverting the two operands of the A-B subtraction if |B|>|A|.
The calculation then becomes B-A, and the result is both positive (and therefore valid) and the opposite of the result initially sought.
It is then necessary to negate the result at the output of the subtractor to obtain the desired result.

This solution is precisely implemented for addition and subtraction operations on floating-point numbers by the integrated circuit that I designed.
I named this circuit the FPU Configuration Unit.
The circuit has three functions:
- comparison of the strict superiority of one floating-point operand over another
- generation of the parameter bit for a demultiplexer network located upstream of the FPU
- generation of the sign bit for the result at the FPU output

Here are all the floating-point additions and subtractions that pose a problem:
  - (+A)+(-B) -> A-B
  - (-A)+(+B) -> (-A)+B
  - (+A)-(+B) -> A-B
  - (-A)-(-B) -> (-A)+B

You can verify for yourself that these calculations do indeed have the potential to be problematic (we have already seen the case of the first one).
We can see that these operations amount to either an addition or a subtraction.
But there is a catch, because the sum (-A)+B is not possible. Remember that the floating-point adder is limited to working only with 
the absolute value of the floating-point operands.
Since (-A)+B is a calculation of the difference between the two operands, we can then perform the subtraction A-B and assign to the result 
a sign bit previously calculated by the FPU configuration unit.

We will see all this in detail in the next chapter.
