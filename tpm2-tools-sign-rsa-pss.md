So this one time, a client wanted to encrypt a hard drive
 of an embedded system with a key stored in the cloud,
 and release the encryption key back to the client only if the client
 proved to have possession of a TPM, by performing a signature
 using a private key corresponding to the public key
 recorded when the system was provisioned.

So when the system is being provisioned, ask the TPM to generate a key pair.

The TPM does not have enough storage to remember the key so the key must be stored outside the TPM.

But the key must not be exported, to keep it secure.

So the TPM encrypts the new key using the key that is hardcoded into the TPM
at manufacturing time and gives you the encrypted blob to store.

Later you can load the encrypted blob back into the TPM where it will decrypt it
using its hardcoded key and then it will be able to perform operations on the key.

Sounds ok, though I was never very clear the attack that is stopped by this defense is a serious risk.

Two problems:

1. The API of the library for working with a TPM (tpm2-tss) is very unpleasant to use. The documentation is very unpleasant to read.

Solution: use the command line tool instead of the library. The flags change every release, but whatever.

2. The command line tool (tpm2-tools) does not support RSA-PSS, it only supports the old RSA PKCS#1 v1.5.

Solution: If it's good enough for everyone's TLS certificate, it's good enough for me.

Right? No. Just add RSA-PSS support to the cli tool and write
 [unit tests](https://github.com/tpm2-software/tpm2-tools/blob/master/test/integration/tests/sign.sh)
 in order to get the change upstreamed so we won't need to carry a fork forever.

So I was allowed to do that and it was a nice experience.
