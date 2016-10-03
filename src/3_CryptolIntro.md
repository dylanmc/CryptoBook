# Introducing Cryptol

Now that you understand how data can be represented in bits and have
been introduced to cryptography using a paper computer, you're ready
to learn a computer language that was designed for implementing
cryptographic _algorithms_. An algorithm is a careful description
of the steps for doing something. Algorithms themselves are math, but
when you implement them, they're programs. The Caesar cipher from chapter
one is a simple cryptographic algorithm. In this chapter, you'll learn
about the programming language _Cryptol_, which was designed to make
cryptographic implementations look as much like their mathematical
algorithm description as possible. By the end of this chapter, you'll
understand how to read and write simple cryptographic programs
using Cryptol, and will be ready for the more complicated algorithms
in the next chapter.

## Installing and running Cryptol on your computer

How exactly to install and run Cryptol depends on whether your computer
is running MacOS, Windows or Linux. Instructions for all three are
provided on the Cryptol website, which is `http://cryptol.org`

You're ready to go with the rest of the chapter (and book) if you can run
Cryptol, and follow along with the session below. In the examples below, **what
you type will be in bold**, and `what the computer types will be in non-bold
typewriter font (like this)`.

_start cryptol, however you're supposed to on your system_
```
                       _        _
  ___ _ __ _   _ _ __ | |_ ___ | |
 / __| '__| | | | '_ \| __/ _ \| |
| (__| |  | |_| | |_) | || (_) | |
 \___|_|   \__, | .__/ \__\___/|_|
           |___/|_|  version 2.4.0
```
`Loading module Cryptol`  
`Cryptol>  ` **`4 + 2`**  
`Assuming a = 3`  
`0x6`  
`Cryptol>`  

Here you've asked Cryptol to evaluate $4 + 2$, and it responded with
`0x6`, which is a hexadecimal answer, but it's still just 6.

## Types of data in Cryptol

One of Cryptol's main features is that it is very careful about
how data is represented. How data is layed out is an example of a
_type_ in Cryptol. When you're talking about the type of something,
often you'll see the thing, a colon (:) and then a description
of its type

For example, the type of the hex constant `0xFAB` would be written:
```
0xFAB : [12]
```

You could read the above as "the type of the hex constant `FAB` is a
sequence of twelve bits." Cryptol can also talk about sequences
of sequences. For example:
```
[ 0xA, 0xB, 0xC, 0xD, 0xE ] : [5][4]
```
I would call this "a sequence of five elements, each having four bits".
You can ask Cryptol what the type of something is with the `:t` command, like this:

`Cryptol> ` **`:t 0xAB`**  
`0xab : [8]`  
Here we've said "Hey, Cryptol: what's the type of Hex AB?" and Cryptol replied (in a friendly robotic voice "Hex AB has the type _a sequence of length 8 of bits_".

What do you think the type of a string of text should be? For example,
what should the type of "hello cryptol" be? Assume the text is ASCII, not UTF-8.
Stop reading for a minute and think about it.

Really, don't just read ahead, think about the type of the string "hello cryptol".

Okay, did you think about it? What did you come up with? One way to
start answering questions like this one is outside in. By that I mean
start by counting how many elements there are. In this case the length
of "hello cryptol" is 13 characters. So, the start of the Cryptol type
would be `[13]`. Next, think about the type of each character.
Remember that ASCII characters are 8-bits each, so the rest of the
type is [8]. To check your answer you can just ask Cryptol:

`Cryptol> ` **:t "hello cryptol"**  
`"hello cryptol" : [13][8]`

### Enumerations: sequence shortcuts
Cryptol has some fancy ways of creating sequences other than just having
you type them in. One way is called _enumerations_. They're a short-hand
way of writing sequences of numbers that increment in a predictable way.
Here are some examples:

`Cryptol> ` **`[1 .. 10]`**  
`Assuming a = 4`  
`[0x1, 0x2, 0x3, 0x4, 0x5, 0x6, 0x7, 0x8, 0x9, 0xa]`  

