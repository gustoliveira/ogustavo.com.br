---
title: "Fun with Overflowing Integers"
date: 2025-02-21
draft: true
description: "A deep dive into integer overflow detection and handling in Go, exploring bitwise operations and creating a safe math library for time tracking applications."
comments:
  - text: "If the overflow happens on the lower end of the range, it's technically called 'underflow'"
  - text: "In C++ for example, signed integer overflow is undefined behaviour"
params:
  math: true
---

Today, I finally got around to fixing a fundamental issue with klog (my command line tool for time tracking): if someone were to use klog on a daily basis for a few trillion years, there would be an increased risk of incorrect computations due to integer overflow. In particular, after precisely **9223372036854775807** minutes worth of time tracking, the total time amount would wrap around and continue at **-9223372036854775808** minutes. I wouldn’t go as far to call this a time bomb ticking right there, but you know, time is money, and that’s where things get serious (especially in this ballpark). In all fairness, while the practical value of this fix is rather academic, it was mainly a fun exercise to dive into the bits and bobs of overflowing integers. I created a small Go library called safemath for this purpose, which I published on its own.

### Integer overflows

Signed integers are basic (primitive) types, whose values are represented by fixed-width bit patterns. The minimum and maximum values depend on the machine architecture and on the declared width. On 64-bit systems, for instance, the default numeric range stretches from **-9223372036854775808** (lowest possible value) to **9223372036854775807** (largest possible value). When using built-in mathematical operators such as \(+\) or \(*\), the respective computations are carried out via simple bitwise instructions. That, however, does not include any logical checks whether the calculation would fit into the representable value range. If the result were to exceed the boundaries, it would do so silently, and in a way that wouldn’t make any sense from an arithmetic point of 

One way to detect overflows is to run explicit checks on the operands ahead of time. For a computation with two operands \(A\) and \(B\), an overflow would necessarily occur if:

- \(A_{abs} \gt ({Max} - B_{abs})\) when adding
- \(A_{abs} \gt (\dfrac{Max}{B_{abs}})\) when multiplying

One tricky thing is that the minimum and maximum numbers don’t have the same absolute value, which requires extra caution: for example, if one of the operands happens to be the minimum number, we cannot trivially compute the inverse value by multiplying by \(-1\), and we also have to make sure that we don’t divide the minimum value by \(-1\). It’s worse for multiplications, because while \(\frac{1}{2}{Min} \times {2}\) is okay, \(\frac{1}{2}{Min} \times {-2}\) is not.

 

If overflow is defined behaviour, we can also carry out the computation naively, and then run a check on the result. In this case, an overflow can be detected by inspecting the signs of the operands and the resulting value. The overflow conditions are:
when adding two operands, which both have the same sign, the result must have the same sign
when multiplying two operands,
the result must have a positive sign if both operands have the same sign, or otherwise the result must have a negative sign,
and the according inverse operation \(\dfrac{Res}{B} = {A}\) must match. 
