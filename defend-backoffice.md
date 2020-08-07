A customer had a system that managed phishing campaign training for employees of organizations.

An organization would hire him, give him a spreadsheet file with names and email addresses,
he would customize emails and landing pages (some with forms), load them into the system
and use the system to schedule emailing out the phishing emails with links to
landing pages.

Either the landing page immediately told the employee they have fallen for a phishing attack by their own IT/security,
or it said that only after the employee cliked "submit" on a form on the page.

The customer said that his customers were very worried about someone hacking into
the system's backoffice and extracting the email addresses and even who among the
employees fell for the phishing attack.

He wanted help securing the system.

We did two things.

1. A cosmetic thing that was done in order to be able to say that the
email addresses were encrypted in the database.
I encrypted the email addresses using deterministic encryption (AES-CBC-HMAC-SHA256 with fixed IV).

The determinism was needed in order to allow lookup by exact match in the backoffice.

Today I would have used the 
[blind indexing technique from Paragonie](https://paragonie.com/blog/2017/05/building-searchable-encrypted-databases-with-php-and-sql#solution-literal-search)
, but I didn't know about it at the time.

2. A strong defense.

I suggested, and the customer agreed, to convert the system to be a static site generator and log parser.

The landing pages were generated as static pages served by nginx.

They got deployed by the backoffice system to be served when the phishing campaign was launched,
ahead of phishing emails being sent.

nginx access logs were periodically rotated and ingested by the backoffice (over ssh) to determine
which landing pages got "hit" and whether the form was submitted (the form did not submit any of the data,
only the names of fields that were not empty).

The backoffice system itself stayed safe behind a VPN.

This converted a system that could be attacked easily into a system that is hard to attack.

Personally, I believe training people to not click links in emails is counterproductive
and systems should just be secure even if people click links in emails. For example, using U2F.

But that is a topic for another time.
