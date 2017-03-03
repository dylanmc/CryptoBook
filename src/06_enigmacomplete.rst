Completing the Enigma in Cryptol
================================

.. maybe-note:  *This chapter can be skipped the first time through if you're
  impatient. The Enigma is really cool, though, and there are some
  neat programming techniques in here, so make sure you come back to
  this chapter later on.*

Compared to our paper Enigma (and the real thing), our Cryptol version
lacks:

#) Multiple rotors,

#) Stepping the rotors, and the "rings" that control *when* the rotors step, and

#) the plugboard, which modifies the I/O ring

Let's implement them!

Combining multiple rotors
--------------------------

First get our your paper Enigma and set Rotor II and Rotor
III to ``A``. Then trace, from the right towards the left, each
letter from ``A .. Z``, just like we did for Rotor I in the
previous chapter. Starting with Rotor II (the middle one), you'll
see that the line from ``A`` goes straight across to ``A``, so
the first letter in that rotor is just ``A``.

As a starting point, here are the first few entries for rotors ``II``
and ``III``:

.. code-block:: console

   rotorIIchars  = "AJD ..."
   rotorIIIchars = "BDF ..."

Once you complete the rotor strings, the next job is to combine them
in the encryption function. One annoying thing that you may have noticed in our
``encryptOneRotor`` function is when we had to individually rotate the
``rotorOff`` and ``rotorRevOff`` sequences. Let's create a data
structure that lets us deal with an entire rotor (and its reverse) at
the same time:

.. literalinclude:: cryptol/threeRotors.cry
   :language: cryptol
   :start-after: BeginRotorType
   :end-before: EndRotorType

.. index::
   single: tuple

This declares ``RotorElement`` to be a *tuple*, which is a
comma-separated list of items inside parentheses. Unlike a sequence,
whose elements all have to be the same type, the elements of a tuple
can have different types--in this case, ``Offset`` and ``Bit``. The ``Bit`` at the
end is for the Rings, which we'll deal with in the next section.
To create a rotor from the characters we carefully traced, we need a helper function,
``buildRotor``:

.. literalinclude:: cryptol/threeRotors.cry
   :language: cryptol
   :start-after: BeginRotorDef
   :end-before: EndRotorDef

.. index::
    single: refactor

Returning to how the multi-rotor Enigma works, and thinking about
what we've written so far, our one-rotor Enigma is the left-hand half
of the 3-ring Enigma: the Reflector and first Rotor. We need to add
two more rotors to it. It's not quite as simple as that, though,
because our ``encryptOneRotor`` function takes a Char as input, and
returns a Char as output, but if we want to connect our rotors
together, the type of data that flows between them is ``Index``, not
``Char``. So we'll need to *refactor* our code a bit so that it's more
modular.

.. literalinclude:: cryptol/threeRotors.cry
   :language: cryptol
   :start-after: BeginReflectorDef
   :end-before: EndReflectorDef

Let's start with
our function that implements one rotor and the reflector.
If we want to add another rotor to our Enigma, we can call our
``doOneRotor`` function as a helper. First we go through our new
rotor forwards, then call the ``doOneRotor`` function, then use the
output of that, and go through our rotor backwards.

Exercise: write the three rotor function
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Follow the pattern established above, and write ``doThreeRotors``.
It should start off like this:

.. code-block:: console

   doThreeRotors refl r1 r2 r3 c =

Make sure you use ``doTwoRotors``, and keep track of how many
arguments it takes and what they are.

.. answer:
   doThreeRotors refl r1 r2 r3 i =
       doRotorRev r3 (doTwoRotors refl r1 r2
                     (doRotorFwd r3 i))

We can wrap this function with another function that translates a
``Char`` to ``Index`` and back again so that we can test our code:

.. code-block:: cryptol

    doOneChar c refl r1 r2 r3 =
        indexToChar ( doThreeRotors refl r1 r2 r3 (charToIndex c))

Now we can encode single characters with the three rotor Enigma code,
and compare it to our cardboard Enigma.
As a test, set all three rotors to ``A``, and make sure
that encoding ``A`` results in ``U``, and ``B`` goes to ``E``. If you
don't get those results, or if your program loads with errors, see if
you can figure out what's wrong with your version.

.. code-block:: console

   Enigma> doOneChar 'A' reflB rotorI rotorII rotorIII
   'U'
   Enigma> doOneChar 'B' reflB rotorI rotorII rotorIII
   'E'

Stepping the rotors
-------------------

