---
layout: post
title: "Go: Basic Types"
date: 2017-11-29 00:00:00 +0000
tags: ["go", "golang", "programming"]
---

These are just some notes I've taken from [The Go Programming Language by Alan A. A. Donovan & Brian W. Kernighan](http://www.gopl.io/) (which I strongly recommend if you are willing to learn Go).

These notes and code samples are mainly for self-reference. Don't expect anything new or exciting here 🙄

Go's types can be grouped in 4 categories:

- interface types
- reference types
- aggregate types
- basic types

This post focus on basic types.

# Basic types

Basic data types can be numbers, strings or booleans.

## Numbers

Go has 3 types of numbers: integers, floating-points and complex numbers.

Signed numbers are represented in [2's-complement form](https://en.wikipedia.org/wiki/Two%27s_complement).

### Integers

The zero value for type `int` is… `0`.

Go provides signed (i.e. `int8`, `int16`, `int32`, and `int64`) and unsigned (i.e. `uint8`, `uint16`, `uint32`, and `uint64`) integers.

`int` and `uint` are the most efficient size for integers on a given platform. They have the same size, either 32 or 64 bits. Different compilers can make different choices on whether so use one or the other.

`rune` is a synonym for `int32` and indicates that a value is a Unicode code point. Runes are printed with verbs `%c` or `%q`:

```go
a := 'a'
fmt.Printf("%d %[1]c %[1]q\n", a) // "97 a 'a'"
```

`byte` is a synonym for `uint8`. It usually contains a piece of raw data.

`uintptr` is an unsigned integer type used to represent pointer values. It's size is not specified. It is generally used for low-level programming (i.e. when interoperating with C).

`int`, `int32`, `uint`, `uintptr`, and so forth, are different types and require explicit type conversion when used in a same operation:

```go
var _1 int32 = 1
var _2 int16 = 2
var sum int
sum = _1 + _2 // compile error: mismatched types int32 and int16
sum = int(_1) + int(_2) // works fine
```

When an overflow happens (the result of an arithmetic operation has more bits than its result type can hold), the high-order bits that do not fit are silently dropped.

Float to integer conversion discards any fractional part, truncating toward zero.

Integer literals can be also written as octal numbers if they start with `0`, like in `0444`, or as hexadecimal if they begin with `0x` (or `0X`), as in `0xcafe`.

By using the `fmt` package, we can control the radix and use the verbs `%d`, `%o`, and `%x` to format:

```go
o := 0444
fmt.Printf("%d %[1]o %#[1]o\n", o) // "292 444 0444"
x := int64(0xcafe)
fmt.Printf("%d %[1]x %#[1]x %#[1]X\n", x) // "51966 cafe 0xcafe 0XCAFE"
```

### Floating-Point Numbers

Go provides two sizes of floating-points, `float32` and `float64`. The limits of floating-point values can be found in the `math` package.

A `float32` provides about 6 decimal digits of precision. A `float64` provides about 15 digits.

Use scientific notation to write very small or large numbers, with the letter `e` or `E` preceding the decimal exponent:

```go
const Avogadro = 6.02214129e23
const Planck = 6.62606957e-34
```

Floating-point values are usually printed with `%g` verb. But it is also possible to use `%e` (exponent) or `%f` (no exponent) verbs.

```go
const Avogadro = 6.02214129e23
fmt.Printf("%g %8.3[1]f %[1]e\n", Avogadro) // 6.02214129e+23 602214128999999968641024.000 6.022141e+23
```

Testing whether a specific result is equal to `math.NaN` (not a number) is dangerous because comparisons with NaN always yields `false`.

### Complex Numbers

Go provides two sizes of complex numbers, `complex64` and `complex128`. The built-in function `complex` creates a complex number from its real and imaginary components. The built-in `real` and `imag` functions return the real and imaginary components, respectively:

```go
var x complex128 = complex(1, 2) // 1+2i
var y complex128 = complex(3, 4) // 3+4i
fmt.Println(x*y) // "(-5+10i)"
fmt.Println(real(x*y)) // "-5"
fmt.Println(imag(x*y)) // "10"
```

Floating-point literals or decimal integer literals immediately followed by `i`, as in `3.14i` or `1i`, denote a complex number with a zero real component.

The `math/cmplx` package provides functions for working with complex numbers.

## Booleans

Two possible values: `true` and `false`. The zero value for boolean types is `false`.

Use the verb `%t` to print booleans:

```go
fmt.Printf("%t", false) // "false"
fmt.Printf("%t", !false) // "true"
```

## Strings

In Go strings are immutable sequences of bytes. By convention, text strings are interpreted as UTF-8 encoded sequences of Unicode code points (runes). To know more about Unicode/UTF-8, I recommend [this excelent blog post from Joel Spolsky](https://www.joelonsoftware.com/2003/10/08/the-absolute-minimum-every-software-developer-absolutely-positively-must-know-about-unicode-and-character-sets-no-excuses/).

A string literal is a sequence of bytes inside double quotes, such as `"hello world"`. The zero value for string types is the empty string `""`.

Raw string literals are represented like `` `hello world` ``. In a raw string literal, no [escape sequences](https://en.wikipedia.org/wiki/Escape_sequence) are processed. Raw string literals are a generally used to write regular expressions, HTML, JSON or YAML.

The built-in `len` function returns the number of bytes (not runes) in a string, and the index operation `myString[i]` retrieves the i-th byte of `myString`.

The i-th byte of a string is not necessarily the i-th character of a string, because the UTF-8 encoding of a Unicode code point can require two or more bytes.

The `unicode` package provides functions for working with runes, and the `unicode/utf8` package provides functions for encoding and decoding runes as bytes using UTF-8.

It is possible to specify characters in a string using Unicode escapes followed by their numeric code points (h is a hexadecimal digit):

- `\U` hhhhhhhh for a 32-bit value
- `\u` hhhh for a 16-bit value

```go
fmt.Println("\xe4\xb8\x96\xe7\x95\x8c") // 世界
fmt.Println("\u4e16\u754c") // 世界
fmt.Println("\U00004e16\U0000754c") // 世界
```

The substring operation `s[i:j]` returns a new string based on the original one starting at index `i` until the byte at index `j` (`j` is not included). `i` and/or `j` operands can be omitted.

```go
s := "hello world"
fmt.Println(s[:5]) // "hello"
fmt.Println(s[7:]) // "world"
fmt.Println(s[:]) // "hello, world"
```

The `+` operator makes a new string by concatenating two strings.
