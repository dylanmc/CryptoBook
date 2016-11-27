Introducing Cryptol
===================

.. index:: algorithm, Cryptol
Now that you understand how data can be represented in bits and have
been introduced to cryptography using a paper computer, you're ready to
learn a computer language that was designed for implementing
cryptographic *algorithms*. An algorithm is a careful description of the
steps for doing something. Algorithms themselves are math, but when you
implement them, they're programs. The Caesar cipher from chapter one is
a simple cryptographic algorithm. In this chapter, you'll learn about
the programming language *Cryptol*, which was designed to make
cryptographic implementations look as much like their mathematical
algorithm description as possible. By the end of this chapter, you'll
understand how to read and write simple cryptographic programs using
Cryptol, and will be ready for the more complicated algorithms in the
next chapter.

Installing and running Cryptol on your computer
-----------------------------------------------

How exactly to install and run Cryptol depends on whether your computer
is running MacOS, Windows or Linux. Instructions for all three are
provided on the Cryptol website, which is ``http://cryptol.org``

You're ready to go with the rest of the chapter (and book) if you can
run Cryptol, and follow along with the session below. In the examples
below, **``what you type will be in bold``**, and
``what the computer types will be in non-bold (like this)``.

*start cryptol, however you're supposed to on your system*

.. literalinclude:: 03_code-01.icry
   :language: console

Here you've asked Cryptol to evaluate :math:`4 + 2`, and it responded
with ``0x6``, which is a hexadecimal answer, but it's still just 6.

Types of data in Cryptol
------------------------

.. index:: type
One of Cryptol's main features is that it is very careful about how data
is represented. How data is layed out in bits is an example of a *type* in
Cryptol. When you're talking about the type of something, often you'll
see the thing, a colon (:) and then a description of its type

For example, the type of the hex constant ``0xFAB`` would be written:

::

    0xFAB : [12]

You could read the above as "the type of the hex constant ``FAB`` is a
sequence of twelve bits." Cryptol can also talk about sequences of
sequences. For example:

::

    [ 0xA, 0xB, 0xC, 0xD, 0xE ] : [5][4]

.. index:: sequence
I would call this "a sequence of five elements, each having four bits".
You can ask Cryptol what the type of something is with the ``:t``
command, like this:

.. code-block:: console

  Cryptol> :t 0xAB
  0xab : [8]

Here we've said "Hey, Cryptol: what's the type of Hex AB?" and Cryptol
replied (in a friendly robotic voice) "Hex AB has the type *a sequence
of length 8 of bits*".

What do you think the type of a string of text should be? For example,
what should the type of ``"hello cryptol"`` be? Stop reading for a
minute and think about it.

Really, don't just read ahead, think about the type of the string
``"hello cryptol"``.

Okay, did you think about it? What did you come up with? One way to
start answering questions like this one is outside in. By that I mean
start by counting how many elements there are. In this case the length
of ``"hello cryptol"`` is 13 characters. So, the start of the Cryptol
type would be ``[13]``. Next, think about the type of each character.
Remember that ASCII characters are 8-bits each, so the rest of the type
is [8]. To check your answer you can just ask Cryptol:

.. code-block:: console

  Cryptol> :t "hello cryptol"
  "hello cryptol" : [13][8]

Enumerations: sequence shortcuts
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. index:: enumeration
Cryptol has some fancy ways of creating sequences other than just having
you type them in. One way is called *enumerations*. They're a short-hand
way of writing sequences of numbers that increment in a predictable way.
Here are some examples:

.. code-block:: console

  Cryptol> [1 .. 10]
  Assuming a = 4
  [0x1, 0x2, 0x3, 0x4, 0x5, 0x6, 0x7, 0x8, 0x9, 0xa]

The ``Assuming a = 4`` is Cryptol helpfully telling you that it decided to
use 4 bits per element of the sequence, because we weren't specific.
From here on out, I'll leave out the ``Assuming...`` messages, unless
they matter. Cryptol used 1 as the lower bound, 10 as the upper bound
(which is ``0xa`` in hex) and it incremented by one for each of the
elements in between.

