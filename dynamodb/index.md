# Amazon DynamoDB: A Scalable, Predictably Performant, and Fully Managed NoSQL Database Service


> Hundreds of thousands of customers rely on DynamoDB for its fundamen- tal properties

> In 2021, during the 66-hour Amazon Prime Day shopping event, Amazon systems - including Alexa, the Amazon.com sites, and Amazon fulfill- ment centers, made trillions of API calls to DynamoDB, peak- ing at 89.2 million requests per second, while experiencing high availability with single-digit millisecond performance.

> For DynamoDB customers, consistent performance at any scale is often more important than median request service times be- cause unexpectedly high latency requests can amplify through higher layers of applications that depend on DynamoDB and lead to a bad customer experience. 

> DynamoDB offers an availability SLA of 99.99 for regular tables and 99.999 for global tables (where DynamoDB replicates across tables across multiple AWS Regions)

> a service-oriented ar- chitecture was adopted to encapsulate an application’s data behind service-level APIs that allowed sufficient decoupling to address tasks like reconfiguration without having to disrupt clients.

> Only the leader replica can serve write and strongly consis- tent read requests.

> we discovered that application workloads frequently have non-uniform access patterns both over time and over key ranges. When the request rate within a table is non-uniform, splitting a partition and dividing performance allocation proportionately can result in the hot portion of the partition having less available per- formance than it did before the split.

> The key observation that partitions had non-uniform access also led us to observe that not all partitions hosted by a storage node used their allocated throughput simultaneously.

>  If a table experienced throttling and the table level throughput was not exceeded, then it would automatically increase (boost) the allocated throughput of the partitions of the table using a proportional control algorithm. 

> adaptive capacity was also best-effort but eliminated over 99.99% of the throttling due to skewed access pattern.

> Bursting was only helpful for short-lived spikes in traffic and it was dependent on the node having throughput to sup- port bursting. Adaptive capacity was reactive and kicked in only after throttling had been observed. This meant that the application using the table had already experienced brief pe- riod of unavailability. 

> quick healing of impacted replication groups using log replicas ensures high durability of most recent writes.

>  the data of the live replicas matches with a copy of a replica built offline using the archived write-ahead log entries. 

> we have learned that continuous verification of data-at- rest is the most reliable method of protecting against hardware failures, silent data corruption, and even software bugs.

> We use formal methods [16] extensively to ensure the correctness of our replication protocols. The core replication protocol was specified using TLA+. Model checking has allowed us to catch subtle bugs that could have led to durability and correctness issues before the code went into production.

> formal methods have also been used to verify the correctness of our control plane and features such as distributed transactions.

> Backups or restores don’t affect performance or availability of the table as they are built using the write-ahead logs that are archived in S3.

> If one of the replicas is unresponsive, the leader adds a log replica to the group. Adding a log replica is the fastest way to ensure that the write quorum of the group is always met. 

> We have been able to run millions of Paxos groups in a Region with log replicas.

> To solve the availability problem caused by gray failures, a follower that wants to trigger a failover sends a message to other replicas in the replication group asking if they can communicate with the leader.

> The rolled-back state might be different from the initial state of the software. The rollback procedure is often missed in testing and can lead to customer impact. DynamoDB runs a suite of upgrade and downgrade tests at a component level before every deployment. Then, the software is rolled back on purpose and tested by running functional tests. DynamoDB has found this process valuable for catch- ing issues that otherwise would make it hard to rollback if needed.

> The additional challenge with distributed deployments is that the new software might introduce a new type of message or change the protocol in a way that old software in the system doesn’t understand. DynamoDB handles these kinds of changes with read-write deployments. Read-write deployment is completed as a multi- step process. The first step is to deploy the software to read the new message format or protocol. Once all the nodes can handle the new message, the software is updated to send new messages. New messages are enabled with software deploy- ment as well. Read-write deployments ensure that both types of messages can coexist in the system. Even in the case of rollbacks, the system can understand both old and new mes- sages.

> DynamoDB should be able to continue to operate even when the services on which it depends are impaired. If IAM or KMS were to become unavailable, the routers will continue to use the cached results for pre-determined extended period.


