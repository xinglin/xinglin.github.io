# SPDK on Samsung 960 EVO NVMe SSD


Today, I started to play with Intel SPDK with a 250GB Samsung 960 EVO NVMe SSD. 
I was thinking that using SPDK might lead to better performance, than through traditional block device interface. 
However, using fio with 4KiB random read requests at the queue depth equal to 1, as it turned out, the difference is rather small.

| Metric | SPDK    | /dev/nvme0n1 |
|--------|---------|-------------|
| IOPS   | 12.8 K  | 11.6 K      |
| slat   | 0.18 usec | --        |
| clat   | 77.73 usec| 84.19 usec|
| lat    | 77.91 usec| 84.32 usec|

slat, clat and lat are average submission latency, submission to IO completion latency and total IO latency. As we can see, this device still have a quite high read latency. The latest Intel Optane DC P4800X SSDs can achieve 10 us read latency, with 550 K IOPS. For this Samsung device, software overhead seems still fine. It would be really interesting to test out for the Intel Optane PC4800X SSDs though.

## Tips
*  Dependencies to install, in order to compile SPDK in debian 8  
        ``sudo apt-get install libnuma-dev uuid-dev libaio-dev libcunit1-dev``  

