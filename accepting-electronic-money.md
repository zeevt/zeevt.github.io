A customer has an online shop.

They wanted to accept payment via bitcoin.

It was a few years ago (Dec '16), so I needed to research how to do that.

Using a service that mimicked credit card payment
(you send the shopper to pay $100,
they pay $100 via bitcoin to the processing company,
the processing company credits your account with $100 minus 3% fee,
the processing company performs a money transfer into your bank at end of business day)
was out of the question because the idea was to avoid moving the money through a bank.

Running a full node bitcoin server that participates in the p2p network
was out of the question because of the storage requirements.

Using blockchain exploration APIs to watch for transactions and blocks (and dollar exchange rate) made sense.

Couldn't have the private keys on the server, too tempting a target.

Didn't want to pre-generate payment addresses and store just the public addresses in the server,
because it's a manual process and loading the private keys into a wallet to spend them is bad UX.

Found out about BIP-0032 hierarchical deterministic wallets.

Then found Mycelium Gear.

A free service that lets you put a public seed,
generates sequential payment addresses,
watches the bitcoin network (or blockchain exploration APIs? unclear),
and sends webhooks when a transaction with the watched output is generated and confirmed.

The money does not pass through them so it's safe.

The worst thing that can happen is someone takes over the account and replaces
the payment address with his own and gets the payment for the next few transactions,
until this is noticed and the payment address is changed back.

But it's impossible to steal previously paid coins through compromising this payment method.

The service is free of charge, probably used to generate a good name for the company.

Perfect.

[A single issue](https://bitcoin.stackexchange.com/q/50801/45111)
with when they send the webhooks, fixed two weeks after my request.

Quickly implemented a solution, used google to generate QR codes,
tested using testnet3, deployed, forgot about it.

Some time later they sent an email that it's no longer free,
now they want some percent of transactions valued over X,
or they'll stop processing more payment addresses for you (remember they can't take coins from you).

The owner of the shop thinks the price is fine,
cheaper than paying me to write all the code to not need Gear.

Would I use them again?

I think I would prefer to use my own code to compute next payment address,
query multiple blockchain explorer type APIs to determine USD exchange rate,
expected transaction fees, watch for transactions and blocks, etc.

But using Gear is quicker, so maybe yes.
