An IOT project. Device periodically connects to the server to upload collected data.

The customer wants an API that given a device id, goes over the last month of uploaded data stored on S3
and extracts timestamps out of the data and sends the list of timestamps, sorted, back.

Sounds slow and expensive.

Trying to get more info.

Well, don't need the timestamps, only the ranges to detect gaps.

That is a lot less data.

Can store ranges of uploaded data in the database when processing each upload.

Process the timestamps into sorted ranges, split ranges at midnight,
store every value as delta from the previous value,
store every value in [LEB128](https://en.wikipedia.org/wiki/LEB128#Unsigned_LEB128),
store a single day's worth of per-device ranges in a single database row, delete rows older than 30 days.

The code behind the API performs the reverse and returns a nice JSON with int unix timestamps.

I calculated the worst case is about 200 bytes per device per day of storage, which is fine.

In the end, the API was used to draw an UI indicator that was only interested in gaps of X hours or more,
so gaps smaller than that were elided (neighbouring ranges merged) in the API.

The storage is still with 1 second granularity, but if the number of devices grows a lot, this will be improved.

After a few times when that customer asked for something very generic and then used it in a very suboptimal way,
I became more cautious and started asking more details about intended use.
