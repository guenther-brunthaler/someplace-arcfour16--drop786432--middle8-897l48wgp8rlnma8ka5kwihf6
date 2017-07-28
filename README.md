arcfour16-drop786432-middle8
============================

arcfour16-drop786432-middle8 is a minimalistic encryption
utility, implementing a 16-bit analogue of the 8-bit ARCFOUR
algorithm.

In fact it uses exactly the same algorithm, except that all
variables are 16 bits wide except of 8 bits, and the SBOX has
2**16 instead of 2**8 entries.

Usage
-----

	arcfour16-drop786432-middle8 key_file < infile > outfile

where key_file is the name of a file containing the
encryption/decryption key.

Only the first 128 KiB of key_file will be read, although it may
be shorter as long as it has an even byte size.


Mode of operation
-----------------

There is no difference between encryption and decryption: If the
input is already encrypted it will be decrypted, otherwise it
will be encrypted.


Implementation details
----------------------

Analyses have shown that the initial bytes generated by ARCFOUR
are not "random" enough, a fact which has allowed some attacks
against plain ARCFOUR.

In order to eliminate those problems, SCAN recommends that at
least the first 768 bytes should be dropped, or even 3072 bytes
to be more conservative. (The latter being 12 times the size of
the SBOX.)

arcfour16-drop786432-middle8 follows that recommendation,
dropping the first 12 times SBOX size output units generated by
the algorithm. This means the first 786432 16-bit outputs will be
dropped.

As an additional safety measure, arcfour16-drop786432-middle8
only uses the middle 8 bits of every generated 16-bit output for
actual encryption.

This also exposes less of the internal state to attackers for
analysis.


Security
--------

The internal state of ARCFOUR consists of a counter and the
SBOX contents.

The SBOX always contains a permutation of all possible integer
values that fit into an SBOX entry.

The initial contents of both SBOX and the counter are derived
from the key by a similar operation like the actual encryption.

The number of possible internal states of ARCFOUR is therefore
the number of possible counter values times the number of
possible SBOX permutations.

If the number of bits needed to represent the number of possible
internal states of ARCFOUR is considered its "maximal security",
then the following figures apply to ARCFOUR:

ARCFOUR-8: floor(log2(256 * 256!)) = 1691 bit  
ARCFOUR-16: floor(log2(65536 * 65536!)) = ??? bit

Unfortunately, my CAS software was not able to calculate the
maximal security fot ARCFOUR-16 because the numbers got too big.
Anyway, it is *huge*.

However, there is no guarantee that all possible internal states
will actually be generated durung operation: Depending on the
key, there could be loops through which the state cycles.

But this has not be an problem with ARCFOUR-8 so far, and will
even be less likely a problem with ARCFOUR-16 because of its much
larger internal state.

Most security problems with ARCFOUR have come from improper usage
or weak keys; see the next section how to avoid this.


Key file requirements
---------------------

Instead of a password, arcfour16-drop786432-middle8 requires a
binary key file which consists of an even number of bytes. This
file will be read as a sequence of 16-bit units in big-endian
byte order which represent the encryption key.

This file could also be a text file, arcfour16-drop786432-middle8
does not care as long as it consists of an even number of bytes.


Nonce and pass phrase
---------------------

ARCFOUR is a stream cipher, and it is therefore important that
the same key is never used for more than one message.

Otherwise, the comparison of two encrypted messages could be used
to decrypt all of them.

It is therefore recommended that the key consists of a password
(or pass phrase) which may be used for more than one message,
plus a nonce which is only used once but need not be secret.

The binary key could then be created from two files containing
the password and nonce as follows:

	$ cat nonce.bin password.txt nonce.bin > key_file

(Double inclusion of the nonce makes preprocessing harder for
dictionary attacks.)

Other than the password, the nonce does not need to be secret and
must be sent along with the encrypted file.

nonce.bin can either be a simple text file containing a counter
which is incremented for every message using the same key, or
consist of binary random data long enough to make it extremely
unlikely that the same nonce could ever generated again by
accident.

Like password.txt, nonce.bin should consist of an even number of
bytes to ensure the concatenated binary key is also made up of an
even number of bytes.

This is how to use a counter as file nonce.bin:

	$ read c 2> /dev/null < nonce.bin || c=0 \
	; printf '%011u\n' `expr $c + 1` | tee nonce.bin

And here is how to use a random number as nonce:

	$ dd if=/dev/urandom bs=1 count=24 2> /dev/null > nonce.bin

(Note that /dev/urandom is good enough for creating the nonce
since it is not secret and therefore does not need any entropy.)

Before using your password.txt, check that its byte size is even.
If not, edit it and add another character to the password,
padding it to an even number of bytes. The padding character has
no effect on security, so it is OK to always use the same
character and it does not need to be kept secret.

Here is to how to properly pad password.txt with a zero byte
(which is as good as any other padding value) if necessary:

	$ expr `wc -c < password.txt` % 2 = 0 > /dev/null \
	|| printf '\0' >> password.txt

-----

arcfour16-drop786432-middle8 version 2017.209.1

Copyright (c) 2017 Guenther Brunthaler. All rights reserved.

This program is free software.  
Distribution is permitted under the terms of the GPLv3.

The UUID of this project is 35944240-73a2-11e7-86f3-b827eb0d201c.
