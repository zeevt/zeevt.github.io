A long time ago a customer was using actual FTP to transfer
a nightly backup of production database to separate hosting.

With the server that performs the backup having the password to delete backups.

With the password being sent in the clear over the internet.

With the backup file stored in the clear on a sketchy server.

I insisted on using FTPS and encrypting the backup file.

When deciding what to use for encryption we settled on gpg because
it was reputable (turns out it was famous among non-specialists while
actual infosec professionals thought it was shit).

I used some command line arguments to force it to use better algorithms,
since the defaults were shit (less secure and slower than what I chose).

We used it for a while until I've had enough and we switched to AWS S3.

Anyway, over the years I've seen lots of people asking what should they
use for encryption and signing.

The answer is [age](https://github.com/FiloSottile/age) and
[minisign](https://github.com/jedisct1/minisign).

Don't use gpg. Don't try to encrypt email.

Links:

https://blog.filippo.io/giving-up-on-long-term-pgp/

https://gist.github.com/grugq/03167bed45e774551155

https://grugq.github.io/presentations/COMSEC%20beyond%20encryption.pdf

https://blog.cryptographyengineering.com/2014/08/13/whats-matter-with-pgp/

https://moxie.org/blog/gpg-and-me/

https://latacora.micro.blog/2019/07/16/the-pgp-problem.html
