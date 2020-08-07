One time I wanted to know which compression tool and setting I should use for a database dump in a particular project.

I wrote a script to loop over the options, measure compression file size,
time it took to compress and time it took to decompress.

I wrote another script to draw that data on [a graph](/compress_pareto.html) and automatically add a pareto frontier
to make it obvious which options were never the right answer for any speed-size tradeoff.

The graph lets you hide some compression tools from the graph to declutter it.

The graph shows bz2 and ppmd (available in p7zip-full) being better than expected at compression ratio,
because the db dump had a lot of natural language text in it.

I ended up using zstd, mostly for reason of excellent decompression speed.

These days Yann Collet's zstd is my default general purpose compression tool for everything.

The only thing better, AFAIK, are RAD Oodle compressors, and they are (1) proprietary and (2) are not optimized for compression speed.

I do recommend Charles Bloom's blog though (
[example1](https://cbloomrants.blogspot.com/2018/06/zstd-is-faster-than-leviathan.html),
[example2](https://cbloomrants.blogspot.com/2018/01/the-natural-lambda.html)).

Years later, different project.

- "Where can I get a db dump of dev to use on my local env?"
- "Here is a a file on S3".
- File takes literal 3 hours to download. File compresses to 7.6% of its size using zstd -12. (╯°□°）╯︵ ┻━┻

