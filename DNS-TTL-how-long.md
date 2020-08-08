An IOT device has DNS domain name of its cloud server flashed into it
at manufacturing time, and later it uses DNS to get an IP address and
connect to the server.

We even made sure there is no IP address hardcoded in the firmware by
creating a new server with new IP and new domain and writing that into
a new device and it uploaded to the new server.

So we were quite surprised when after a server changed its IP address
the DNS propagation seemed to take a very long time.

Such a long time that it was soon obvious that devices in the field
were broken.

Turned out that FreeRTOS DNS client caches DNS results forever. Infinite TTL.

So those devices kept trying to connect to the old IP.

This was probably a performance optimization.

I told the customer that you can either use DNS and then cache according to TTL,
or use DNS and cache for 24h regardless of TTL,
or use a fixed IP you own (with your own ASN) that you can have hosted by
some provider, but you can't use DNS and then cache forever - that's just a recipe for bugs.

The firmware was fixed to use DNS properly.
