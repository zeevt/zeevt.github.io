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

The interface exposed by the agent was very bad.

You could not reset the status, and it was not reset automatically by "read status",
so for example you asked for an operation, then you performed "read status", the status was "completed",
then you asked for the second operation, but the status was still "completed", and then it changed
either to "error" or "completed" again, and if you saw "completed" you could not know
whether it's _still_ "completed" or _again_ "completed".

I inserted superflous operations of a different sort in between every two operations of the same sort, to change the status.

It did not improve user visible latency.

After a while they trusted me more and I discovered one of the reasons it crashed when
you tried to read a larger chunk of data from the device (for a new feature) was because
of a classic buffer overflow in a C language base64 encoder.

Basically, you used JSONP to request a read of some length from some address,
and then you did JSONP to poll it until it said the read was done and the result ready,
and then you did JSONP to get the result and got BASE64 of the data, which was parsed in JS,
some AJAX requests performed to the server, some UI stuff, yada yada.

Anyway, they used this code from http://sourceforge.net/projects/libb64,
you can see it [here](https://github.com/BuLogics/libb64/blob/c1e3323498e1b5512e509716c5720029853846bc/src/cencode.c)
on a github mirror of the sourceforge project I found.

It is a dumb C function to produce base64 encoded parts for MIME email.

It thinks there should be a line break after 72 chars.

The C API does not pass the available space in the output buffer to the function,
so the code cannot check whether it's overflowing the output buffer even if it wanted to.

You see where this is going.

They calculated the space required as `((input / 3) * 4)`, without taking account of the line breaks.

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

Once it turned out that they wrote softwatr that works on the manufacturing line to issue 
unique serial numbers and bluetooth MACs to devices, and when they created a new product line 
they copied over the software and just started it with an offset, without a limit.
So some time later the one with the lower offset caught up and produced devices with the same serials as other already produced devices.
They caught it after those devices reached customers and used the website to change their serial numbers during an update.
I explained how a ticketing system should work, issuing non-overlapping batches of numbers to production lines.
They seemed surprised by the idea.