You can increment by a different amount by providing two starting
elements. The step-value is the difference between them. For example:

.. code-block:: console

  Cryptol> [1, 3 .. 10]        // the step here is 2 (because 3-1=2)
  [0x1, 0x3, 0x5, 0x7, 0x9]
  Cryptol> [10, 9 .. 1]        // counting down (step = -1)
  [0xa, 0x9, 0x8, 0x7, 0x6, 0x5, 0x4, 0x3, 0x2, 0x1]``

Comprehensions: manipulating sequences
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. index:: sequence comprehension, variable
In addition to shortcuts for creating sequences, Cryptol has powerful
ways of manipulating them, called *sequence comprehensions*. The way you
write them in Cryptol is based on mathematical notation, so once you get
used to them, you'll know some advanced math notation, too!

Here's how it works: a sequence comprehension is inside of square
brackets, just like the sequences we've seen already. Then inside of
that, there are two parts: first is a formula for building each element
of the sequence The formula is a mathematical expression that can have
one or more *variables* in it. The second part is to define the values
of the variables as being *extracted* from other sequences. This will
make more sense with some examples:

.. code-block:: console

  Cryptol> [ 2 * x | x <- [1 .. 10]]
  Assuming a = 4
  [0x2, 0x4, 0x6, 0x8, 0xa, 0xc, 0xe, 0x0, 0x2, 0x4]

Reading the line we typed in goes like this: "Construct a sequence whose
elements are two times x, where x is drawn from the list one to ten."

Cryptol helpfully told us that it decided the elements of the list are
four bits each. Without being told otherwise, that's also the size of
the elements of the new list, which is why our numbers wrap around to
0x0, 0x2, 0x4 at the end. If we want Cryptol to keep track of more bits
in our output sequence, we can specify the type of the comprehension,
like this:

.. code-block:: console

  Cryptol> [ 2 * x | x <- [1 .. 10]]:[10][8]
  [0x02, 0x04, 0x06, 0x08, 0x0a, 0x0c, 0x0e, 0x10, 0x12, 0x14]

Here we've asked for the comprehension's type to be ten elements of
eight bits each, and the result doesn't wrap around.

Defining functions
------------------

.. index::
    single: function
    single: parameters
    single: defining functions
    single: function definition
In math, *functions* describe a way of creating an *output* from one or
more *inputs*. Functions in Cryptol are the same thing, and you can give
them names if you want. Here's a picture example of a functioned named
:math:`f`, which takes two *parameters*, :math:`x` and :math:`y` and
returns their sum:

::

    Inputs            Function                  Output
                    _________________
    7 ---------> x |                 |
                   | f(x,y) = x + y  |---------> 12
    5 ---------> y |_________________|

    (TODO: make this pretty)

One way to define a function is with the ``let`` command, like this:

.. code-block:: console

  Cryptol> let double x = x + x
  Cryptol> double 4
  Assuming a = 3
  0x0

What? :math:`4+4=0`? Oh, yeah, Cryptol let us know it was working with 3
bits, because that's how many you need for 4, but 4+4 is 8 which needs 4
bits, and the remainder is 0. The quickest way to get Cryptol to work
with more bits is to use hex and add a leading 0:

.. code-block:: console

  Cryptol> double 0x04
  0x08

Whew. That's better. Here's a definition of our function :math:`f`,
which has two parameters:

.. code-block:: console

  Cryptol> let f x y = x + y
  Cryptol> f 0x07 0x05
  0x0c

If you're tired of reading hex, you can ask Cryptol to speak back to you
in decimal:

.. code-block:: console

  Cryptol> :set base=10             // <- use base 10 output
  Cryptol> f 0x07 0x05
  12

You can also call functions inside a sequence comprehension, like this:

.. code-block:: console

  Cryptol> [ double x | x <- [ 0 .. 10 ]]
  Assuming a = 4
  [0x0, 0x2, 0x4, 0x6, 0x8, 0xa, 0xc, 0xe, 0x0, 0x2, 0x4]

