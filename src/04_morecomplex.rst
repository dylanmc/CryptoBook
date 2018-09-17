Implementing More Complex Programs
==================================

At this point, you've written some Cryptol at the command-line, but if
you had to leave your machine, reboot, or whatever, you had to start
from scratch. "That's no good", I hear you say, "there must be another
way!" Indeed there is.

Storing Cryptol programs in files
---------------------------------

In addition to typing commands at the Cryptol interpreter, Cryptol can
load files that have programs in them. Programs are slightly different
from the commands you've been typing so far. The main difference is that
you don't need ``let`` when you're defining a variable (like
``alphabet``) or a function (like ``encrypt``).

Creating a text file in a project directory
-------------------------------------------

.. index:: text file, text editors

The next thing you'll want to do is create a directory (or folder) to
keep your project files for this book. You'll need to figure out how to
edit *text files* on your computer. Text files are different from, say,
word processing documents, in that they do not have any formatting (no
**bold** or *italics*, for example). They're literally just sequences of
ASCII characters (or UTF-8 if you have a fancy editor). The command-line
programs ``vim`` and ``emacs`` are still popular, even though they look
like Wargames-era technology\ [#]_. More modern text editors include
*Sublime Text* (``https://www.sublimetext.com/``), *nano*
(``https://www.nano-editor.org/``), and many others. If you're on
Windows, and are in a pinch, both *Notepad* and *Wordpad* can "Save As"
*plain text*.

.. [#] no coincidence, either: the original versions of these programs
    were written almost 10 years before Wargames came out. They're still
    popular because they're very powerful, but that's a lesson for another
    book.

Exercise: rewriting our Caesar cipher program
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Start this exercise by creating your project directory. On a Unix
systems (like Linux or MacOS), you'll want to start up a Terminal
window, on Windows start a shell window (type ``cmd`` in the search
bar), and type something like this:

.. code-block:: console

   $ mkdir CryptoBook
   $ cd CryptoBook

Then, using whichever text editor you choose, create a file called
``caesar.cry`` inside that directory. It should contain the following:

.. code-block:: cryptol

    // Caesar cipher
    alphabet = ['a' .. 'z']
    toLower c = if c < 'a' then c + 0x20 else c
    asciiToIndex c = (toLower c) - 'a'

    encryptChar wheel c = if c == ' '
        then c
        else wheel @ (asciiToIndex c)

    codeWheel key = reverse alphabet >>> key

    encrypt key message =
        [ encryptChar (codeWheel key) c | c <- message ]

    decrypt key message = encrypt key message

The first line (``// Caesar cipher``) is a comment: it's text to remind
you what's in the file. With our more complex programs, we'll add more
comments. They'll help remind us what's going on.

Running Cryptol with ``caesar.cry``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Now that we have a file with our Caesar cipher in it. To run that
program, launch Cryptol from inside your project directory but include
the name of the file you want it to load, like this:


.. code-block:: console

  $ cryptol caesar.cry
                           _        _
      ___ _ __ _   _ _ __ | |_ ___ | |
     / __| '__| | | | '_ \| __/ _ \| |
    | (__| |  | |_| | |_) | || (_) | |
     \___|_|   \__, | .__/ \__\___/|_|
               |___/|_|  version 2.4.0

    Loading module Cryptol
    Loading module Main

  Main> encrypt 11 "using files now"

    [warning] at <interactive>:1:1--1:28:
      Defaulting type parameter 'bits'
                 of literal or demoted expression
                 at <interactive>:1:9--1:10
      to 3
    [0x6a, 0x6c, 0x76, 0x71, 0x78, 0x20, 0x79, 0x76,
     0x73, 0x7a, 0x6c, 0x20, 0x71, 0x70, 0x68]

Whoops, we still have to turn on ASCII output. If you got
``[error] can't find file: caesar.cry`` instead, then you need to use a
shell command to move into your CryptoBook directory. If that's proving
difficult, ask a partner or Google for *command line navigating files*
for the operating system you're running on.

Let's see our encrypted string:

.. code-block:: console

  Main> :set ascii=on
  Main> encrypt 4 "using files now"
  "jlvqx yvszl qph"

Hooray! You'll never have to type the Caesar cipher again.

Exercise: motivating stronger encryption
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Now that you can have a program perform the Caesar cipher for you, it's
a simple thing to write a program that cracks a message that has been
encoded with the Caesar cipher: just try all possible keys, and see
which one decodes into an intelligible message.

But before using brute force, let's look at the following encoded
message, and see if there are any clues to decryption:

::

    "seh zldy wuxkahz pdse seh jlhtlu jdwehu dt sehuh
    luh xyan sphysn tdo wxttdkah bhnt"

Before reading further, come up with two approaches you'd take to
decrypting this message, and try them out.

Now that you've done that approach, use brute force to crack this
message:

::

    "wlszknzlosgeyfzueyalknzqlsfmoaosqlhozzob"

If you don't know how to start, think about using a sequence
comprehension, where each element of the sequence is a decryption of
the message using a different key. For the Caesar cipher, the keys to
try would be ``[ 1 .. 25 ]``, right?

Implementing our next cipher: Vigenère
--------------------------------------

.. index:: Vigenère cipher

Given the previous exercise (please actually do it -- it's a lot of fun,
and very informative), it's probably occurred to you that the Caesar
cipher leaves a lot of room for improvement. You may have even thought
of some changes to the Caesar cipher that would make it harder to crack,
given the tools you've already developed.

Since we have brute force as an option, the main way to combat that
attack is to greatly increase the number of guesses a brute force
attacker will have to try. A simple approach to doing this is to have
the key be a sequence of shift amounts. This is essentially the idea
behind the Vigenère cipher. It was so successful at thwarting
decryption, it was used for almost 300 years - from the 1500's through
the 1800's, and was known for a lot of that time as "the unbreakable
cipher".

Here's how it works:

Take a plaintext message, like "how do you like my fancy new cipher" and
a key, like "thisismyfancykey", and to encrypt the i\ :sup:`th`
character, use the Caesar cipher with the i\ :sup:`th` character of the
key to specify the shift amount. To translate an ASCII key into a shift
amount, we do the classic "subtract the ASCII value of ``a`` from the
ASCII value of the key character".

.. index::
   single: ! modulo arithmetic

The last detail is what to do if the message is longer than the key.
What the Vigenère cipher does in this case is to "wrap around" to the
first character of the key, and so on. In mathematical terms, this is
known as *modulo arithmetic*. You're already familiar with modulo
arithmetic from how we read clocks: there are 24 hours in a day, but our
clocks only go to 12. For the hour past noon (or midnight) we "wrap
around" to 1, and the next hour is 2, and so on.

.. index::
   single: key expansion
   single: infinite sequences

Cryptol offers us a couple ways of expressing this notion of wrapping
around the key. The first one is to use modulo arithmetic on the index.
The second one is to create an *infinite sequence*, which consists of
the key appended to itself over and over. Using this infinite sequence,
we never have to worry about running out of key. This latter technique
is a simple version of what we call *key expansion* in more
sophisticated ciphers. Here's one way to implement the Vigenère key
expansion in Cryptol:

``expandKey key = key # expandKey key``

.. index:: recursion

In that one line of code, there are a number of things to explain!
First, it seems a bit magic (or cheating) that we're using ``expandKey``
in the definition of ``expandKey``. This trick is a technique in
programming called *recursion*. (POINT AT A RECURSION SECTION /
RESOURCE)

The second thing we need to explain is "how / why this doesn't run
forever, as soon as you expand any key - we haven't told Cryptol ever to
stop!" That's right, we haven't, but let's give it a try anyway. First,
copy your ``caesar.cry`` into a new file called ``vigenere.cry``
(because we'd like to reuse a lot of the code in ``caesar.cry``, and I
promised you wouldn't have to type it in again). Second, add the above
definition of ``expandKey`` to the end of your ``vigenere.cry`` Finally,
start up Cryptol:

.. code-block:: console

   $ cryptol vigenere.cry
                           _        _
      ___ _ __ _   _ _ __ | |_ ___ | |
     / __| '__| | | | '_ \| __/ _ \| |
    | (__| |  | |_| | |_) | || (_) | |
     \___|_|   \__, | .__/ \__\___/|_|
               |___/|_|  version 2.4.0

   Loading module Cryptol
   Loading module Main

   Main> :set ascii=on
   Main> let myXkey = expandKey "HELLO"
   Main> myXkey
   ['H', 'E', 'L', 'L', 'O', ...]
   Main> myXkey@1000
   'H'
   Main> myXkey@1001
   'E'

Let's go through that line-by-line. First, we ``:set ascii=on`` so we can
see the ASCII key strings. Second, we defined a temporary variable
``myXkey`` to be the result of expanding ``"HELLO"``. Next we asked
Cryptol to show it to us. But rather uninterestingly, it just showed us
the first five characters, but at the end it shows "...", signifying the
list goes on. So, we decide to test Cryptol's expansion by indexing our
``myXkey`` at index 1000, which happens to be an ``'H'``, and then the
next one, 1001, which is the expected ``'E'``. So far, so good!

.. index::
   single: parallel comprehension
   single: sequence comprehension

The last Cryptol feature you'll need to learn in order to implement the
Vigenère cipher is how to access two sequences at once in a sequence
comprehension. We need this because we need to access both the next
character of the expanded key stream and of the message in order to
produce the next character of the ciphertext. Cryptol's way of doing
this is called a *parallel comprehension*, and it looks like this:\ [#]_

.. [#] Note the \\'s - you can either type this all on one line, or
    use \\'s and include newlines where they appear here.

::

    Main> [ encryptChar (codeWheel k) c \
          | c <- "hi there" \
          | k <- expandKey [0 .. 10] ]
    "ss jwaoc"

One way to think of how parallel comprehensions work is like a zipper,
When you zip up your jacket, the pull joins the elements (teeth) from
each side and combines them (zips the teeth together). The length of the
resulting sequence is the shorter of the two lengths of the sides of the
zipper. In this case, it's the length of our message, "hi there",
because our expanded key has infinite length.

::

      h   i       t   h   e   r   e
      0   1   2   3   4   5   6   7   8   9   10   0   1 ...
      -> zip along the elements (TODO: make this pretty)

.. index:: lazy evaluation

This also explains how Cryptol doesn't have to run forever when you ask
it to define an infinitely expanded key: it only evaluates elements of a
list as you ask for them. As long as you ask for only a finite number of
them, Cryptol only evaluates that many of them. This way of approaching
infinite sequences and "evaluate on-demand" is called *lazy evaluation*;
it's a really powerful feature of Cryptol, and you'll see it's used
quite a bit.

This is *almost* the Vigenère cipher: we're shifting our plaintext a
different amount each time, but we're getting the shift amount from
``[0 .. 10]`` repeated, instead of a key string turned into indexes.

See if you can finish the implementation of the Vigenère cipher based on
what you know about Cryptol now.

It should start like this:

::

    vigenere key message =
      ... you fill in the rest

Exercises
~~~~~~~~~

1. Use your implementation of the Vigenère cipher to encode and decode
   some messages. Notice some of the improvements using longer keys
   makes.

2. Decrypt the following message using the key: ``"thisphraseismykey"``

   ``"qrucuvso dtoezje yzspd yordwt llsarvnij jzrwyam"``

.. index: known plaintext attack

3. One kind of attack against a code is when you know what some or all
   of a message is, and use that knowledge to learn something about the
   key. This was used during World War II when the Allied cryptanalysts
   guessed the word "weather" would appear in a German message that was
   encrypted with the Enigma machine. This technique is called a *known
   plaintext attack*.

   Think about how you could learn the key used in a Vigenère-encrypted
   message if you knew that a message started with the plaintext
   ``"At the tone the time will be...".`` started with the following
   ciphertext: ``"gg njc fyjg doa joec jyfv xi"``.

What we learned this chapter
----------------------------

We started by learning how to program in Cryptol using files, and how to
run Cryptol using a file you wrote. Next, we discussed some of the
weaknesses of the Caesar cipher, and even did some codebreaking that
uses those weaknesses. This lead to a discussion of increasing key
length, which is exactly what the Vigenère cipher does. We learned about
*key expansion*, and did it in Cryptol using *recursion*. We combined
the expanded key with the message using *parallel comprehensions*, and
learned that Cryptol uses *lazy evaluation* to avoid infinite loops when
sequences are infinitely long. Finally, you used the techniques learned
in this chapter to implement your own Vigenère cipher. The exercises
gave you a chance to exercise your new code, and learn about the *known
plaintext* technique to learn a key from an encoded message.
