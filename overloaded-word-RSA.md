Another overloaded word is "RSA".

The RSA [trapdoor permutation](https://en.wikipedia.org/wiki/Trapdoor_function),
named after its inventors.

RSA asymmetric encryption, in [PKCS#1](https://en.wikipedia.org/wiki/PKCS_1)
v1.5 and OAEP flavors.

RSA [digital signatures](./bookmark_digital_signatures_are_not_encryption.md),
in PKCS#1 v1.5 and RSA-PSS flavors.

The TLS cipher suites whose names begin with "TLS_RLS_"
and which use RSA encryption to pass a secret value
from the client to the server (for key exchange)
and use the server's ability to decrypt using
the private key matching the public key from the certificate
for server authentication. This does not provide PFS and
allows retroactive decryption of all recorded TLS traffic
using the key from the certificate, even after the certificate
is expired and an IT person thinks it's useless. These
cipher suites are not allowed in
[TLS 1.3](https://tools.ietf.org/html/rfc8446) (2018)
and were
[not allowed for use with HTTP/2](https://tools.ietf.org/html/rfc7540#appendix-A)
(2015).

[The RSA company](https://en.wikipedia.org/wiki/RSA_Security),
founded by the inventors of the RSA
trapdoor permutation / cryptosystem, now has little
to do with the inventors or cryptography.
Infamously used to sell a cryptographic software library
called BSAFE which used the NSA's backdoored RNG called
[DUAL_EC_DRBG](https://en.wikipedia.org/wiki/Dual_EC_DRBG)
and took $10M from the NSA to make that the default RNG.

The [RSA Conference](https://en.wikipedia.org/wiki/RSA_Conference),
a mostly non-technical
sales conference where lots of companies try to sell security
snake oil (and working security products), organized by
the RSA Company. Sometimes used by US government officials to
give a speed about "cyber".
