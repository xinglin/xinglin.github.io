# AWS re:Invent 2018 tidbits


# Amazon RDS PostgreSQL 
 * 2 flavors.
    * Open-source PostgreSQL on top of EBS
    * Aurora PostgreSQL on top of Aurora Storage (better performance)   
 * Supports Postgre 9.6/10
 * Aurora Postgres supports one RW master node and many read replicas. A read replica will be promoted to the master node when the original master node fails. 
 * Aurora Postgres supports fast clones. Only need to pay for the storage of changed data.  
 * Will support logical replication by introducing the logical decoding plugin which converts physical changes to SQL statements. The logical decoding plugin can also be used as the event source for publish/subscribe. 
 * Aurora Postgres bypasses filesystem page cache and manages its memory directly (more memory as the shared buffer). Aurora Postgres supports survivable cache: the shared buffer will survive postgres process crashes. This enable faster performance recovery.  

# S3: Simple Storage Service 
 * AWS S3 team: ~1000 engineers
 * When a bucket is created, it will be assigned one initial partition. Each partition can support up to 3,500 PUTs or 5,500 GETs. This means you either get 3,500 PUTs and 0 GET tps or 0 PUTs and 5,500 GET tps. 
    * 50% PUT and 50% GET = (0.5 * 3,500) + (0.5 * 5,500) = 4,500 tps combined
    * 30% PUT and 70% GET = (0.3 * 3,500) + (0.7 * 5,500) = 4,900 tps combined. 
 * In the backend, they detect access load on a partition. When a partition becomes hot, they will split the partition based on object prefix. They do the re-partition only when the load is consistently high for at least half an hour. They do not support auto scale-down for now.  

# ML frameworks
 * MXnet: efficient distributed training; portability; efficient inference; inference on edge
 * Gluon: easy coding; easy debugging; toolkits for rapid prototyping
 * SageMaker: end-2-end platform; zero setup; distributed training; AB/testing; scalable endpoints; automatic model tuning

# DynamoDB Deep Dive
 * why choose NoSQL?
    * SQL: optimized for storage
        * Normalized/relational
        * Ad hoc queries
        * Scale vertically
        * Good for OLAP
    * NoSQL: optimized for compute
        * Denormalized/hierarchical
        * Instantiated views
        * Scale horizontally
        * Built for OLTP at scale
 * Partition key
    * Large number of distinct values
    * Items are uniformly requested and randomly distributed
 * Sort key
    * Efficient/selective patterns
    * Leverage range queries
 * 1 application service = 1 table
    * Reduce round trips
    * Simplify access patterns
    * Inside Amazon, they have managed to support ~40 different queries with one table. 

# Lambda/Serverless
 * One function per one microVM
 * No orchestration code in a Lambda function. Retry/error handling should be done using Step functions. 
 * Should avoid monorepo for functions: one repo per function. This simplifies permissions and keeps each function small (minimize dependency for each function and use less memory). 
 * Four stages in the function lifecycle
    1. Download your code (cold start)
    2. Start new execution environment (cold start)
    3. Bootstrap the runtime (Partial cold start)
    4. Start your code (Warm start) 
 * Use AWS X-Ray for instrumenting Lambda functions
 * CPU is proportional to memory size: increase the memory size will make a function run much faster and in some cases, could save the cost as well.
    * < 1.8 GB: single core
    * > 1.8 GB: 2 cores

# AWS RedShift
 * Column data is stored as 1 MB immutable blocks
 * Blocks are encoded with 1 of 12 encodings.
 * Zone map contains min/max values for each block. Used to eliminate unnessary IOs.

# Misc
* [choose boring technology][boring-tech]
* [How We Massively Reduced Our AWS Lambda Bill with Go][go-bill]

[boring-tech]: http://mcfunley.com/choose-boring-technology
[go-bill]: https://runbook.cloud/blog/posts/how-we-massively-reduced-our-aws-lambda-bill-with-go/
