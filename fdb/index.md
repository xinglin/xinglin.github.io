# FoundationDB: A Distributed Unbundled Transactional Key Value Store


> Key and value sizes are limited to 10 KB and 100 KB respectively for better performance. Transaction size is limited to 10 MB

> ## Transaction processing
> ### Optimistic Concurrency Control + MVCC
> A client transaction starts by contacting one of the Proxies to obtain a read version (i.e., a timestamp). The Proxy then asks the Sequencer for a read version that is guaranteed to be no less than any previously issued transaction commit version, and this read version is sent back to the client. Then the client may issue multiple reads to StorageServers and obtain values at that specific read version. Client writes are buffered locally without contacting the cluster. At commit time, the client sends the transaction data, including the read and write sets (i.e., key ranges), to one of the Proxies and waits for a commit or abort response from the Proxy. If the transaction cannot commit, the client may choose to restart the transaction from the beginning again.
> 
> A Proxy commits a client transaction in three steps. First, the Proxy contacts the Sequencer to obtain a commit version that is larger than any existing read versions or commit versions. The Sequencer chooses the commit version by advancing it at a rate of one million versions per second. Then, the Proxy sends the transaction information to range-partitioned Resolvers, which implement FDB’s optimistic concurrency control by checking for read-write conflicts. If all Resolvers return with no conflict, the transaction can proceed to the final commit stage. Otherwise, the Proxy marks the transaction as aborted. Finally, committed transactions are sent to a set of LogServers for persistence. A transaction is considered committed after all designated LogServers have replied to the Proxy, which reports the committed version to the Sequencer (to ensure that later transactions’ read versions are after this commit) and then replies to the client. At the same time, StorageServers continuously pull mutation logs from LogServers and apply committed updates to disks.
> ### Resolver Conflict Detection
> - Divide the key space into ranges. key ranges are assigned/partitioned among resolvers. 
> - For each key range, resolver stores its last commit version. 
> - For a pending transaction, get key ranges for each read key. 
> - check against the last commit version for each key range. 
> - if the read version is smaller than any of the last commit version of any read key range, we have a read-write conflict (read value can be stale). mark this transaction as failed. 
> - Otherwise, this transaction can be committed. 
> - update the last commit version of that key range to the commit version of the current transaction.  
> ### Versioning & Transaction Batching
> At 1 million versions per sec, using a signed int64, we can support ~300,000 years. When we have more than 1 millions transactions per sec, proxy uses batches: assign the same transaction ID for a batch of transactions. 
> 
> **Transaction batching.** To amortize the cost of committing transactions, the Proxy groups multiple transactions received from clients into one batch, asks for a single commit version from the Sequencer, and sends the batch to Resolvers for conflict detection. 
> - The resolver evaluates each transaction in the batch one at a time in strict order. 
> - If transaction B after A (both from the same batch) has a conflict with A, B will be marked as failed transaction.  
> 
> The Proxy then writes committed transactions in the batch to LogServers. The transaction batching reduces the number of calls to obtain a commit version from the Sequencer, allowing Proxies to commit tens of thousands of transactions per second without significantly impacting the Sequencer’s performance. Additionally, the batching degree is adjusted dynamically, shrinking when the system is lightly loaded to improve commit latency, and increasing when the system is busy in order to sustain high commit throughput.
> 5 sec MVCC transaction window: every transaction has to complete in 5 secs. Main reason is then resolver only need to keey the most recent 5-sec updates in memory to detect conflicts among trasactions. 


> ### Failure Handling through the Recovery Path
> In the transaction management system of FDB, we handle all failures through the recovery path: instead of fixing all possible failure scenarios, the transaction system proactively shuts down when it detects a failure. As a result, all failure handling is reduced to a single recovery operation, which becomes a common and well-tested code path. Such error handling strategy is desirable as long as the recovery is quick, and pays dividends by simplifying the normal transaction processing.


## Simulation Testing
> - Flow: the underlying runtime engine can be swapped with a simulated implementation from a real implementation. 
> - Everything runs in a single thread, with the seed determines the code path. With different seed values, different executions (disk IO/packets delays, etc) will happen. Virtual time. can jump directly to next timestamp.
> - Buggify MACRO: developers can add buggy cases into source code and it will be triggered probabilistically.



