So this company had devices which could connect to a computer via USB to perform firmware updates and also change settings and add personalization.

Maybe there used to be a native Windows app that managed it, but I've never seen it.

They wanted to have browser UI, and also the personalization and discovering and fetching new firmware required cloud-side integration anyway.

So someone designed a good architecture, with an agent application that talks to the device via USB and has an HTTP server
(using Mongoose) and it listens on some port on localhost and the browser connects to it to talk to it.

The principle was fine.

They paid some contractor company to do the driver and the agent app.

They paid another contractor company to do the website and the javascript code that talks to the agent
and to perform settings changes and personalization updates and manages the firmware update process and
stores stuff for you in your personal account and links devices to user accounts.

It was long enough ago that "social" was a leading industry buzzword.

Unfortunately, it seems that some hardware engineers who were in charge at that company that made devices were very contemptuous
of software, especially non-embedded software, and underestimated budget and time for
the project by a lot (something like three years into the project,
the PM told me it was supposed to be a four month project).

I don't know whether technical issues were because of a lack of time, hiring the cheapest or just bad luck at the HR roulette.

They switched contractors, I was brought in to try to fix the website side after the customer was very unhappy.

I couldn't rewrite the whole thing, and it was built on shaky ground, bad php code, bad js code.

For some reason they tried to compartmentalize the thing, so I was not given access to information about the device
and I was not given the source code for the agent software with the web server interface.

Working with it was an adventure. Here are some examples:

1

It had libcurl inside, but they handcoded the query string parsing and
it would crash if you constructed the query string with the fields in an unexpected order.

2

It had a feature of "here is a URL, fetch binary data from there and write into flash at address X".

It used libcurl for that. I did not test how well it did TLS cert verification (if at all)
but the bug they actually encountered was that it did not check the http status code.

So sometimes the server that produced those binary blobs that should be flashed into the device
produced a php exception, and the html output with the php exception got written into device flash.

When the firmware tried to parse the data in the flash, it was not very robust
so it crashed and the device became bricked. It was returned, analyzed by embedded devs
and I was asked to fix the server to not do that.

One of the fixes was to make sure the php saved the blob to a file that was served by nginx,
and if you requested a blob to be produced and then downloaded the blob multiple times,
you always got the same blob.

The JS code would do an ajax request to download the blob to verify it looks sane before asking
the agent to download it and flash it into the device.

3

The interface exposed by the agent was very bad.

For commands to read short values from the device, you could not reset the "last command's status",
and it was not reset automatically by "read status",
so for example you asked for an operation, then you performed "read status", the status was "completed",
then you asked for the second operation, but the status was still "completed"
because the operation has not been performed yet,
and then the status changed
either to "error" or "completed" again, and if you saw "completed" you could not know
whether it's _still_ "completed" or _again_ "completed".

I inserted superfluous operations of a different kind in between
every two operations of the same kind, to force the status to change.

It did not improve user-visible latency.

4

For a new feature, my JS code issued a larger read-from-flash operation than before,
and the agent promptly crashed. Reliably!

Since I have been working there for a while they trusted me more 
and I got access to the source code of the agent (also the contractor that built it was no more)
and I discovered the reason it crashed was 
a classic buffer overflow in a base64 encoder implemented in C.

Basically, you used JSONP to request a read of some length from some address,
and then you did JSONP to poll it until it said the read was done and the result ready,
and then you did JSONP to get the result and got BASE64 of the data, which was parsed in JS,
some AJAX requests performed to the server, some UI stuff, yada yada.

Anyway, they used this code from [libb64](http://sourceforge.net/projects/libb64),
you can see it [here](https://github.com/BuLogics/libb64/blob/c1e3323498e1b5512e509716c5720029853846bc/src/cencode.c)
on a github mirror of the sourceforge project I found for this blog post.

It is a dumb C function to produce base64 encoded parts for MIME email.

It thinks there should be a line break after 72 chars.

The C API does not pass the available space in the output buffer to the function,
so the code cannot check whether it's overflowing the output buffer even if it wanted to.

This is terrible C API design, basically unusable, would not pass code review anywhere.

You see where this is going.

They calculated the space required as `((input / 3) * 4)`, without taking the line breaks into account.

For a big enough chunk, with enough line breaks, something important got overwritten and it crashed.

Could it be exploited from the internet? Probably. Did it run as root? Yes.


I am not proud of the state of that project when I left.

I fixed bugs, sure, but it's not a triumph.

AFAIK, they still don't do the right thing
(see for example [mkcert](https://blog.filippo.io/mkcert-valid-https-certificates-for-localhost/)) w.r.t local TLS cert.

I think the lessons on that project were in the sphere of project management, not software.

A funny story from that project was the time they posted a wanted ad for "webmaster for site running on a PHP CMS"
or something and then were super surprised that none of the people who turned up could perform operations
on a bitmap (like, "here is an array of integers sized x bits each, write code to determine whether bit 20 is on or off, write code to turn bit 20 off", etc).

I tried to explain that their wanted ad invited people who could not do that and precluded the people who could do that from answering it.

Another funny story is how I had to explain unicode encodings to the embedded software team lead, because he had never worked with UTF-8 or UTF-16 and vaguely thought "Unicode" meant "UCS-2". This happened because of a bug.

They were also surprised that it was possible to unpack that bitmap into fields in an SQL view for reporting.

Once it turned out that they wrote software that works on the manufacturing line to issue
unique serial numbers and bluetooth MACs to devices, and when they created a new product line
they copied over the software and just started it with an offset, without a limit.
So some time later the one with the lower offset caught up and produced devices with the same serials as other already produced devices.
They caught it after those devices reached customers and used the website to change their serial numbers during an update.
I explained how a ticketing system should work, issuing non-overlapping batches of numbers to production lines.
They seemed surprised by the idea.
