The Enigma
===========

.. figure:: figures/Enigma1.jpg
   :alt: The Enigma Machine
   :figclass: align-center
   :scale: 60%

.. index::
   single: Alan Turing
   single: Enigma
   single: World War II
   single: Bletchley Park

*Enigma* was the name of a cryptographic typewriter-like machine that
was used by Germany before and during World War II.  German military
commanders used the Enigma to encrypt instructions sent by radio to
their commanders across Europe and the Atlantic ocean. The Allies
could eavesdrop on the radio transmissions and hear the encrypted
messages, but without the key used for that day, they couldn't decrypt
them.  A top-secret team of mathematicians were gathered in Bletchley
Park, in England, with the task of breaking the Enigma. The team's
intellectual leader was a young man named Alan Turing.  The story of
how the Enigma was broken is told pretty well in the movie *The
Imitation Game*, and even better (but without Benedict Cumberbatch) in
the book *Alan Turing: The Enigma* by Alan Hodges.  Alan Turing's life
ended tragically due to how homosexual people were treated. The
British government issued a formal pardon and apology in 2013.  In
addition to ending WWII early, Alan Turing is credited with inventing
the modern computer, and artificial intelligence.

In this chapter you will learn how to use a cardboard model of the
Enigma that works exactly like the original. Then, step by step, we'll
implement the Enigma in Crytpol, and use it to encode and decode
messages.

.. finally, we'll use Cryptol's advanced features to break the enigma code.

Using a real Enigma
-------------------

