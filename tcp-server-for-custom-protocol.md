An IOT device needs to connect to its cloud server to upload collected data.

I worked on the server side, but not from the beginning:
I was consulted at the beginning, then other people worked on it,
then I worked on it building version 2.

The IOT device could not do TLS, or at least the embedded devs believed it could not do TLS
and I didn't know enough to argue with them.

I recommended using HTTP POST to upload data, because so much infrastructure exists to handle HTTP.

Multiple HTTP requests could be used with a single connection one after another,
so the overhead is minimal - maybe a couple hundred bytes of bandwidth overhead
per communication session.

They did not listen to me and used a custom TCP protocol.

The interesting operation in this protocol is when after the handshake phase, the
IOT device sends a few thousand 1KB blocks of data to the server.

An interesting aside is that there are no ACKs in the protocol,
they relied on TCP ACKs and then
they discovered that cellular networks "accelerate" connections
by doing store and forward. The device sends data to the server,
the cellular network gets the data,
sends ACKs immediately with the source IP of the server,
and forwards the data to the server at a later time.

This helps the device upload all its data fast and shut down the radio.

But sometimes the data is not actually forwarded to the server.

The IOT device already deleted it, after receiving ACKs,
thinking it means the data reached the server.

Anyway, that was not my work, I was just present to look at wireshark
to see that data indeed did not reach the server.

Because of the custom protocol, they needed a server built from scratch.

They built one in python and the performance was atrocious.

I will write a separate blog post about the most ridiculously
badly performing code I have ever seen in my career so far.

Here, I'll just write about accepting data records,
checking and stripping the wire protocol header off them and
writing the data to disk.

The basic idea is something like this (pseudocode):

```python

def recv_full(a_socket, expected_size):
  buf = b''
  more_bytes = expected_size - len(buf)
  while more_bytes:
    more = a_socket.recv(more_bytes)
    if not more:
      raise IOError('client disconnected before sending expected data')
    buf += more
    more_bytes = expected_size - len(buf)
  return buf

disk_blocks = []
expected_blocks = handshake.expected_blocks
for block_idx in range(expected_blocks):
  wire_protocol_block = a_socket.recv_full(WIRE_PROTOCOL_BLOCK_SIZE)
  wire_protocol_header = wire_protocol_block[:WIRE_PROTOCOL_HEADER_SIZE]
  disk_block = wire_protocol_block[WIRE_PROTOCOL_HEADER_SIZE:]
  if not is_bad(wire_protocol_header):
    disk_blocks.append(disk_block)

with open(filename_from_handshake(handshake), 'wb') as f:
  f.write(b''.join(disk_blocks))
```

Turns out recv_full is inefficient because all that copying
and a_socket.recv is inefficient because it allocates memory.

Also I didn't like buffering all the uploaded data in memory and only writing to disk at the end
(because it uses too much memory and limits how many concurrent uploads the server supports),
but I also didn't want to write to disk each disk_block because that is inefficient.

What I ended up with was a single 256KB bytearray allocated per connection once,
`a_socket.recv_into` into a memoryview into it,
using offsets into the array to create memoryviews of wire_protocol_header and disk_block,
doing the work on each wire_protocol_header and then efficiently copying
(`ba[1000:2000] = ba[5000:6000]`) the disk_block data to be consecutive
in the bytearray, without the wire_protocol_header bytes between them.

When the bytearray is close to full,
a memoryview into it that contains all the received data it's holding
is written to disk and the index
pointing to the free space in the bytearray is reset back to 0.

The size was chosen as a goldielocks value and because EBS disks have 256KB writes as single IOP.

After processing the bytes written after each call to recv_into,
a state machine is updated to record the current block_idx and
how many bytes into a wire_protocol_block we are,
to resume processing after the next recv_into.

Memory allocations are eliminated, each block is copied only once.

I thought of it as basically writing C in python.

The server uses gevent, to support many concurrent clients.

Actually it's even better, it uses gunicorn, monkeypatching `def handle`
and not doing HTTP or WSGI but instead doing our custom flow.

Gunicorn code handles managing worker processes which is convenient.

In the future, something cool that wraps Linux io_uring might be used instead,
or a rewrite in Go might be worthwhile, but a custom gunicorn+gevent app was simple and performant.

The server code was benchmarked using a test client that uses multiprocessing and gevent inside each process,
to create a big load, and the server exceeded the performance requirements by a lot.

A more naive solution probably would have been fine, but because this was a rewrite
after v1 had absolutely abysmal performance, I wanted performance to be
as good as I could make it while maintaining memory safety.
