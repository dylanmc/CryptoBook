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

## Encoding integers using ones and zeros

If the joke at the beginning of this chapter makes sense, and you know about
_number bases_, encoding integers using ones and zeros is simply converting to
base 2, and you can skip to the next section.  To learn what this means, and
why that joke isn't leaving out eight kinds of people, read on[^1].

[^1]: Note that we didn't promise we'd convince you this joke is funny, only
that you'll understand what it's getting at.

## Encoding text into ones and zeros

Now that you understand how numbers can be represented as ones and zeros,
we can explain how text can be represented as sequences of numbers, and
you can convert those numbers into bits.

It turns out that how to assign numbers to letters can be arbitrary.
Until the early 1960's, there were a number of competing text $\rightarrow$ bits
encoding systems. People realized early on that deciding on one
system would let them communicate more easily between different
machines. The most common text encoding, called ASCII, was agreed on
in 1963, and was in wide use through the mid 1990's.

Here's how ASCII represents the basic letters, numbers and punctuation:

~~~
 32 sp    33  !    34  "    35  #    36  $    37  %    38  &    39  '
 40  (    41  )    42  *    43  +    44  ,    45  -    46  .    47  /
 48  0    49  1    50  2    51  3    52  4    53  5    54  6    55  7
 56  8    57  9    58  :    59  ;    60  <    61  =    62  >    63  ?
 64  @    65  A    66  B    67  C    68  D    69  E    70  F    71  G
 72  H    73  I    74  J    75  K    76  L    77  M    78  N    79  O
 80  P    81  Q    82  R    83  S    84  T    85  U    86  V    87  W
 88  X    89  Y    90  Z    91  [    92  \    93  ]    94  ^    95  _
 96  `    97  a    98  b    99  c   100  d   101  e   102  f   103  g
104  h   105  i   106  j   107  k   108  l   109  m   110  n   111  o
112  p   113  q   114  r   115  s   116  t   117  u   118  v   119  w
120  x   121  y   122  z   123  {   124  |   125  }   126  ~   127 del
~~~

So the string "Hi there" in ASCII is: 72, 105, 32, 116, 104, 101, 114, 101.

_TODO_: come up with some exercises to do with ASCII

### Encoding other languages: Unicode[^2]

[^2]: This section is a deep-dive: you can do the rest of the book knowing only
ASCII. On the other hand, if you like to know how things work under the hood,
you'll enjoy learning how non-Latin web pages are encoded and transmitted.

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

The hard work of item 1 is to decide which symbols go in (and which to leave
out), what to call them, and how to organize them. The folks working on item 2
have come up with a number of encodings, but the one that is most common on the
Internet is UTF-8.  The genius of UTF-8[^3] is that it's _backwards compatible_
with ASCII. What that means is that if your text does fit in the ASCII table,
the ASCII representation of it is also the UTF-8 representation of it. The key
to making that work is that while ASCII is an 8-bit representation, the
top-most bit of the ASCII table is always 0.

[^3]: UTF-8 was invented at Bell Labs by Ken Thompson, who co-invented Unix, and Rob Pike, who subsequently invented the Go programming language.

If you're decoding a UTF-8 stream of bytes, and you encounter any byte
with its top bit off (<= 127), decode it as ASCII. If the top bit is on,
follow this procedure:

 1. This first byte (the one with the high-bit set) tells you how many
    bytes are in this character. Count the number of bits set before
    the first "0"-bit. That number is the number of bytes in this character.
    The remaining bits after the 0 are data.

 2. The remaining bytes are tagged with a leading "10" (so you can tell
    they aren't beginnings of characters), and the remaining 6-bits are data.

 3. Concatenate the data bits into one binary number.

 4. Look up that number in the Big Unicode Table.

Pretty cool!
