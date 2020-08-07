The story about accepting payments in electronic money reminded me of
another story about accepting payments.

Customer wanted their shop to use a particular credit card processing company,
because the company was well established and had some major companies as their users.

No, not PayPal. An Israeli company that makes PayPal look like a most modern,
hip, cool, easy to use, visually leasing and secure service.

From working with them, I get the impression that someone implemented the service
in the 90s, possibly even mid-90s, sold it and moved on.

The buyer is a company that keeps
turning the crank to print more money and offers customer support,
but they do not appear to have any software developers on staff.

I don't know if they have security people on staff, but I doubt it.

Besides the backoffice, API and API docs being from the 90s,
which is a quirk but not necessarily a problem,
I've had three actual issues with them:

1. They offer PayPal integration, meaning you don't need to
open a merchant account with PayPal, you open one with them,
their payment page can send the shopper to PayPal,
PayPal will send the money to send, they'll send the money to you.
I'm unclear on whether that means you would be double paying fees.

The problem is that you don't get PayPal's IPN webhook,
and if they get and verify the webhook they don't forward it.

You as a shop don't have any way to know whether the shopper has paid or not.

The shopper just ends up on your "thank you for paying" page.

If your shop uses manual processing, you can manually open the list of paid transactions
in the payment processor's system and look for the matching transaction
and approve the transaction in your shop as "paid", manually.

Maybe this lookup can be done with an API, but come on.

2. Their system sends webhooks to let the shop know that a payment actually happened for a particular transaction
(and how much was paid, though probably some of the shops that use the service don't actually check the amount).

Such webhooks need to be verified, to make sure they actually come from the payment processing company.

Usually this is done with HMAC, but HMAC was only invented in '96, so possibly their code was written before HMAC was known.

I'm only 90% joking, I think.

PayPal IPN does not have HMAC, it is verified by the shop making a request back to paypal.com to verify the webhook.

This is slow and silly and sometimes fails with HTTP 500 from paypal, but when it works, if sone over TLS, it's secure.

The credit card processing system that is the subject of this blog post does not feature a way to verify the webhook.

The security relies entirely on the URL of the webhook being secret.

So anyone with access to the web server access logs can find out the secret webhook URL and then fake payments.

The faked payments will be discovered at the end of the month, when the amount of money received from the payment processor won't match.

Or sooner, but only if the shop owner uses the payment processor's system to compare payments with the shop's system, manually.

3. As a shopper, you could type your name (or address, any form field)
as `&paid=true` (not the right key, but you get the idea) and
not pay (give bad credit card number) and the card processing server will send a
webhook with the form data, including your value, unquoted, to the shop's server.

To the shop's server, the request looks exactly like the request that is sent when the shopper paid.

There is no way to tell, no extra form fields to check.

If the shop sells electronic goods and performs immediate fulfillment (e.g. selling ebooks, songs or fonts)
then the shop has nothing to go on.

With goods that need to be shipped, the thieves will need a way to accept delivery of goods they didn't pay for,
but I guess this too can be arranged.

This last issue was serious enough that I reported it to the company and they did fix it.

But it's been there for probably 20 years.

I would not use that company again if I could help it.
