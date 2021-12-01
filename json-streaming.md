There is a process that packages some data into a file and a process to load that data from the file.

It used to run fine but when it was used with twice as much data it used so much memory that I was asked to fix it.

The original code was not written in a streaming fashion,
rather everything was loaded into memory and processed that way.

After I've optimized the easy parts, I've reached the actual problem.
I've noticed that issue before, but at the time it was deemed "inefficient" but not a problem.

Turns out had we measured how much of a problem it is instead of assuming,
we probably would have decided to fix it earlier.

If you have about 30 text (UTF-8) files, some of which are small while others are up to 200MB in size,
and you need to package them into a single file, what would you use? zip? tar? tar.gz? tar.zst?
Well, that system used a JSON map, with file name as key. Why? No good reason.
Because of the need to coordinate file format versioning and deprecation when upgrading to fix it,
we didn't "just" fix it.

Anyway, as I wrote, after optimizing the easy parts I had something like this to create the file:

```python
import base64, json, io, os
import zstandard

def gen_v(size_mb):
    compressed = io.BytesIO()
    cctx = zstandard.ZstdCompressor(level=1)
    with cctx.stream_writer(compressed, closefd=False) as w:
        for i in range(size_mb * 1024):
            w.write(base64.b64encode(os.urandom(15)))
            w.write(base64.b64encode(os.urandom(3)) * 251)
    compressed.seek(0)
    return compressed

def gen_data():
    data = {}
    for i in range(1, 6):
        data['big' + str(i)] = gen_v(500)
    for i in range(1, 21):
        data['small' + str(i)] = gen_v(1)
    return data

def write_data(data):
    for k, compressed in data.items():
        dctx = zstandard.ZstdDecompressor()
        decompressed = io.BytesIO()
        dctx.copy_stream(compressed, decompressed)
        decompressed.seek(0)
        data[k] = decompressed.getvalue().decode('UTF-8')
    cctx = zstandard.ZstdCompressor(level=1)
    with open('file.json.zst', 'wb') as fb:
        with cctx.stream_writer(fb, closefd=False) as w:
            with io.TextIOWrapper(w, encoding='UTF-8') as ft:
                json.dump(data, ft)

write_data(gen_data())
```

And something like this to load it:

```python
import base64, json, io, os
import zstandard

def read_data():
    dctx = zstandard.ZstdDecompressor()
    with open('file.json.zst', 'rb') as f:
        with dctx.stream_reader(f, closefd=False) as r:
            data = json.load(r)
    for k, v in data.items():
        compressed = io.BytesIO()
        cctx = zstandard.ZstdCompressor(level=1)
        with cctx.stream_writer(compressed, closefd=False) as w:
            w.write(v.encode('UTF-8'))
        compressed.seek(0)
        data[k] = compressed
    return data

data = read_data()
```

This generates a ~50MB file containing ~500MB of junk, but it uses about 3GB of RAM
to save the file and about the same to load it.

It is clear that it is possible to generate the file using less than 1MB of RAM and
loading the file into a compressed buffer in memory should only require
the space for the compressed buffer and maybe 1MB for overhead.

But the JSON library in python's standard library uses a lot of memory, because of the DOM API.

A nice JSON library doing `write_data` could have accepted an output stream writer `w` 
and a dict that has [stream readers](https://python-zstandard.readthedocs.io/en/latest/decompressor.html#zstandard.ZstdDecompressor.stream_reader) for values,
and decompressed the input in little chunks, json-escaped the string chunks and compressed them into the output.

When doing `read_data()`, the input should be stream reader `r` and a factory that creates stream writers backed by io.BytesIO.
The code can read chunks from r, decode keys, creates stream writers for each value using the factory,
read and json-unescape the string values in chunks, write the unescaped chunks into the stream writers,
then flush the close the stream writers and put the `bytes` returned by `BytesIO.getvalue()` into the result `dict`.
I dind't find a nice abstraction for this so I made a less nice one,
and it does support using real files instead of comrepessed buffers in memory.

In a cursory search, I didn't find a nice SAX or StAX JSON library for python, so I solved my problem
by writing specialized code for just this case, not a general purpose JSON library.

To generate the JSON, my code takes a `dict` with values that have type `IO[bytes]`
(they have a `read` method that returns bytes), writes `{"key1": "` and then reads 4KB
from the input stream, JSON escapes it, and writes it to the output stream.
Then it writes `", "key2": "` and handles the next value.

If you look at https://github.com/python/cpython/blob/3.10/Lib/json/encoder.py, you'll see
that `c_encode_basestring` which is the fast code that does the escaping, is not the exact thing we want.
We want something like "json-escape this chunk of UTF-8 bytes".
Instead we have something that adds doublequotes and works on unicode strings, not UTF-8 bytes arrays.
Removing the doublequotes is simple.
I considered writing my own in C and calling it using cffi but didn't want to add a compilation stage to the package.
I considered using regex to implement the escaping. Unfortunately I don't remember why I rejected this idea.
What I ended up doing was reading 4KB chunk, finding the last byte that begins a new code point,
cutting that off to prepend before the next chunk, decoding to Unicode string, json-escaping using `encode_basestring`,
encoding back to UTF-8 bytes, and writing that to output stream.

Decoding was more complicated.

Looking at https://github.com/python/cpython/blob/3.10/Lib/json/decoder.py,
I did not find a general solution of where to cut the input into chunks to feed the chunks into `c_scanstring`.

I ended up adapting the code in `py_scanstring` and `_decode_uXXXX` to use
a buffered reader that can peek and consume bytes from a stream.
It doesn't need to convert from bytes to Unicode strings and back,
except for the case when it reads a code point outside the BMP encoding in `ensure_ascii=True` mode,
which doesn't happen often in the data.

I wrote a lot of tests for all that ensuring the behavior (especialy in case of errors) matches the stdlib json library,
it's used in production and the performance cost is acceptable.

It was the most fun task I've done on the clock in a while.

I have been considering writing an open source library to handle the general case of this problem.
