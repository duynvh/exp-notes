# Random and Sequential Access
Random access là gì?
- is the ability to access an arbitrary element of a sequence in equal time or any datum from a population of [addressable](https://en.wikipedia.org/wiki/Address_space "Address space") elements roughly as easily and efficiently as any other, no matter how many elements may be in the set. In [computer science](https://en.wikipedia.org/wiki/Computer_science "Computer science") it is typically contrasted to [sequential access](https://en.wikipedia.org/wiki/Sequential_access "Sequential access") which requires data to be retrieved in the order it was stored
- is a term used to describe a computer's ability to immediately locate and retrieve data from a storage device.
- Random access memory (RAM) where data can be accessed in any order

Sequential access là gì?
- sequential access memory (SAM) is **a class of data storage devices that read stored data in a sequence**.
- With sequential access, the device must move through all information up to the location where it is attempting to read or write.
- While sequential access memory is read in sequence, arbitrary locations can still be accessed by "seeking" to the requested location. This operation, however, is often relatively inefficient (see [seek time](https://en.wikipedia.org/wiki/Seek_time "Seek time"), [rotational latency](https://en.wikipedia.org/wiki/Rotational_latency "Rotational latency")).

RAM thì dùng loại nào?
Disk dùng loại nào?

Cái nào tối ưu hơn?

“Comparing random versus sequential operations is one way of assessing application efficiency in terms of disk use. Accessing data sequentially is much faster than accessing it randomly because of the way in which the disk hardware works. The seek operation, which occurs when the disk head positions itself at the right disk cylinder to access data requested, takes more time than any other part of the I/O process. Because reading randomly involves a higher number of seek operations than does sequential reading, random reads deliver a lower rate of throughput. The same is true for random writing.”

https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-2000-server/cc938619(v=technet.10)?redirectedfrom=MSDN