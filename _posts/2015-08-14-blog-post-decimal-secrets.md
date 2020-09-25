---
title: 'The Double's Secret
date: 2019-09-23
permalink: /posts/2019/29/the-double-secret/
tags:
  - double
---

1. Prerequisite

Normal and Subnormal (Denormal) numbers.

A normal value has a 1 before the binary point: 1._significand bits_ x 2^exp
A subnormal value has a 0 before the binary point: 0._significand bits_ x 2^exp
https://programming.guide/normal-vs-subnormal-floats.html

[How many floating-point numbers are in the interval [0,1]?](https://lemire.me/blog/2017/02/28/how-many-floating-point-numbers-are-in-the-interval-01/)

The correct answer is 1,065,353,216.
This is easy to work out yourself if you remember one basic, handy property of floats: adjacent floats are adjacent in bit representation, except -0.0f and 0.0f.
For example, 0x00000000 is +0.0f. 0x00000001 is the smallest non-zero positive float. 0x00000002 is the second smallest.
The only exception to this is -0.0f and +0.0f, which are 0x80000000 and 0x00000000. The rule works with denormal floats, normal floats, and even right on the line between the two. If you want the next positive float, you always just add 0x00000001.
Now, +1.0f happens to be 0x3f800000. Recall +0.0f is 0x00000000. The number of values between the two is 0x3f800000 - 0x00000000 == 0x3f800000. Write that in decimal and you get 1065353216. 

https://news.ycombinator.com/item?id=13759458

https://stackoverflow.com/questions/2978930/how-many-double-numbers-are-there-between-0-0-and-1-0

https://stackoverflow.com/questions/28112500/how-to-get-the-smallest-next-or-previous-possible-double-value-supported-by-the
```
    private static long howMany(float start, float end) {
		long count = 0;
		while (start < end) {
			start = Math.nextAfter(start, end);
			count++;
		}
		return count;
	}
	
	public static void main(String[] args) {
		System.out.println(howMany(0.0f, 1.0f));
	}
```

Mathematician: "Infinite"
Statistician: "Depends"
Programmer: "1.065.353.216"
https://twitter.com/olafurw/status/1278017890365640708