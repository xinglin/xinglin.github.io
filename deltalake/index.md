# Delta Lake: High-Performance ACID Table Storage over Cloud Object Stores


Read/write performance for cloud object store.  

> Each read operation usually incurs at least 5–10 ms of base latency, and can then read data at roughly 50–100 MB/s, so an operation needs to read at least several hundred kilobytes to achieve at least half the peak throughput for sequential reads, and multiple megabytes to approach the peak throughput.

> The VM types most frequently used for analytics on AWS have at least 10 Gbps network bandwidth, so they need to run 8–10 reads in parallel to fully utilize this bandwidth.

> Google Cloud Storage and Azure Blob Store support atomic put-if-absent operations, so we use those.

> Delta Lake is deployed at thousands of Databricks customers that process exabytes of data per day, with the largest instances managing exabyte-scale datasets and billions of objects.

> although many systems support reading and writing to cloud object stores, achieving performant and mutable table storage over these systems is challenging, making it difficult to implement data warehousing capabilities over them. Unlike distributed filesystems such as HDFS [5], or custom storage engines in a DBMS, most cloud object stores are merely key-value stores, with no cross-key consistency guarantees. Their performance characteristics also differ greatly from distributed filesystems and require special care.

> Parquet files include footers with min/max statistics that can be used to skip reading them in selective queries. Reading such a footer on HDFS might take a few milliseconds, but the latency of cloud object stores is so much higher that these data skipping checks can take longer than the actual query.

> in the first few years of Databricks’ cloud service (2014–2016), around half the support escalations we received were due to data corruption, consistency or performance issues due to cloud storage strategies (e.g., undoing the effect of a crashed update job, or improving the performance of a query that reads tens of thousands of objects).

> The core idea of Delta Lake is simple: we maintain information about which objects are part of a Delta table in an ACID manner, using a write-ahead log that is itself stored in the cloud object store.

> enable a “lakehouse” paradigm that combines the key features of data warehouses and data lakes: standard DBMS management functions usable directly against low-cost object stores.

> cloud object stores do not provide cheap renames of objects or of “directories”.

> S3’s LIST only returns up to 1000 keys per call, and each call takes tens to hundreds of milliseconds, so it can take minutes to list a dataset with millions of objects using a sequential implementation.

> Updating objects usually requires rewriting the whole object at once. 

> Other cloud object stores offer stronger guarantees [31], but still lack atomic operations across multiple keys.


