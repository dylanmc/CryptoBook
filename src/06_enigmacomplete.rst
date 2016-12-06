Completing the Enigma in Cryptol
================================

  *This chapter can be skipped the first time through, if you're
  impatient. The Enigma is really cool, though, and there are some
  neat programming techniques in here, so make sure you come back to
  this chapter later on.*

Compared to our paper Enigma (and the real thing), our Cryptol version
lacks:

#) Multiple rotors,

#) the ways the rotors step,

#) the "rings" that adjust *when* the rotors step, and

#) the plugboard, which modifies the I/O ring

Let's implement them!

Combining multiple rotors
--------------------------

First, get our your paper Enigma and set ``rotor II`` and ``rotor
III`` to ``A``. Then trace, from the right towards the left, each
letter from ``A .. Z``.

As a starting point, what I get when I did this:

.. code-block:: console

  rotorII = "AJD ..."
  rotorIII = "BDF ..."

Once you complete the rotor strings, the next job is to combine them
in the encryption function.
Stepping the rotors
-------------------

Implementing adjustable rings
------------------------------

Finishing up: the plugboard
----------------------------

Decoding real Enigma messages
-------------------------------
