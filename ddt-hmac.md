One day I was looking at the source code of django-debug-toolbar,
 a useful plugin that (among other things) renders a panel that shows
 the database queries that were performed by the app server to render the page.

Of course this doesn't help with API endpoints,
and of course it only works with DEBUG=True, but it's very useful.

Even though the documentation tells you to never install it in production, people do.

I don't, because I know it was not meant to be secure.

The panel shows a list of database queries that ran,
with how long it took to run next to each one.

Also next to each query are two buttons,
one to run and display results and the other to display explain query plan.

That panel is not supposed to allow you one to run arbitrary database queries,
only those that have been run to process the request to render the page.

This requirement is not enough to protect you from harm.

Imagine performing a legitimate action (e.g. buying something from an online shop) that,
as part of the operation, performs `UPDATE user_store_credit SET value = value + 10 WHERE user_id = 123`.

What if you're able to selectively execute some of the database operations again?

If limited to only SELECT, a login form loads the password hash matching the username,
exposing all user accounts to offline password hash cracking.

Anyway, there is still the goal of allowing only queries that the application displayed
on the panel to be submitted to "please run this sql" endpoint.

A simple and secure solution is to store the queries linked to
session and let you run any of them from your own session.

To avoid filling up the database, expire them after some time.

This would be very inefficient.

Cryptography to the rescue!

Sign the query on the server, require submitting queries to be run with signatures, validate the signature.

Because the signer and the verifier are the same, symmetric cryptography is sufficient.

Simple and elegant.

Except that django debug toolbar used `hash(key || message)` as a MAC.

I've submitted [a patch](https://github.com/jazzband/django-debug-toolbar/commit/d27d8174c9aea8858c492c357e2b46559306364b) to use HMAC.

The code is still probably not very secure, so don't run it anywhere except localhost while debugging.