If you ever get a chance to visit the National Cryptologic Museum\ [#]_, you
can use a real Enigma machine. Here's what you would do to encrypt
something with a real Enigma machine.

.. [#] The National Cryptologic Museum is next to Fort Meade, in
   Maryland, where the US National Security Agency is located. It's well worth
   visiting. It's about halfway between Washington, D.C, and
   Baltimore.

First, the key is the combination of two things: the *rotor settings*
and the *plugboard settings*. The rotors are the wheels at the top of
the machine. Choose a set of initial positions and write them down.
You probably will not be able to change the plugboard settings, but
write them down (or take a photo) if you can. After you configure the key, then you
start typing your message. As you type, the machinery moves, and
lights up letters in the area above the keyboard. As you type the
message, a partner writes down the sequences of lit-up letters. This
is the encrypted message. As you type, the rotors change positions, so
if you want to immediately decode the message, you'll need to reset
the rotor, then you start typing in the cyphertext. If things are
working right, the plaintext of your message will show up in the
lights.

Making and using your own Enigma
---------------------------------

Franklin Heath, a cybersecurity company in the UK, released an
excellent cardboard model of the Enigma. It is similar to a
pen-and-paper version of the Enigma designed by Alan Turing while he
was at Bletchley Park, and can be used to encrypt and decrypt messages
interchangeably with real Enigma machines.

The first thing you'll need to assemble your own paper Enigma is a
tube that has a diameter of 75mm. One source of such tubes is
Pringles cans (they call them "crisp tubes" in Britain). If you don't
want to buy a can of Pringles, you can make your own tube from
cereal box cardboard.

Let's build our paper Enigma step by step, and write Cryptol code that
simulates the system at each step.

Following the directions on the Franklin Heath paper Enigma wiki,
build the most simple Enigma, which has three paper bands: the
Input/Output band, Rotor I, and Reflector B. Make sure to line up the
gray bar on the Reflector and the I/O band, and tape those bands
stationary, while allowing the Rotor to move. The key in this Simple
Enigma is the letter between the gray bars when you start. Before
decoding each character, slide the rotor toward you one position, so
the next higher letter in the alphabet is between the grey bars.

Using this one-rotor setup, set the key to A, and decode the following
message:

..

  Y M X O V P E

The result should be two English words\ [#]_.

.. [#] If the answer doesn't look right, and starts with ``P``, then
   you forgot to advance the rotor *before* decoding each letter.

Implementing Enigma rotors in Cryptol
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Now let's write that in Cryptol.

If you think about what the Rotor does, it takes a letter as input,
and produces a different letter as output. If you put the Rotor's A
between the grey bars, you can trace out what each letter gets
transformed to. For example, A from the I/O band goes up to E, and B
goes to K.

If we want to express this in Cryptol, a sequence of letters
that are the result of translating A -> Z looks like this:


.. code-block:: console

  alphabet = "ABCDEFGHIJKLMNOPQRSTUVWXYZ"
  rotorI   = "EKMFLGDQVZNTOWYHXUSPAIBRCJ"

To use the rotor encoded like this, all we need is the *index
operator*, which is ``@``. To transform the letter ``C`` with the
rotor, we first get the index of ``C`` in the alphabet using
``asciiToIndex``, then we use that index on the ``rotorI`` sequence,
like this:

.. code-block:: console

  Cryptol> let i = asciiToIndex 'C'
  Cryptol> rotorI@i
  'M'

If you got a hex result instead, don't forget to ``:set ascii=on``.

Note that the rotor's function is not *self-inverting.* What this means
is that if ``C`` goes to ``M``, ``M`` does not go to ``C`` (in this case, it goes to
``O``.

Implementing the reflector in Crypol
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Now let's look at the Reflector. In this case what the reflector does *is*
self-inverting. The reflector connects, for example, A and Y. So A
input produces Y as output, and Y input produces A as output.

Come up with the Cryptol string that represents the Reflector's
actions. It should start like this:

.. answer: "ABCDEFGHIJKLMNOPQRSTUVWXYZ"
   "YRUHQSLDPXNGOKMIEBFZCWVJAT"

.. code-block:: console

  reflector = "YRU // ... you finish the rest

We use the reflector exactly the same way we used the rotor. In this
example, we've placed the call to ``asciiToIndex`` as the argument to
the index operator:

.. code-block:: console

  Cryptol> reflector @ (asciiToIndex 'C')
  'U'
  Cryptol> reflector @ (asciiToIndex 'U')
  'C'

Here we see that the reflector transforms ``C`` to ``U``, and because
it's self-inverting, ``U`` transforms to ``C``.

Running the reflector backwards
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Finally, we need to consider that the lines you trace go first from
right-to-left, go through the reflector, and then go left-to-right. So
looking at Rotor I again, if you start at the letters on the left side
of the ring, and trace them to the I/O band, they with A goes to U, B
goes to W, and so on.

We could go through, one by one, and produce another string that
represents the backwards transformation. However, we have the
information we want already in the previous Rotor I string. Look at
this:

.. code-block:: console

  alphabet = "ABCDEFGHIJKLMNOPQRSTUVWXYZ"
  rotorI   = "EKMFLGDQVZNTOWYHXUSPAIBRCJ"
              ^- shows E -> A     ^- shows A -> U

If we look at the letters in the ``rotorI`` string, we see that it
tells us the backwards-mapping too - because E is in the first
position, that tells us that E -> A. Because K is in the second
position, we know K -> B. We can automate the process of reversing
this operation in Cryptol! It's a bit tricky, so we'll go carefully:

.. code-block:: cryptol
  :linenos:

  indexOf c shuffle = candidates ! 0 where
      candidates = [ -1 ] # [ if c == s then i else p
                            | s <- shuffle
                            | p <- candidates
                            | i <- [ 0 .. 25 ]
                            ]

  invertShuffle shuffle = [ alphabet @ (indexOf c shuffle)
                          | c <- alphabet ]

.. index::
   single: recursion
   single: sequence comprehension
The first function we want is one that gives us the index of a
character in a shuffled string. Line 1 defines our function, and says
that it returns the last item of a sequence called ``candidates``. The
``where`` says we're about to define some variables (in this case only
one). Line 2 says that candidates is a sequence that starts off by
concatenating the sequence of one element (``[-1]``) with a *sequence
comprehension* (remember those from Chapter 3?). Each element of the
sequence is the result of an if statment: if ``c == s`` it's ``i``
otherwise it's ``p``. We don't yet know what any of those variables
(except ``c``) is yet, but fear not: they're defined right below. Line
3 says that ``s`` *is drawn from the elements of shuffle*. So each
time through the loop, ``s`` is the next element of the shuffle. Line
4 says that ``p`` is drawn from the elements of the ``candidates``
sequence. Interesting: We're using the sequence in the definition of
itself! Just like in Chapter 3, this is an instance of *recursion*.
Finally, line 5 says that ``i`` is drawn from the list ``[ 0 .. 25 ]``.

When this function runs, it builds up the ``candidates`` sequence,
starting with ``-1``, each element keeps being set to ``p`` (which
starts out with ``-1``) until the letter from shuffle being examined,
called ``s`` is equal to ``c``, the letter we're searching for. When
that happens, the new element of ``candidates`` gets set to ``i``,
which is the index of the match, because the numbers 0 .. 25 are the
indexes of the elements of shuffled list.

Here are the values of candidates as it proceeds through the shuffled
list, with the call ``findIndex 'L' rotorI``:

.. code-block:: console

   c: 'L'
   i:           [ 0,  1,  2,  3,  4,  5,  6, .., 25 ]
   candidates = [-1, -1, -1, -1,  4,  4,  4, .., 4  ]
   s:             E   K   M   F   L   G   D  ... J
                                  ^
   note:             s == 'L' here|, so the index i
                     is saved to candidates


With this function, we can create the left-to-right version of a rotor
given its right-to-left version:

.. code-block:: cryptol

  invertShuffle shuffle = [ alphabet @ (indexOf c shuffle)
                          | c <- alphabet
                          ]

Save these functions and the definition of ``rotorI``, ``reflectorB`` and
``alphabet`` to a file called ``enigma.cry``, and run Cryptol on it:

.. code-block:: console

  $ cryptol cryptol/enigma.cry
                          _        _
     ___ _ __ _   _ _ __ | |_ ___ | |
    / __| '__| | | | '_ \| __/ _ \| |
   | (__| |  | |_| | |_) | || (_) | |
    \___|_|   \__, | .__/ \__\___/|_|
              |___/|_|  version 2.4.0

  Loading module Cryptol
  Loading module Main
  Main> :set ascii=on
  Main> invertShuffle rotorI
  Assuming a = 7
  "UWYGADFPVZBECKMTHXSLRINQOJ"


