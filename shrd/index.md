# SHRD: Improving Spatial Locality in Flash Storage Accesses by Sequentializing in Host and Randomizing in Device


I really liked the paper. The idea is neat. The paper is also well written and 
the design and the new idea is clearly presented. A very good paper to read and learn from
on how to write good papers.  

# Summary
The key idea is to convert random small writes into large sequential writes at the device driver. 
This reduces data fragmentation and also reduces request handling overhead for the SSD, as 
it can combine multiple small write requests into fewer large write requests. 

![](../shrd-arch.png)
![](../shrd-example.png)

The SSD reserves a special LBA range, called random write log buffer (RWLB),  
whose logical addresses can be used as temporary LBAs for small random write requests. 
When the device driver receives small random write requests,
they are translated into large sequential write requests,
using temporary LBAs from RWLB. For example, 
write requests to four random pages can be converted into 
one 4-page sequential write request into the RWLB area. 
The device driver then only needs to send the resulting large write request
to the SSD, instead of 4 small write requests. 

Two new write requests are added to send data and metadata for sequentialized write requests
to the SSD: twrite_data() and twrite_header().
twrite_data() sends data pages to the SSD. 
twrite_header() sends the orignal/actual LBA to the SSD, which will be stored 
in the OOB area for mapping table recovery in crash.
A read redirect table is maintained in the host DRAM, to redirect read requests
for sequentialized write requests, to read from the RWLB area, instead of their
original LBAs. 
When the RWLB space is exausted, a remap operation is issued to the SSD, 
to restore the mapping entries for sequentialized pages to their original LBAs.
No data copying is needed during this remap process.   