And it should be no surprise that you can call functions from inside
functions:

.. code-block:: console

  Cryptol> let quadruple x = double (double x)
  Cryptol> quadruple 0x04
  16      <-  we still have output set to base 10

Functions on sequences
----------------------

Now that you know about functions and sequences, it's time to learn
about some functions that operate on sequences.

Extracting elements from sequences
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. index:: index operator
The first one is called the *index operator*. That's a fancy way of
saying getting the n\ :sup:`th` element out of a sequence. It works
like this:

.. code-block:: console
   :linenos:

   Cryptol> let alphabet=['a' .. 'z']
   Cryptol> alphabet @ 5
   102
   Cryptol> :set ascii=on
   Cryptol> alphabet @ 5
   'f'

.. index:: zero-based indexing
On line 1, we created a variable called *alphabet*, which is a sequence
of 8-bit integers that are the ASCII values of the letters of the
alphabet. On line 2 we used the *index operator*, which is the ``@``
symbol, to extract the element at location 5 of that sequence, which is
102. Since wanted to see it as a character, on line 4 we used
``:set ascii=on``, which tells Cryptol to print 8-bit numbers as
characters. Finally, on line 5 we re-did the ``@`` operation, which gave
us ``f``, which is the 6\ :sup:`th` letter of the alphabet. Why the 6th
character and not the 5th? Cryptol, like most programming languages,
uses *zero-based indexing*, which means that ``alphabet @ 0`` is the first
element of the sequence, ``alphabet @ 1`` is the second element and so
on.

.. index:: reverse index operator
Cryptol also provides a *reverse index operator*, which counts backwards
from the end of the sequence, like this:

.. code-block:: console

  Cryptol> alphabet!25
  'a'
  Cryptol> alphabet!0
  'z'

What happens if you try to go off the end (or past the beginning) of a
sequence? Let's try:

.. code-block:: console

  Cryptol> alphabet@26
  invalid sequence index: 26

.. index:: infix operators
One more thing: ``@`` and ``!`` act a lot like functions, but they're
called *infix operators*. The only difference between a function and an
operator is that when you call a function, its name comes first followed
by the values you want the function to operate on (we call those its
*arguments*). Operators only work with two arguments, and the operator
name comes *between* the two arguments. All of the normal math operators
you're familiar with are infix operators, like: :math:`5 + 2 - 3`.

.. index::
   single: arguments
   single: parameters
   single: arguments vs. parameters
***Arguments vs. parameters***: when we talk about defining and calling
functions, we've talked about both *arguments* and *parameters*, so you
may wonder "what's the difference?" The answer is that *parameters are
in a function's definition*, and *arguments are what you pass to a
function when you call it*. So:

.. code-block:: console

  let foo x y = x - y   // x and y are the parameters of *f*
  f 5 3                 // here we've passed 5 and 3 as arguments to f

Reversing a sequence
~~~~~~~~~~~~~~~~~~~~

Cryptol provides a function called ``reverse``. Let's try it:

