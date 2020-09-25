---
title: 'Floating point number to binary representation'
date: 2020-09-14
permalink: /posts/2020/09/floating-point-to-binary-representation
tags:
  - floating point
  - binary
  - IEEE 754
---

### Why are 0.3+0.2=0.5 and 0.1+0.2=0.30000000000000004 in JavaScript (Java double)??
This is because it’s using floating point number which is based on binary. There is no way to represent 1/10 exactly in base 2 using a finite number of binary places, it’s the same reason you can not represent ⅓ in base 10 with a finite number of decimal places. To represent 1/10 you need to add up a series of 1/(2^n). So 1/10 is

1/16 + 1/32 + 1/256 + 1/512 + 1/4096 + …

and on and on getting closer and closer

### http://qvanphong.tech/posts/floating-point-error-la-gi-tai-sao-0-1-0-2-0-30000000000000004/

### https://stackoverflow.com/a/28679423

When you convert .1 or 1/10 to base 2 (binary) you get a repeating pattern after the decimal point, 
just like trying to represent 1/3 in base 10. 
The value is not exact, and therefore you can't do exact math with it using normal floating point methods.


*My answer is quite long, so I've split it into three sections. Since the question is about floating point mathematics, I've put the emphasis on what the machine actually does. I've also made it specific to double (64 bit) precision, but the argument applies equally to any floating point arithmetic.*

**Preamble**

