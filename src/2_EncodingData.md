# Encoding data into bits
_"There are 10 kinds of people in the world:
those who know binary and those who don't."_

Now that you've seen encryption and decryption at work, it's time to learn how
computers do it. Our Caesar's Cipher wheel is a paper computer which has an
alphabet of 26 elements. You've heard (most likely) that computers work with
ones and zeros. One's and zeros are not very helpful by themselves, so
people figured out how to represent integers, floating point numbers and all of
the letters in all of the languages around the world using only ones and
zeros 

The process of representing one set of things (integers, for example) using
another set of things (sequences of ones and zeros) is called _encoding_.
_Decoding_ is reversing the process; getting back the original information from
the new representation. In this chapter, we'll learn how to encode and decode
unsigned and signed integers, simple Latin alphabets, as well as the rest of
the alphabets in the world

## Encoding integers

If the joke at the beginning of this chapter makes sense, and you know about
_number bases_, encoding integers using ones and zeros is simply converting to
base 2, and you can skip to the next section.  To learn what this means, and
why that joke isn't leaving out eight kinds of people, read on[^1].

[^1]: Note that we didn't promise we'd convince you this joke is funny, only
that you'll understand what it's getting at.

We're so used to seeing a number like 533 and understanding it to mean
"five hundred and thirty-three" that we forget that it's
an encoding of a numeric value into the symbols 0, 1, 2 ... 8, 9. Reading
from right to left, the n^th^ digit is the $10^{n-1}$-place^[Remember that $10^0 = 1$]. So deconstructing our example number we get:

 ----------- ------  ------  ------  ------
    Exponent 10^3^   10^2^   10^1^   10^0^
       Value 1000    100     10      1
       Digit 0       5       3       3
 Digit value 0       500     30      3
 ----------- ------  ------  ------  ------

So finally we get $0 + 500 + 30 + 3 =$ 533.

### Binary representations of numbers

Encoding numbers in binary is the same recipe, but with 2 as the base
of the exponent instead of 10. Each place (digit) can either have a 1
or a zero in it. As a result, you need more digits to represent the
same values, but it works out.

----------- ----- ----- ----- ----- ----- ----- ----- ----- ----- -----
   Exponent $2^9$ $2^8$ $2^7$ $2^6$ $2^5$ $2^4$ $2^3$ $2^2$ $2^1$ $2^0$
      Value 512   256   128   64    32    16    8     4     2     1
      Digit 1     0     0     0     0     1     0     1     0     1
Digit value 512   0     0     0     0     16    0     4     0     1
----------- ----- ----- ----- ----- ----- ----- ----- ----- ----- -----

In this case we get $512 + 16 + 4 + 1$ = 533.

To convert a number into binary, take the largest digit that isn't
bigger than your value, and set it to 1, then subtract that digit's value
from your value and repeat down the line, not forgetting to put in 0's
for the values you don't want to add.

Adding binary numbers is super-easy. It's a lot like
the addition you're used to, with carrying and everything, except simpler.
If both places are 0, the sum is 0. If either is 1, the sum is 1.
If both are 1, the sum is 0, and you carry 1. When you include the carry,
the rule is the same, except sometimes you have both 1 along with the carry,
in which case the sum is 1 and the carry is 1.

Here's a four-bit addition of 2 and 3:

~~~
   1    <- carry
  0010
+ 0011
------
  0101
~~~

