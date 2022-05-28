# Flush data to stable storage in C


When writing to a FILE object in C, it turns out there are 
multiple steps one needs to take in order to flush data
from an application buffer to the stable storage (disk). 

Assume we have a FILE object pointer, called fp. 
After the fwrite(fp) call, one first needs to call fflush(fp),
to flush the application buffer to the OS kernel page cache. 
Since the data still sits in the OS kernel space, it won't survive
on a power lose. 
To flush the OS page cache to the stable storage which will survive
a power lose, we need to further call fsync(fileno(fp)).
fsync() will make sure to flush the data in the kernel to the storage
media. 

```
# to force flushing data to the storage media after fwrite()

assert( fflush(fp)==0 );
assert( fsync(fileno(fp))==0 );
```

Reference:  
* [Ensuring data reaches disk][lwn]

[lwn]: https://lwn.net/Articles/457667/

