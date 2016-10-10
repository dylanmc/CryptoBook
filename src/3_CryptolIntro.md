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
Cryptol, and follow along with the session below. In the examples below, **`what
you type will be in bold`**, and `what the computer types will be in non-bold
(like this)`.

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
what should the type of `"hello cryptol"` be? Assume the text is ASCII, not UTF-8.
Stop reading for a minute and think about it.

Really, don't just read ahead, think about the type of the string `"hello cryptol"`.

Okay, did you think about it? What did you come up with? One way to
start answering questions like this one is outside in. By that I mean
start by counting how many elements there are. In this case the length
of `"hello cryptol"` is 13 characters. So, the start of the Cryptol type
would be `[13]`. Next, think about the type of each character.
Remember that ASCII characters are 8-bits each, so the rest of the
type is [8]. To check your answer you can just ask Cryptol:

`Cryptol> ` **`:t "hello cryptol"`**  
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
names if you want. Here's a picture example of a functioned named $f$,
which takes two variables, $x$ and $y$ and returns their sum:

```
Inputs            Function                  Output
                _________________
7 ---------> x |                 |
               | f(x,y) = x + y  |---------> 12
5 ---------> y |_________________|

(TODO: make this pretty)
```

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

Whew. That's better. Here's a definition of our function $f$, which
takes two inputs:

`Cryptol> ` **`let f x y = x + y`**  
`Cryptol> ` **`f 0x07 0x05`**  
`0x0c`  

If you're tired of reading hex, you can ask Cryptol to speak back to
you in decimal:

`Cryptol> ` **`:set base=10           `** $\leftarrow$ _use base 10 output_  
`Cryptol> ` **`f 0x07 0x05`**  
`12`

You can also call functions inside a sequence
comprehension, like this:

`Cryptol> ` **`[ double x | x <- [ 0 .. 10 ]]`**  
`Assuming a = 4`  
`[0x0, 0x2, 0x4, 0x6, 0x8, 0xa, 0xc, 0xe, 0x0, 0x2, 0x4]`

And it should be no surprise that you can call functions from
inside functions:

`Cryptol> ` **`let quadruple x = double (double x)`**  
`Cryptol> ` **`quadruple 0x04`**  
`16            `  $\leftarrow$ _we still have output set to base 10_


## Functions on sequences

Now that you know about functions and sequences, it's time to learn
about some functions that operate on sequences. 

### Extracting elements from sequences

The first one is
called the _index operator_. That's a fancy way of saying getting
the _n^th^_ element out of a sequence. It works like this:

_1:_ ` Cryptol> ` **`let alphabet=['a' .. 'z']`**  
_2:_ ` Cryptol> ` **`alphabet @ 5`**  
_3:_ ` 102`  
_4:_ ` Cryptol> ` **`:set ascii=on`**  
_5:_ ` Cryptol> ` **`alphabet @ 5`**  
_6:_ ` 'f'`

On line 1, we created a variable called _alphabet_, which is a
sequence of 8-bit integers that are the ASCII values of the letters of
the alphabet. On line 2 we used the _index operator_, which is the `@`
symbol, to extract the element at location 5 of that sequence, which
is 102.  Since wanted to see it as a character, on line 4 we used
`:set ascii=on`, which tells Cryptol to print 8-bit numbers as
characters. Finally, on line 5 we re-did the `@` operation, which gave
us `f`, which is the 6^th^ letter of the alphabet. Why the 6th
character and not the 5th? Cryptol, like most programming languages,
uses _zero-indexing_, which means that `alphabet @ 0` is the first
element of the sequence, `alphabet @ 1` is the second element and so
on.

Cryptol also provides a _reverse index operator_, which counts
backwards from the end of the sequence, like this:

`Cryptol> ` **`alphabet!25`**  
`'a'`  
`Cryptol> ` **`alphabet!0`**  
`'z'`  

What happens if you try to go off the end (or past the beginning) of a
sequence?  Let's try:

`Cryptol> ` **`alphabet@26`**  
`invalid sequence index: 26`

One more thing: `@` and `!` act a lot like functions, but they're called
_infix operators_. The only difference between a function and an operator is
that when you call a function, its name comes first followed by the
values you want the function to operate on (we call those its
_arguments_). Operators only work with 2-arguments, and the operator
name comes _between_ the two arguments. All of the normal math operators you're
familiar with are infix operators, like: $5 + 2 - 3$.

