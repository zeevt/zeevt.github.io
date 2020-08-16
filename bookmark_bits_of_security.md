TL;DR: use [keylength.com](https://www.keylength.com/)

Ok, so what is up with the number of bits of security of various cryptographic things?

Cipher and MAC tags: their length in bits is their "bits of security",
because the best attack is to try all values.
HMAC can be truncated to 128 bits to get 128 bits of security.

RSA modulus n, RSA public key and private key size (close enough),
RSA signature size, RSA ciphertext size,
the field size of DH over finite field of integers modulo a big safe prime,
the size of public keys in FFDHE:
large because of "index calculus",
attacks have gotten better such that the security in bits has gotten lower.
Attacked using [GNFS](https://en.wikipedia.org/wiki/General_number_field_sieve).
Due to the [complexity of GNFS](https://crypto.stackexchange.com/a/8692/24949),
2048 bits give about 112 bits of security, 3072 bits give about 128 and
15360 bits are required for 256 bits of security.
The number of bits being large makes calculations slow and
the attacks keep improving so people are worried and move to ECC.

Hash functions, the size of the private keys in FFDHE, ECC curve size
(close enough to ECC private key, public key, ECIEC ciphertext,
ECDH key shares):
half their length in bits is their "bits of security" because of
[Pollard's Rho](https://en.wikipedia.org/wiki/Pollard%27s_rho_algorithm)
and [baby-step, giant-step](https://en.wikipedia.org/wiki/Baby-step_giant-step).

ECDSA and EdDSA signatures are twice the length of the private key,
so four times the "bits of security".