If you're talking about numbers in different bases, and it's not clear
which one you're refering to, it's common to include the base in the subscript
after the number. So the joke at the beginning of the chapter would be
"There are 10~2~ types of people..."^[But that way kind of ruins the joke,
doesn't it?].

Working from the right, $0+1 = 1$ with no carry, then $1+1 = 0$ with carry,
and finally we have a carry with $0+0$, so that's 1. 101~2~ is 5, which is what
we're hoping for.

### Representing negative numbers:

> This section is a deep-dive. You can skip it the first time
> through - but if you get bored or ever get curious, come back here
> for some cool stuff.

The encodings we've discussed so far are only for positive numbers.
If you want to represent negative numbers, there are a few options,
but one fairly universally agreed-on best way to go.

The obvious (but not best) way is to reserve the top-most bit to represent
negation. If the bit is 0, the rest of the bits are a positive number,
if it's 1, the rest of the bits are interpreted as a negative number.
This encoding makes sense, but it makes arithmetic difficult. For example
if you had 4-bit signed numbers, and wanted to add -1 and 3, you'd get

~~~
   11  <- carry
  1001
+ 0011
------
  1100
~~~

This shows that if we apply our naive addition to $-1 + 3$, we get the
unfortunate answer -4. It turns out that if you represent negative numbers by
flipping the bits and adding one, you can do arithmetic using simple unsigned
operations and have the answers work out right.

To get a four-bit -1 in two's complement, here's the process:

~~~
Step 1: 0001   <- +1
Step 2: 1110   <- flipped
Step 3: 1111   <- add 1 is -1 in two's complement
~~~

Here's $-1 + 3$ again, in two's complement:

~~~
  111  <- carry
  1111 <- -1 (from above)
+ 0011 <- 3
------
  0010
~~~

In the one's place, $1+1=0$ carry 1, then we have $1+1$ + carry = 1 carry 1,
then we have 1 + carry = 0 with carry, and the last digit is also 1 + carry = 0
(and the carry goes away). You'll see the answer, $0010$~2~ = 2, which is what
we're hoping for.

<!---
### Representing floating-point numbers

*TODO:* do this.
-->

### Things to think about

 1. What's the largest value you can represent with one base-ten digit?
    Two digits?  $n$-digits?

 2. What's the largest value you can represent with one binary digit?
    Eight digits? $n$-digits?

 3. When we did $-1 + 3$, the carry bit got carried off the end of the
    addition. This is called overflow. In some cases (like this one),
    it's not a problem, but in other cases, it means that you get the wrong
    answer. Think about whether you can check whether overflow has occured
    either before or after the addition has happened.

 4. Two's complement is a slight change from _one's complement_, in which
    negative numbers just have their bits flipped, but you don't add a 1
    afterwards. A big advantage of two's complement is that there are two
    ways to write 0 in one's complement: 10000... and 0000.... Essentially
    you have a positive and a negative zero. Think about what problems
    this might cause.

 5. What's the largest value you can represent in a two's complement
    8-bit number? What's the smallest?

## Why ones and zeros?

It's a reasonable question - _why do computers only use ones and zeros_?  The
oversimplified, but essentially correct answer is performance and simplicity.
Making computers faster has been a goal since they were first invented.
_Simplicity enables speed_ is a common theme in computer engineering, and
binary code is a great example.  To represent values the voltage on a wire is
either _high_ (representing a 1) or _low_ (representing 0). What exact voltage
corresponds to high and low can vary. As systems get faster, the voltages that
make a "1" tend to decrease. In current Intel CPUs, for example, it's common
for a "1" to be as low as 1 volt. On older systems, it be as high as 13 volts.

_Transistors_ are the building blocks that work with the voltages inside
computers.  They're essentially just switches that can be controlled by a
voltage level.  A transistor has an input, an output, and a controlling switch.
It's easy to tell when a transistor is all the way "on" or "off", but measuring
values in between is much more complex and error-prone, so modern computers
don't bother with those, and instead just deal with "high" voltages and "low"
voltages.  Taking this approach has allowed us to create computers that can
switch many _billions of times per second_.

## Encoding text into ones and zeros

Now that you understand how numbers can be represented as ones and zeros,
we can explain how text can be represented as sequences of numbers, and
you can convert those numbers into bits.

It turns out that how to assign numbers to letters can be arbitrary.  Until the
early 1960's, there were a number of competing text $\rightarrow$ bits encoding
systems. People realized early on that deciding on one system would let them
communicate more easily between different machines. The most common text
encoding, called ASCII, was agreed on in 1963, and was in wide use through the
mid 1990's.

Here's how ASCII represents the basic letters, numbers and punctuation:

~~~
sp 32  ! 33  " 34  # 35  $ 36  % 37  & 38  '   39
(  40  ) 41  * 42  + 43  , 44  - 45  . 46  /   47
0  48  1 49  2 50  3 51  4 52  5 53  6 54  7   55
8  56  9 57  : 58  ; 59  < 60  = 61  > 62  ?   63
@  64  A 65  B 66  C 67  D 68  E 69  F 70  G   71
H  72  I 73  J 74  K 75  L 76  M 77  N 78  O   79
P  80  Q 81  R 82  S 83  T 84  U 85  V 86  W   87
X  88  Y 89  Z 90  [ 91  \ 92  ] 93  ^ 94  _   95
`  96  a 97  b 98  c 99  d 100 e 101 f 102 g   103
h  104 i 105 j 106 k 107 l 108 m 109 n 110 o   111
p  112 q 113 r 114 s 115 t 116 u 117 v 118 w   119
x  120 y 121 z 122 { 123 | 124 } 125 ~ 126 del 127
~~~

So the string "Hi there" in ASCII is: 72, 105, 32, 116, 104, 101, 114, 101.

#### Some exercises

 1. Encode your name in ASCII.

ASCII has some clever design features. Here are some questions that
may uncover some of that cleverness:

 2. Is there an easy way to convert between upper and lower-case in ASCII?
    Think about the binary representations.

 3. Is there an easy way to convert between a digit and its ASCII
    representation? Does the binary representation help here?
    What aspects of the ASCII encoding make this easy/difficult?


### Encoding _all_ languages: Unicode

> This section is a deep-dive: you can do the rest of the book knowing only
> ASCII. On the other hand, if you like to know how things work under the hood,
> you'll enjoy learning how non-Latin web pages are encoded and transmitted.

Up until the mid 1990's, computer systems that needed to process languages
whose characters are not in the ASCII tables each used their own encodings.
When the Internet and Word Wide Web started to gain adoption, people realized
that they would have to standardize how these other languages encoded their
alphabets into bits. The Unicode Consortium was the group founded to make
those standards. They took the sensible approach of splitting the problem
into two stages:

 1. Enumerating all of the symbols that can be represented. This includes
    accents, special glyphs, and now also includes emoji. As of 2016, there
    are over 1.1 million different "code points" in the master Unicode table.

 2. Devising efficient ways of representing sequences of those symbols as bits.

The hard work of the first stage is to come to agreement on which symbols go in
(and which to leave out), what to call them, and how to organize them. The
folks working on stage two have come up with a number of encodings, but the one
that is most common on the Internet is UTF-8.  The genius of UTF-8[^3] is that
it's _backwards compatible_ with ASCII. What that means is that if your text
_does_ fit in the ASCII table, the ASCII representation of it is also the UTF-8
representation of it. The key to making that work is that while ASCII is an
8-bit representation, the top-most bit of the ASCII table is always 0.

[^3]: UTF-8 was invented at Bell Labs by Ken Thompson, who co-invented Unix,
and Rob Pike, who subsequently invented the Go programming language.

If you're decoding a UTF-8 stream of bytes, and you encounter any byte with its
top bit off (i.e., its decimal value is <= 127), decode it as ASCII. If the top
bit is on (the number is > 127), follow this procedure:

 1. The first byte tells you how many bytes are in this character. Count the
    number of bits set before the first "0"-bit. That number is the number
    of bytes in this character.  The remaining bits after the 0 are data.
    UTF-8 supports up to 4 bytes, so the longest (4-byte) UTF-8 character will start
    `11110...`

 2. The remaining bytes are tagged with a leading "10" (so you can tell
    they aren't beginnings of characters), and the remaining 6-bits are data.

 3. Concatenate the data bits into one binary number.

 4. Look up that number in the Big Unicode Table.

Pretty cool!

### An aside: Hexadecimal

Writing numbers in binary is tedious for mere humans^[computers, on the other
hand, seem to thrive on tedium.]. It takes eight digits to count up to 128,
after all! Writing them in decimal is convenient for us humans, but a downside
is that there's no easy way to tell how many bits a number has. Computer
scientists have settled on _hexadecimal_, or base 16, to write numbers when the
number of bits matters. How does one write a hexadecimal number? After all,
we've only got ten digits, 0 -> 9, right? Well, as a convention we use the
first six letters of the alphabet to represent the digits past 9. So counting
to 16 in hexadecimal (or "hex" for short), looks like this:

1,  2,  3,  4,  5,  6,  7,  8,  9,  A,   B,   C,   D,   E,   F,   10

Hex, just like decimal and binary, has a _one's place_, but the next bigger
digit in hex is the _sixteen's place_^[and the next digit is the 256'ths
place!], so 10 in hex is 16 in decimal (also written as 10~16~ == 16~10~). A in
hex is 10 in decimal. This means that one hex digit holds exactly four bits,
and it takes two hex digits to hold a byte. This is important right now,
because Unicode tables are all written in hex, as you're about to see:

### Back to Unicode

Below is a table with three sample Unicode symbols. Each symbol has a long,
boring unambiguous name, its graphical symbol (which can vary from font to font), its global numeric code in the master Unicode table, and finally how that number is encoded in UTF-8.

![Some example Unicode glyphs, their official Unicode name, number and UTF-8 encodings](figures/UnicodeFunnyFigure.pdf)

In the table above, the "U+" lets you know that the hex number that follows is
the location in the Unicode table, and you see that the UTF-8 encoding is also
written in hex. There's a cool webpage at http://unicode-table.com/en/ that has
the whole table in one page. On the right of the page there is a live map with
dots in the parts of the world where the characters visible on the current
screen are used.

## Independent study questions

If you're interested into learning more about how information can be
digitally encoded, here are some questions you can research the answers
to.

 1. Two common ways of **encoding images** are pixel-based (or bitmap)
    and vector-based:

    a. The main aspects of **pixel-based** encoding are
       resolution (how many pixels there are in the image), how to encode
       colors (the value at each pixel), and compression (e.g., to reduce
       the storage for simple scenes like a plain blue sky). Common
       pixel-based formats are PNG, JPEG, and GIF.

    b. The main aspects of **vector graphics** are what _primitives_ to provide,
       which are the shapes that are supported built-in (lines, 
       curves, circles, rectangles) vs. which ones need to be assembled 
       from sequences of primitives, what the _coordinate system_ for
       describing shapes is, and what the _syntax_ is. Vector graphics formats
       tend to more-resemble programming languages, and are often in human-
       readable ASCII. Common vector-based formats are PDF, SVG, and PostScript.

    What's an image encoding method you know about? Use Google to find
    a specification for that format, and Write down how files in
    that format are structured. Most formats have a _header_ which
    provides _metatada_ about the file^[The word metadata literally means
    "data about data", which particularly makes sense in this context].

 2. **File archives** are encodings that combine a bunch of files and folders
    into one file that can be sent by email, or downloaded from a
    website, etc., and then _unpacked_ at the other end.  Archive formats
    often include the ability to compress the files as well. It's often
    surprising which file formats are archives. For example, most word
    processing document formats are file archives, to allow you to include
    graphics. Installers for most systems are also archives, such as
    Windows MSI files, MacOS DMG files, and Linux RPM files.  Early
    archive formats include TAR and ZIP, which were invented more than 30
    years ago, but are still used every day.

    If you know a particular file archive format, look it up on the Internet
    and write it up in a page or so.

## Take-aways

You've learned about how to encode data of different types (numbers,
characters) into binary representations. You've learned some binary arithmetic,
and why 10~2~ is 2~10~.  Finally you've learned that nerds (the author
included) can have a terrible sense of humor.