### Reversing a sequence

Cryptol provides a function called `reverse`. Let's try it:

`Cryptol> ` **`reverse ['a' .. 'z']`**  
`"zyxwvutsrqponmlkjihgfedcba"`

Pretty handy!

### Concatenating sequences

The `#` operator combines two sequences into one sequence, like this:

`Cryptol> ` **`['a' .. 'z'] # ['A' .. 'Z']`**  
`"abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"`

### "Rotating" elements of a sequence

The `>>>` and `<<<` operators _rotate_ the elements of a sequence $n$
places. For example,

`['a' .. 'z'] >>> 1 ` returns `"zabcdefghijklmnopqrstuvwxy"`. All of
the elements get shifted 1 place to the right, but the ones that fall
off the end _rotate_ back to the beginning.

`['a' .. 'z'] <<< 2 ` returns `"cdefghijklmnopqrstuvwxyzab"`.
Everything moves to the left two places, but the first two, which fall
off the front, rotate around to the end.

### Functions have types, too

> _This section is a deep-dive into Cryptol's fancy type system.
> You don't need to know this to complete the first few exercises,
> but it's really neat, and will help you understand some of the
> things Cryptol says to you._

We mentioned earlier in this chapter that Cryptol is very
careful about the types of things. In addition to data, functions
in Cryptol have a type. The type tells you how many arguments
a function takes as input, and what type each of those arguments
needs to have, as well as the type of the output. Just like for
data, you can ask Cryptol what the type of a function is by using
`:t`, like this:

`Cryptol> ` **`:t double`**  
`double : {a} (Arith a) => a -> a`

The way you read a function-type in Cryptol has two parts,
which are separated by a "fat arrow" (`=>`). Before the fat
arrow is a description of the types, and after the fat arrow
is the description of the inputs and the output. Each of them
is separated by a "normal arrow" (`->`). The last one is always
the output. The ones before that are the parameters.

Looking at our type of `double`, we see that it operates on things
that you can perform arithmetic on `(Arith a)`, it takes
one argument and produces output of the same type.

You can ask Cryptol about the types of an _infix operator_
by surrounding it with parentheses, like this:

`Cryptol> ` **`:t (+)`**  
`(+) : {a} (Arith a) => a -> a -> a`

This says that plus takes two inputs, and produces one output,
all of which are _Arithmetic_.

What's an example of an input type
that isn't Arithmetic? Concatenation is one. Check this out:

`Cryptol> ` **`:t (#)`**  
`(#) : {front, back, a} (fin front) =>`  
`      [front]a -> [back]a`  
`      -> [front + back]a`  

This is a bit complex: What is says is that `front` and `back` are both
sequence-lengths, and that `front` is of finite length `(fin)`. After
the `=>`, it lets us know that the first argument has
`front` elements, the second argument has `back` elements, and the
output has `front + back` elements. The `a` everywhere lets us know
that the sequence could be of anything: a single `Bit`, or another
sequence, or whatever. They do all have to be the same thing, though.

## Implementing the Caesar Cipher in Cryptol

Using what you've learned so far, let's implement the Ceasar Cipher in
Cryptol. Let's start by breaking down the process of encrypting
and decrypting data using the Caesar Cipher.

Let's guess what the function declaration should look like.
We know that the encrypt operation takes a key and a message, so
the function declaration probably looks something like:

`caesarEncrypt key message =`  

Let's talk about how we can represent the key. In Chapter 1, we talked about
the key being something like K$\leftrightarrow$D, but that's hard to
represent mathematically. If we straighten out our Caesar Cihper
wheels into a line, it looks something like this:
```
zyxwvutsrqponmlkjihgfedcba <- outer wheel
abcdefghijklmnopqrstuvwxyz <- inner wheel
```
If we then think about the _rotate_ operator (`>>>`), we see that they
do something really useful. For example, let's rotate the outer wheel
by 3:

```
zyxwvutsrqponmlkjihgfedcba <- outer wheel
xyzabcdefghijklmnopqrstuvw <- inner wheel: ['z' .. 'a'] >>> 3
```

Hey - that makes sense: and even the description (rotating the inner
wheel by 3 positions) _sounds_ like what we did with the paper Caesar Cipher.

