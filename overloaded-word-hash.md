People overload meanings of words and this causes confusion, news at 11.

Naming things is famously one of the hard things in computer science, but still.

This time I'm complaining about the word "hash" or "hash function".

First, a function you would use to protect against accidental data change is not a hash function at all. See CRC or Luhn algorithm.

The properties you want in a function used has tables are speed and
distributing inputs into buckets mostly evenly.

[xxHash](https://cyan4973.github.io/xxHash/) is the new hotness.

I once wrote code to pick the most "selective" bits from a set of 32bit ints,
to create a fast frozen hash table of those ints. Beat more popular hash functions for my use case.

If you're defending the hash table from algorithmic complexity attacks, you need
something more like (but not exactly) a cryptographic hash function (SipHash is popular for this use case).

A function you would use for data deduplication needs to be cryptographic if your storage relies on no collisions.
If you handle collisions, a function for hash tables would do, since you're storing a hash table.

If you're hashing data to anonymize it, you're probably doing it wrong and talk to a cryptographer.

A hash function used inside HMAC to create a MAC is not the same as the hash function you use
in digital signatures that protect certificates and authenticated DH key exchanges.

Blake3 seems to be the new hotness. Blake2 was good too. SHA3 is slow, KangarooTwelve is not standardized.
Truncated SHA2-512 to 256 is both faster and more secure than SHA-256,
because 64 bits operations and defense against length extension attack.

SHA-2 is [likely not going to be broken any time soon](https://twitter.com/veorq/status/834872988445065218).

SHA1 and even MD5 are secure in HMAC, but don't use them anyway because they're a code smell.
But TLS did not need to create CBC-SHA2 cipher suites after all.

Lastly, password storage systems can be built out of hash functions, like PBKDF2, but
they have completely different requirements from hash functions.

Maybe you need a PRF or a VRF.

The hash part of the URL is named after another name for the typographic sign.