An [IEEE 754 double-precision binary floating-point format (binary64)](http://en.wikipedia.org/wiki/Double-precision_floating-point_format) number represents a number of the form

> value = (-1)^s * (1.m<sub>51</sub>m<sub>50</sub>...m<sub>2</sub>m<sub>1</sub>m<sub>0</sub>)<sub>2</sub> * 2<sup>e-1023</sup>

in 64 bits:

* The first bit is the [sign bit](http://en.wikipedia.org/wiki/Sign_bit): `1` if the number is negative, `0` otherwise<sup>1</sup>.
* The next 11 bits are the [exponent](http://en.wikipedia.org/wiki/Exponentiation), which is [offset](http://en.wikipedia.org/wiki/Offset_binary) by 1023. In other words, after reading the exponent bits from a double-precision number, 1023 must be subtracted to obtain the power of two.
* The remaining 52 bits are the [significand](http://en.wikipedia.org/wiki/Significand) (or mantissa). In the mantissa, an 'implied' `1.` is always<sup>2</sup> omitted since the most significant bit of any binary value is `1`.

<sup>1</sup> - IEEE 754 allows for the concept of a [signed zero](http://en.wikipedia.org/wiki/Signed_zero) - `+0` and `-0` are treated differently: `1 / (+0)` is positive infinity; `1 / (-0)` is negative infinity. For zero values, the mantissa and exponent bits are all zero. Note: zero values (+0 and -0) are explicitly not classed as denormal<sup>2</sup>.

<sup>2</sup> - This is not the case for [denormal numbers](http://en.wikipedia.org/wiki/Denormal_number), which have an offset exponent of zero (and an implied `0.`). The range of denormal double precision numbers is d<sub>min</sub> ≤ |x| ≤ d<sub>max</sub>, where d<sub>min</sub> (the smallest representable nonzero number) is 2<sup>-1023 - 51</sup> (≈ 4.94 * 10<sup>-324</sup>) and d<sub>max</sub> (the largest denormal number, for which the mantissa consists entirely of `1`s) is 2<sup>-1023 + 1</sup> - 2<sup>-1023 - 51</sup> (≈ 2.225 * 10<sup>-308</sup>).

---

**Turning a double precision number to binary**

Many online converters exist to convert a double precision floating point number to binary (e.g. at [binaryconvert.com](http://www.binaryconvert.com/convert_double.html)), but here is some sample C# code to obtain the IEEE 754 representation for a double precision number (I separate the three parts with colons (`:`):

    public static string BinaryRepresentation(double value)
    {
        long valueInLongType = BitConverter.DoubleToInt64Bits(value);
        string bits = Convert.ToString(valueInLongType, 2);
        string leadingZeros = new string('0', 64 - bits.Length);
        string binaryRepresentation = leadingZeros + bits;

        string sign = binaryRepresentation[0].ToString();
        string exponent = binaryRepresentation.Substring(1, 11);
        string mantissa = binaryRepresentation.Substring(12);

        return string.Format("{0}:{1}:{2}", sign, exponent, mantissa);
    }

---

**Getting to the point: the original question**

(Skip to the bottom for the TL;DR version)

[Cato Johnston](https://stackoverflow.com/users/62118/cato-johnston) (the question asker) asked why 0.1 + 0.2 != 0.3.

Written in binary (with colons separating the three parts), the IEEE 754 representations of the values are:

    0.1 => 0:01111111011:1001100110011001100110011001100110011001100110011010 
    0.2 => 0:01111111100:1001100110011001100110011001100110011001100110011010

[Representation of number](https://en.m.wikipedia.org/wiki/IEEE_754-1985#Representation_of_numbers)

[Convert double to binary tool](https://www.binaryconvert.com/result_double.html?decimal=049)

[Convert double to binary tutorial](http://mathcenter.oxford.emory.edu/site/cs170/ieee754/)

[Reference](http://cstl-csm.semo.edu/xzhang/Class%20Folder/CS280/Workbook_HTML/FLOATING_tut.htm)

[How will we convert (25.6875) in decimal to binary?](https://www.quora.com/How-will-we-convert-25-6875-in-decimal-to-binary)
[Decimal to binary](https://www.quora.com/How-do-you-convert-decimal-numbers-to-binary)
Note that the mantissa is composed of recurring digits of `0011`. This is **key** to why there is any error to the calculations - 0.1, 0.2 and 0.3 cannot be represented in binary **precisely** in a *finite* number of binary bits any more than 1/9, 1/3 or 1/7 can be represented precisely in *decimal digits*.

Also note that we can decrease the power in the exponent by 52 and shift the point in the binary representation to the right by 52 places (much like 10<sup>-3</sup> * 1.23 == 10<sup>-5</sup> * 123). This then enables us to represent the binary representation as the exact value that it represents in the form a * 2<sup>p</sup>. where 'a' is an integer.

Converting the exponents to decimal, removing the offset, and re-adding the implied `1` (in square brackets), 0.1 and 0.2 are:

    0.1 => 2^-4 * [1].1001100110011001100110011001100110011001100110011010
    0.2 => 2^-3 * [1].1001100110011001100110011001100110011001100110011010
    or
    0.1 => 2^-56 * 7205759403792794 = 0.1000000000000000055511151231257827021181583404541015625
    0.2 => 2^-55 * 7205759403792794 = 0.200000000000000011102230246251565404236316680908203125

To add two numbers, the exponent needs to be the same, i.e.:

    0.1 => 2^-3 *  0.1100110011001100110011001100110011001100110011001101(0)
    0.2 => 2^-3 *  1.1001100110011001100110011001100110011001100110011010
    sum =  2^-3 * 10.0110011001100110011001100110011001100110011001100111
    or
    0.1 => 2^-55 * 3602879701896397  = 0.1000000000000000055511151231257827021181583404541015625
    0.2 => 2^-55 * 7205759403792794  = 0.200000000000000011102230246251565404236316680908203125
    sum =  2^-55 * 10808639105689191 = 0.3000000000000000166533453693773481063544750213623046875

Since the sum is not of the form 2<sup>n</sup> * 1.{bbb} we increase the exponent by one and shift the decimal (*binary*) point to get:

    sum = 2^-2  * 1.0011001100110011001100110011001100110011001100110011(1)
        = 2^-54 * 5404319552844595.5 = 0.3000000000000000166533453693773481063544750213623046875

There are now 53 bits in the mantissa (the 53rd is in square brackets in the line above). The default [rounding mode](https://en.wikipedia.org/wiki/IEEE_754-1985#Rounding_floating-point_numbers) for IEEE 754 is '*Round to Nearest*' - i.e. if a number *x* falls between two values *a* and *b*, the value where the least significant bit is zero is chosen.

    a = 2^-54 * 5404319552844595 = 0.299999999999999988897769753748434595763683319091796875
      = 2^-2  * 1.0011001100110011001100110011001100110011001100110011

    x = 2^-2  * 1.0011001100110011001100110011001100110011001100110011(1)

    b = 2^-2  * 1.0011001100110011001100110011001100110011001100110100
      = 2^-54 * 5404319552844596 = 0.3000000000000000444089209850062616169452667236328125
    
Note that *a* and *b* differ only in the last bit; `...0011` + `1` = `...0100`. In this case, the value with the least significant bit of zero is *b*, so the sum is:

    sum = 2^-2  * 1.0011001100110011001100110011001100110011001100110100
        = 2^-54 * 5404319552844596 = 0.3000000000000000444089209850062616169452667236328125

whereas the binary representation of 0.3 is:

    0.3 => 2^-2  * 1.0011001100110011001100110011001100110011001100110011
        =  2^-54 * 5404319552844595 = 0.299999999999999988897769753748434595763683319091796875

which only differs from the binary representation of the sum of 0.1 and 0.2 by 2<sup>-54</sup>.

The binary representation of 0.1 and 0.2 are the *most accurate* representations of the numbers allowable by IEEE 754. The addition of these representation, due to the default rounding mode, results in a value which differs only in the least-significant-bit.

**TL;DR**

Writing `0.1 + 0.2` in a IEEE 754 binary representation (with colons separating the three parts) and comparing it to `0.3`, this is (I've put the distinct bits in square brackets):

    0.1 + 0.2 => 0:01111111101:0011001100110011001100110011001100110011001100110[100]
    0.3       => 0:01111111101:0011001100110011001100110011001100110011001100110[011]

Converted back to decimal, these values are:

    0.1 + 0.2 => 0.300000000000000044408920985006...
    0.3       => 0.299999999999999988897769753748...

The difference is exactly 2<sup>-54</sup>, which is ~5.5511151231258 × 10<sup>-17</sup> - insignificant (for many applications) when compared to the original values.

Comparing the last few bits of a floating point number is inherently dangerous, as anyone who reads the famous "[What Every Computer Scientist Should Know About Floating-Point Arithmetic](http://docs.oracle.com/cd/E19957-01/806-3568/ncg_goldberg.html)" (which covers all the major parts of this answer) will know.

Most calculators use additional [guard digits](https://en.wikipedia.org/wiki/Guard_digit) to get around this problem, which is how `0.1 + 0.2` would give `0.3`: the final few bits are rounded.

### Java: Why's Double.MIN_VALUE is positive? Integer.MIN_VALUE is negative!

https://programming.guide/java/integer-minvalue-negative-double-minvalue-positive.html

### https://medium.com/better-programming/why-is-0-1-0-2-not-equal-to-0-3-in-most-programming-languages-99432310d476

https://math.stackexchange.com/questions/1791562/converting-0-1-to-binary-64-bit-double
https://ryanstutorials.net/binary-tutorial/binary-floating-point.php