Now that we have all three rotors going, our next step is to advance
them before encoding each character. Let's start by describing in a
bit more detail what the *advance rotors* function needs to do:

First, there are pins attached to each rotor. We're representing the pins
as the ``Bit`` in the ``Rotor`` data type. When the pin at
position 0 of the rotor is True, we need to
follow these rules:

 * If the middle rotor's pin is True, we advance all three
   rotors, otherwise
 * if the pin furthest from the reflector (rotor 3) is True
   we advance rotors 2 and 3, otherwise
 * just advance rotor 3.

Here's code that implements these rules, along with the helper
function ``getPins``:

.. literalinclude:: cryptol/threeRotors.cry
   :language: cryptol
   :start-after: BeginRotationsDef
   :end-before: EndRotationsDef

The function ``getPins`` has some tricky indexing, so here's an
explanation, step by step:

 #) ``rs@0`` is the first rotor (type: ``[26](Char,Bit)``)
 #) ``(rs@0)@0`` is the first element of the first rotor (type: ``(Char,Bit)``)
 #) ``((rs@0)@0).0`` is the character part of that first element (type: ``Char``)
 #) ``((rs@0)@0).1`` is the Pin part of that first element (type: ``Bit``)

Now we can write ``advanceRotors`` that applies these rules to our
rotors:

.. literalinclude:: cryptol/threeRotors.cry
   :language: cryptol
   :start-after: BeginAdvanceDef
   :end-before: EndAdvanceDef

Before we can test the function, we need to have some pins in
our rotors. Looking at the cardboard Enigma, each rotor's pin is at the
position where the letter has a grey background. So Rotor I's pin is
at ``'Q'``, Rotor II's pin is at ``'E'`` and Rotor III's pin is at
``'V'``. So all we need to do is pass to ``buildRotor`` a sequence of
bit's with the bit set accordingly. Here's a helper function that will
make that easy:


.. literalinclude:: cryptol/threeRotors.cry
   :language: cryptol
   :start-after: BeginPinsDef
   :end-before: EndPinsDef

Exercise
~~~~~~~~

If you drop the first 99 states, you'll see the sequence of states just before and after a
pin shows up in position 0 of Rotor II, so run this:

.. code-block:: console

  drop`{99} (enigmaStates [rotorI, rotorII, rotorIII])

Explain how the sequence of ``Rotor`` states do, or do not, follow the
rules for rotor advancement.

Implementing the plugboard
--------------------------

Here's how the plugboard works on the cardboard Enigma:
to configure the plugboard, you're given pairs of letters, like
``"AP"`` and ``"BR"``. In the little boxes on the I/O Ring, you write
a ``P`` next to the ``A`` and write an ``A`` next to the ``P``, and so
on. Then, when you're about to encode a letter, you first check if
there's a letter in the box, and if there is, you hop over to the
corresponding letter on the ring, and start encoding from there.
Similarly on the way back after the reflector and the rotors, you
return the letter in the box (if there is one) when you finish
following the lines.

The plugboard is a self-inverting substitution. We can easily represent this
as a string that starts out as the alphabet in order, and for each
pair of swaps in the plugboard, we swap those letters in the string.
Here's a helper function that does that:

.. literalinclude:: cryptol/threeRotors.cry
   :language: cryptol
   :start-after: BeginPlugboardDef
   :end-before: EndPlugboardDef

.. index::
  single: recursion

This function uses a different kind of recursion than we've used
before. Rather than recursion based on elements of a sequence,
we're calling our ``buildPlugboard`` function in its own definition,
but with a bigger value for ``i`` each time around. Finally it ends
when ``i`` is greater than or equal to the number of elements in our
``swaps`` sequence. Cryptol's built-in ``width`` function tells us how
many elements there are in ``swaps``.

Now that our Enigma's state is more than just its rotors, let's create
a *record* that keeps it all together. A record is like a tuple, but with
named elements. Here's its declaration, and a helper function that
creates one from its parts.

.. literalinclude:: cryptol/threeRotors.cry
   :language: cryptol
   :start-after: BeginEnigmaStateDef
   :end-before: EndEnigmaStateDef

Finishing up: adjustable rings and initial rotor positions
----------------------------------------------------------

Install the rings on your cardboard Enigma. Looking at what they do,
it's really just adjusting where the pins are at the start.
Implementing them is simple, it's just a matter of rotating the rotor
for each ring position.

Similarly, the configuration of the Enigma correspending to a *key*
consists of rotating the rotor so that the key's letter is in position
0.

Decoding some real Enigma messages
----------------------------------