The _Assuming a = 4_ is Cryptol helpfully telling you that it decided to
use 4 bits per element of the sequence, because we weren't specific. From
here on out, I'll leave out the `Assuming...` messages, unless they matter.
Cryptol used 1 as the lower bound, 10 as the upper bound (which is `0xa` in hex)
and it incremented by one for each of the elements in between.

You can increment by a different amount by providing two starting elements.
The step-value is the difference between them. For example:

`Cryptol> ` **`[1, 3 .. 10]`**       // the step here is 2 (because 3-1=2)  
`[0x1, 0x3, 0x5, 0x7, 0x9]`  
`Cryptol> ` **`[10, 9 .. 1]`**       // counting down (step = -1)  
`[0xa, 0x9, 0x8, 0x7, 0x6, 0x5, 0x4, 0x3, 0x2, 0x1]`

### Comprehensions: manipulating sequences

In addition to shortcuts for creating sequences, Cryptol has powerful ways of
manipulating them, called _sequence comprehensions_. The way you write them in
Cryptol is based on mathematical notation, so once you get used to them, you'll
know some advanced math notation, too!

Here's how it works: a sequence comprehension is inside of square brackets,
just like the sequences we've seen already. Then inside of that, there are
two parts: first is a formula for building each element of the sequence
The formula is a mathematical expression that can have one or more
_variables_ in it. The second part is to define the values of the variables
as being _extracted_ from other sequences. This will make more sense
with some examples:

`Cryptol> ` **`[ 2 * x | x <- [1 .. 10]]`**  
`Assuming a = 4`  
`[0x2, 0x4, 0x6, 0x8, 0xa, 0xc, 0xe, 0x0, 0x2, 0x4]`

Reading the line we typed in goes like this: "Construct a sequence whose
elements are two times x, where x is drawn from the list one to ten."

Cryptol helpfully told us that it's decided the elements of the list are
four bits each. Without being told otherwise, that's also the size of the
elements of the new list, which is why our numbers wrap around to 0x0, 0x2, 0x4 at the end. If we want Cryptol to keep track of more bits in our output
sequence, we can specify the type of the comprehension, like this:

`Cryptol> ` **`[ 2 * x | x <- [1 .. 10]]:[10][8]`**  
`[0x02, 0x04, 0x06, 0x08, 0x0a, 0x0c, 0x0e, 0x10, 0x12, 0x14]`

Here we've asked for the comprehension's type to be ten elements of eight bits
each, and the result doesn't wrap around.

## Defining functions

In math, _functions_ describe a way of creating an _output_ from one or more
_inputs_. Functions in Cryptol are the same thing, and you can give them
names if you want.

One way to define a function is with the `let` command, like this:

`Cryptol> ` **`let double x = x + x`**  
`Cryptol> ` **`double 4`**  
`Assuming a = 3`  
`0x0`

What? Oh, yeah, Cryptol let us know it was working with 3 bits,
because that's how many you need for 4, but 4+4 is 8 which needs 4
bits, and the remainder is 0. The quickest way to get Cryptol to
work with more bits is to use hex and add a leading 0:

`Cryptol> ` **`double 0x04`**  
`0x08`

Whew. That's better. You can also call functions inside a sequence
comprehension, like this:

`Cryptol> ` **`[ double x | x <- [ 0 .. 10 ]]`**  
`Assuming a = 4`  
`[0x0, 0x2, 0x4, 0x6, 0x8, 0xa, 0xc, 0xe, 0x0, 0x2, 0x4]`

And it should be no surprise that you can call functions from
inside functions:

`Cryptol> ` **`let quadruple x = double (double x)`**  
`Cryptol> ` **`quadruple 0x04`**  
`0x10`  

## Cryptol feature requests

 1. As Sean has asked for years, a "learning" flag, for intro users - suppresses assuming and defaulting warnings
 2. How about :set ascii=on is really UTF-8? so if you did

    `[0xF0, 0x9F, 0x9A, 0xB2]` that you'd see the bicycle emoji?
