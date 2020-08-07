An IOT project in which the device was a bus and a payload.

The bus was a computer and the payload was a smaller computer.

The bus was provided by one contractor and the payload was provided by another contractor.

The bus needed to ping its cloud servers regularly, and the payload also needed to ping its own cloud servers regularly.

The payload only needed to send data, not receive data, so one way communication was possible.

An idea was raised that in order to improve battery life, the payload would not
connect to its own cloud servers using a separate TLS connection but instead would
rely on the bus to forward the data, through the bus's cloud server to the payload's cloud server.

It was important to not expose the data to the company that provided the bus,
so the bus' firmware was supposed to be reviewed to make sure it doesn't do anything to the payload or the data,
and the server side code could not be reviewed because it could be surreptitiously replace and how would you know?

I explained how to do ephemeral-static asymmetric encryption using libsodium to encrypt
the payload's message on the device, let the bus pass it to the bus' cloud server,
and then accept it on the payload's cloud server from the bus' cloud server and there decrypt it.

I explained how, because this is one way communication,
unlike two way communication like TLS this cannot not provide forward secrecy.

This optimization was deemed not worth it, and two TLS connections were used.