.. code-block:: console

  Cryptol> reverse ['a' .. 'z']
  "zyxwvutsrqponmlkjihgfedcba"`

Pretty handy!

Concatenating sequences
~~~~~~~~~~~~~~~~~~~~~~~

The ``#`` operator combines two sequences into one sequence, like this:

.. code-block:: console

  Cryptol> ['a' .. 'z'] # ['0' .. '9']
  "abcdefghijklmnopqrstuvwxyz0123456789"

"Rotating" elements of a sequence
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The ``>>>`` and ``<<<`` operators *rotate* the elements of a sequence
:math:`n` places. For example,

``['a' .. 'z'] >>> 1`` returns ``"zabcdefghijklmnop qrstuvwxy"``. All of
the elements get shifted 1 place to the right, but the ones that fall
off the end *rotate* back to the beginning.

``['a' .. 'z'] <<< 2`` returns ``"cdefghijklmnopqrs tuvwxyzab"``.
Everything moves to the left two places, but the first two, which fall
off the front, rotate around to the end.

Functions have types, too
~~~~~~~~~~~~~~~~~~~~~~~~~

    *This section is a deep-dive into Cryptol's fancy type system. You
    don't need to know this to complete the first few exercises, but
    it's really neat, and will help you understand some of the things
    Cryptol says to you.*

We mentioned earlier in this chapter that Cryptol is very careful about
the types of things. In addition to data, functions in Cryptol have a
type. The type tells you how many arguments a function takes as input,
and what type each of those arguments needs to have, as well as the type
of the output. Just like for data, you can ask Cryptol what the type of
a function is by using ``:t``, like this:

.. code-block:: console

  Cryptol> :t double
  double : {a} (Arith a) => a -> a

The way you read a function-type in Cryptol has two parts, which are
separated by a "fat arrow" (``=>``). Before the fat arrow is a
description of the types, and after the fat arrow is the description of
the inputs and the output. Each of them is separated by a "normal arrow"
(``->``). The last one is always the output. The ones before that are
the parameters.

Looking at our type of ``double``, we see that it operates on things
that you can perform arithmetic on ``(Arith a)``, it takes one argument
and produces output of the same type.

You can ask Cryptol about the types of an *infix operator* by
surrounding it with parentheses, like this:

.. code-block:: console

  Cryptol> :t (+)
  (+) : {a} (Arith a) => a -> a -> a

This says that plus takes two inputs, and produces one output, all of
which are *Arithmetic*.

What's an example of an input type that isn't Arithmetic? Concatenation
is one. Check this out:

.. code-block:: console

  Cryptol> :t (#)
  (#) : {front, back, a} (fin front) =>
  [front]a -> [back]a -> [front + back]a

This is a bit complex: What is says is that ``front`` and ``back`` are
both sequence-lengths, and that ``front`` is of finite length ``(fin)``.
After the ``=>``, it lets us know that the first argument has ``front``
elements, the second argument has ``back`` elements, and the output has
``front + back`` elements. The ``a`` everywhere lets us know that the
sequence could be of anything: a single ``Bit``, or another sequence, or
whatever. They do all have to be the same thing, though.

Implementing the Caesar cipher in Cryptol
-----------------------------------------

Using what you've learned so far, let's implement the Ceasar cipher in
Cryptol. Let's start by breaking down the process of encrypting and
decrypting data using the Caesar cipher.

Let's guess what the function declaration should look like. We know that
the encrypt operation takes a key and a message, so the function
declaration probably looks something like:

``caesarEncrypt key message =``

Let's talk about how we can represent the key. In Chapter 1, we talked
about the key being something like K\ :math:`\leftrightarrow`\ D, but
that's hard to represent mathematically. If we straighten out our Caesar
Cihper wheels into a line, it looks something like this:

::

    abcdefghijklmnopqrstuvwxyz <- outer wheel
    zyxwvutsrqponmlkjihgfedcba <- inner wheel

To use the code wheel in this arrangement, look up a character from the
top line, and the character directly below it is the encoded / decoded
translation of that character.

.. index:: rotate operator (>>>)
If we think about the *rotate* operator (``>>>``), we see that it does
something really useful. For example, let's rotate the inner wheel by 4:

::

    abcdefghijklmnopqrstuvwxyz <- outer wheel
    dcbazyxwvutsrqponmlkjihgfe <- inner wheel >>> 4

This corresponds to the ``A``\ :math:`\leftrightarrow`\ ``D`` key in the
``HELLO`` example in Chapter 1. It even makes sense: the description
(rotating the inner wheel by 4 positions) *sounds* like what we did with
the paper Caesar cipher.

At this point we'd *like to use* the index operator (``@``) to get the
cyphertext from the inner wheel that corresponds to the plaintext on the
outer wheel. The indexing operator needs to be a number, not a letter.
For the index operator to do what we want, plaintext 'a' should be '0',
'b' should be '1', all the way up to 'z' should be 25. Let's pause to
think about how to achieve that in Cryptol. First, remember that a
character in Cryptol is already a number: its ASCII code. So, what if we
subtract the ASCII code for 'a' from our plaintext character?

In ASCII, ``'a'`` is 0x61, so ``'a'`` - ``'a'`` is 0, which is a good
start. ``'b'`` is 0x62, so ``'b'`` - ``'a'`` is 1, which is also what
we're after. Finally, ``'z'`` - ``'a'`` is 25, so for that range of
characters, it's good! Here's a simple function that takes an ASCII
character and returns its index in the alphabet:

::

  Cryptol> let asciiToIndex c = c - 'a'

Using this function to encrypt one letter would look like this:\ [#]_

.. [#] Some of the examples on this page have backslashes (\\) in them: it's
    because they're on more than one line: if you type the \\, Cryptol
    will let you continue typing on the next line. Alternatively you can
    type it all on one line (and skip typing the \\).

.. code-block:: console

  Cryptol> let encryptChar wheel c = \
  wheel @ (asciiToIndex c)
  Cryptol> let codeWheel key = \
  reverse alphabet >>> key
  Cryptol> encryptChar (codeWheel 4) 'h'
  'w'

The ``encryptChar`` function takes a shifted wheel and a character
``c``. It uses the index operator to extract the element from the wheel
corresponding to the index value of the character. On the next line we
defined ``codeWheel`` to be the reversed-alphabet shifted by our key.
Finally we called our function. The first argument is our ``codeWheel``
with ``4`` as the key, and the second argument is our plaintext ``h``.
The output is ``w`` as we hoped.

Now we're ready to have Cryptol do this for every character in a string.
Remember our sequence comprehensions? Here's how that comes together:

.. code-block:: console

  Cryptol> let encrypt key message = \
  [ encryptChar (codeWheel key) c | c <- message ]
  Cryptol> encrypt 4 "hello"
  "wzssp"

Hooray!

Now, what about decryption?

If you recall from Chapter 1, encryption and decryption are the same
process. Let's test if that works:

::

  Cryptol> encrypt 4 "wzssp"
  "hello"

Since that's not a satisfying name for a decryption routine, we can
define ``decrypt`` in terms of our ``encrypt`` function:

.. code-block:: console

  Cryptol> let decrypt k m = encrypt k m
  Cryptol> decrypt 4 "wzssp"
  "hello"

Ah, much better. One thing to note here: in our definition of encrypt,
the parameters were called ``key`` and ``message``, but here we called
them ``k`` and ``m``. The reason that's not a problem is that when
you're defining a function, you are free to name the parameters whatever
you want - the only thing you have to remember is to use those same
names in the body of your function.

This has been a huge chapter. If anything didn't make sense, go back and
read it again, or ask a partner for help. We shouldn't go much further
without really understanding what we've done so far. If Cryptol gives
you mysterious errors instead of the output you expect, check what
you've typed very carefully - we'll learn more about the errors Cryptol
prints, and what you can learn from them.

Handling unexpected inputs
~~~~~~~~~~~~~~~~~~~~~~~~~~

Let's try encrypting something new:

.. code-block:: console

   Cryptol> encrypt 7 "I LOVE PUZZLES"

   [warning] at <interactive>:1:1--1:30:
     Defaulting type parameter 'bits'
                  of literal or demoted expression
                               at <interactive>:1:8--1:9
     to 3
     Assuming a = 7

     invalid sequence index: 232

Egads - what just happened? When I see something like this happen, I
first read the error message, then I think about what I did that could
cause it. Starting at the top, the ``[warning]...`` tells you advisory
things, not errors. That warning goes on for four lines, ending in
``to 3``. The line after that is the normal helpful Cryptol telling you
it's decided to use 7 bits for your ASCII string.

The problem is in that last line ``invalid sequence index: 232``. We've
tried to use the index operator (``@``) with an invalid argument.
``232`` is way bigger than 25 - where did that come from? We subtracted
``'a'`` to make sure our indexes were all between 0 and 25, right?

At this point, it's time to start thinking about what we did wrong to
cause this. Comparing this message to the one that worked, ``"hello"``,
there are two main differences: our new message is in ALL CAPS, and it
also has spaces in it. It turns out those are both problems we need to
fix.

Let's start by handling upper case input. There are (at least) two ways
we could do it. One is to have upper case input produce upper case
output, and the other is to just make everything lower case. I think the
second option is simpler, so let's do that.

Recall from Chapter 2's discussion about ASCII's clever design, that
there's a simple way to convert between upper and lower case. Here are
the Hex values of the ASCII codes for ``a``, ``A``, ``z`` and ``Z``

::

  +-------------+--------+--------+--------+--------+
  | Character   | A      | Z      | a      | z      |
  +-------------+--------+--------+--------+--------+
  | Hex ASCII   | 0x41   | 0x5a   | 0x61   | 0x7a   |
  +-------------+--------+--------+--------+--------+

.. index:: conditional statements
Hey, the difference between the upper and lower case values is exactly
0x20! If we want everything in lower case (WHO LIKES SHOUTING, REALLY?),
if a character is lower than 0x61, we can add 0x20 to make it upper
case. We use *conditional statements* to do that in Cryptol:

.. code-block:: console

  Cryptol> let toLower c = if c < 'a' then c + 0x20 else c
  Cryptol> toLower 'I'
  'i'

and just to make sure we didn't break already lower case input:

.. code-block:: console

  Cryptol> toLower 'i'
  'i'

As you can see, a conditional statement has three parts: the
*condition*, the *if-expression* and the *else-expression*.

Now we can use ``toLower`` to improve ``asciiToIndex``:

.. code-block:: console

  Cryptol> let asciiToIndex c = (toLower c) - 'a'

And now we can encrypt text with upper and lower case (but without
spaces):

.. code-block:: console

  Cryptol> encrypt 7 "iLOVEpuzzles"
  "yvslcrmhhvco"
  Cryptol> decrypt 7 "yvslcrmhhvco"
  "ilovepuzzles"

Now, how to handle spaces. The usual way to handle spaces with the
Caesar cipher (not in cryptography in general) is to pass them through.
Sure, it makes the code weaker (you can see the length of words), but
this part of the lesson isn't about good codes. To pass spaces through
from the input to the output, the best place to do that is with a
conditional in the encryptChar function:

.. code-block:: console

  Cryptol> let encryptChar wheel c = \
  if c == ' ' then c else wheel @ (asciiToIndex c)

Let's test it, first on a space (since that's our new feature), then on
an uppercase letter, and then on a lowercase letter:

.. code-block:: console

  Cryptol> encryptChar (codeWheel 7) ' '
  ' '
  Cryptol> encryptChar (codeWheel 7) 'I'
  'y'
  Cryptol> encryptChar (codeWheel 7) 'i'
  'y'

Yay, it looks like it'll work. Now let's encrypt and decrypt our
original message:

.. code-block:: console

  Cryptol> encrypt 7 "I LOVE PUZZLES"
  "y vslc rmhhvco"
  Cryptol> decrypt 7 "y vslc rmhhvco"
  "i love puzzles"

Wow - it all worked! If it didn't, go through the error messages, and
see if you can figure out what happened.

What we covered this chapter
----------------------------

We covered a lot of ground this chapter:

-  Launching Cryptol and asking about *types* of data with the ``:t``
   command,
-  *enumerations* are shortcuts for creating sequences, like
   ``[1 .. 10]``,
-  *comprehensions* are ways of manipulating elements of sequences,
-  *functions* define how to create an output value from one or more
   inputs (called *arguments*),
-  a number of functions that operate on sequences, like *indexing*,
   *reversing*, *concatenating*,
-  finally, we implemented the Caesar cipher in Cryptol, step by step:

   1. converting ASCII characters to indexes,
   2. rotating the alphabet to make an encryption sequence,
   3. indexing the encryption sequence to encrypt one character,
   4. using a *comprehension* to encrypt a whole string,
   5. using *conditional expressions* to convert uppercase to lowercase,
   6. and handling the space character, ``' '``, by passing it through.

That's a lot of stuff - congratulations!

