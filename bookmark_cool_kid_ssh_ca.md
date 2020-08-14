Bookmark

How do the cool kids use SSH?

Do they rely on TOFU for host authentication?

Do they manually edit ~/.ssh/authorized_keys for client auth?

Do they run a corporate CA and sign people's ubikeys
using the CA to grant them access?

No, they run a CA that requires login with SSO
with U2F and checks the user account is not locked
and then issues a short lived ssh cert that lets you login
for 2 minutes (the connection is not terminated when the cert expires).

links:

https://github.com/gravitational/teleport

https://github.com/Netflix/bless
