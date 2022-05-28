# Transaction


The four properties that a transaction in the database world has are ACID: A stands for atomicity, C for consistency, I for isolation and D for durability. 
The purpose of transactions is to maintain data in the face of concurrent access and system failures.

| Property | Definition | Technique to achieve that property |
| :------: | :--:|:--:|
| Atomicity| A transaction either succeeds or fails as a single operation. | Undo log | 
| Consistency | A database should start in a consistent state and end in another consistent state, after applying a transaction or transactions. | Relies on the other three properties. |
| Isolation | Each transaction should be executed as if it was the only transaction that is running. Transactions are isolated and are not aware of each other. Allowing concurrent transactions. | 2-phase locking or MVCC | 
| Durability | Writes made to the database will survive system failures. | Redo log | 

# Database Management Systems

The following content was selected from chapter 17, Transactions.

* Strict Two-Phase Locking (Strict 2PL)  
>   Rule 1. If a transaction T wants to read (respectively, modify) an object, it first requests 
>           a shared (respectively, exclusive) lock on the object.  
>   Rule 2. All locks held by a transaction are released **when the transaction is completed**.
>   
>   If two transactions access the same object, and one wants to modify it, their actions are 
>   effectively ordered serially: all actions of one of these transactions (the one that gets 
>   the lock on the common object first) are completed before (this lock is released and) the 
>   other transaction can proceed.

* Two-Phase Locking (2PL)  
>   Rule 1. If a transaction T wants to read (respectively, modify) an object, it first requests 
>           a shared (respectively, exclusive) lock on the object (same as strict 2PL).  
>   Rule 2. **A transaction cannot request additional locks once it releases any lock**.

* Recovery Manager  
>   The recovery manager of a DBMS is responsible for ensuring transaction atomicity and 
>   durability. It ensures atomicity by undoing the actions of transactions that do not commit, 
>   and durability by making sure that all actions of committed transactions survive systenl 
>   crashes, (e.g., a core dump caused by a bus error) and media failures (e.g., a disk is 
>   corrupted).

* Lock Implementation in DBMS  
>   The lock manager maintains a lock table, which is a hash table with the data object 
>   identifier as the key.  
>   A lock table entry for an object -- which can be a page, a record, and so on, depending on the DBMS -- contains the following inforrnation: the number of transactions currently holding a. lock on the object (this can be more than one if the object is locked in shared mode), the nature of the lock (shared or exclusive), and a pointer to the queue of lock requests.

* Concurrency control in B+ Trees

>   1. The higher levels of the tree only direct searches. All the 'real' data is in the leaf levels (in the forrnat of one of the three alternatives for data entries).
>   2. For inserts, a node must be locked (in exclusive rnode, of course) only if a split can propagate up to it from the modified leaf.

>   Searches should obtain shared locks on nodes, starting at the root and proceeding along a path to the desired leaf. The first observation suggests that a lock on a node can be released as soon as a lock on a child node is obtained, because searches never go back up the tree.

>   A conservative locking strategy for inserts would be to obtain exclusive locks on all nodes as we go down from the root to the leaf node to be modified, because splits can propagate all the way from a leaf to the root. However, once we lock the child of a node, the lock on the node is required only in the event that a split propagates back to it. In particular, if the child of this node (on the path to the modified leaf) is not full when it is locked, any split that propagates up to the child can be resolved at the child, and does not propagate further to the current node. Therefore, when we lock a child node, we can release the lock on the parent if the child is not full. The locks held thus by an insert force any other transaction following the same path to wait at the earliest point (i.e., the node nearest the root) that might be affected by the insert. The technique of locking a child node and (if possible) releasing the lock on the parent is called lock-coupling, or crabbing (think of how a crab walks, and compare it to how we proceed down a tree, alternately releasing a lock on a parent and setting a lock on a child).


# Concurrency Control Without Locking
## Optimistic Concurrency Control  
    * pessimistic: lock-based approach, 2PL
    * optimistic: do not use locks. Read-Validation-Write

> Optimistic concurrent control:
> 1. Read: The transaction executes, reading values from the database and writing to a private workspace.
> 2. Validation: If the transaction decides that it wants to commit, the DBMS checks whether the transaction could possibly have conflicted with any other concurrently executing transaction. If there is a possible conflict, the transaction is aborted; its private workspace is cleared and it is restarted.
> 3. Write: If validation determines that there are no possible conflicts, the changes to data objects made by the transaction in its private workspace are copied into the database.

> Each transaction Ti is assigned a timstamp TS(Ti) at the beginning of its validation phase, and the validation criterion checks whether the timestamp ordering of transactions is an equivalent serial order. For every pair of transac-tions Ti and Tj such that TS(Ti) < TS(Tj), one of the following validation conditions must hold:
> 1. Ti completes (all three phases) before Tj begins.
> 2. Ti completes before Tj starts its Write phase, and Ti does not write any
database object read by Tj.
> 3. Ti completes its Read phase before Tj completes its Read phase, and Ti
does not write any database object that is either read or written by Tj.

> To validate Tj, we must check to see that one of these conditions holds with respect to each comlnitted transaction Ti such that TS(Ti) < TS(Tj). Each of these conditions ensures that Tj's modifications are not visible to Ti.

> Further, the first condition allows T j to see some of Ti's changes, but clearly, they execute completely in serial order with respect to each other. The second condition allows Tj to read objects while Ti is still modifying objects, but there is no conflict because Tj does not read any object modified by Ti. Although Tj might overwrite some objects written by Ti, all of Ti's writes precede all of Tj's writes. The third condition allows Ti and Tj to write objects at the same time and thus have even more overlap in time than the second condition, but the sets of objects written by the two transactions cannot overlap. Thus, no RW, WR, or WW conflicts are possible if any of these three conditions is met.

> Checking these validation criteria requires us to maintain lists of objects read and written by each transaction. Further, while one transaction is being validated, no other transaction can be allowed to commit; otherwise, the validation of the first transaction might miss conflicts with respect to the newly committed transaction. The Write phase of a validated transaction must also be completed (so that its effects are visible outside its private workspace) before other transactions can be validated.
> A synchronization mechanism such as a critical section can be used to ensure that at most one transaction is in its (combined) Validation/Write phases at any time.

> Clearly, it is not the case that optimistic concurrency control has no overheads; rather, the locking overheads of lock-based approaches are replaced with the overheads of recording read-lists and write-lists for transactions, checking for conflicts, and copying changes from the private workspace. Similarly, the implicit cost of blocking in a lock-based approach is replaced by the implicit cost of the work wasted by restarted transactions.

## Improved Conflict Resolution for optimisic concurrency control
problem: If Ti writes all data items required by Tj before Tj reads them, we can still commit Tj. But basic optimistic concurrency control does not permit this. We don't know when Ti wrote the object at the time when we validate Tj, because we only have a list of read/write objects by Ti (without recording their timestamps).  

Details workflow is as following.  
> Before reading a data item, a transaction enters an access entry in a hash table. The access entry contains the transaction id, a data object id, and a modified flag (initially set to false), and entries are hashed on the data object id. A temporary exclusive lock is obtained on the
hash bucket containing the entry, and the lock is held while the read data item is copied from the database buffer into the private workspace of the transaction.

> During validation of T, the hash buckets of all data objects accessed by T are again locked (in exclusive mode) to check if T has encountered any data conflicts. T has encountered a conflict if the modified flag is set to true in one of its access entries. (This assumes that the 'die' policy is being used; if the 'kill' policy is used, T is restarted when the flag is set to true.)
> If T is successfully validated, we lock the hash bucket of each object modified by T, retrieve all access entries for this object (some of these entries could be added by other transactions), set the modified flag to true, and release the lock on the bucket. If the 'kill' policy is used, the transactions that entered these access entries are restarted. We then complete T's Write phase.


## Timestamp-Based Concurrency Control
This is not MVCC. Each object has a single copy. However, **this protocol may permit schedules that are not recoverable. Use MVCC instead.** 

> Each transaction can be assigned a timestamp at startup, and we can ensure, at execution time, that if action a\_i of transaction Ti conflicts with action a\_j of transaction Tj, a\_i occurs before a\_j if TS(Ti) < TS(Tj). If an action violates this ordering, the transaction is aborted and restarted.
> To implernent this concurrency control scheme, every database object O is given a read timestamp RTS(O) and a write timestamp WTS(O). If a transaction T wants to read object O, and TS(T) < WTS(O), the order of this read with respect to the most recent write on O would violate the timestamp order between this transaction and the writer. Therefore, T is aborted and restarted with a new, larger timestamp. If TS(T) > WTS(O), T reads O, and RTS(O) is set to the larger of RTS(O) and TS(T). (Note that a physical change--the change to RTS(O)-is written to disk and recorded in the log for recovery purposes, even on reads. This write operation is a significant overhead.)
> For writes, we need to check:  
> 1. If TS(T) < RTS(O), the write action conflicts with the most recent read action of O, and T is therefore aborted and restarted.
> 2. If TS(T) < WTS(O), a naive approach would be to abort T because its write action conflicts with the most recent write of O and is out of timestamp order. However, we can safely **ignore** such writes and continue. Ignoring outdated writes is called the Thomas Write Rule.
> 3. Otherwise, T writes O and WTS(O) is set to TS(T).

## Multiversion Concurrency Control
> The goal is to ensure that a transaction never has to wait to read a database object, and the idea is to maintain several versions of each database object, each with a write timestamp, and let transaction Ti read the most recent version whose timestamp precedes TS(Ti).
> If transaction Ti wants to write an object, we must ensure that the object has not already been read by some other transaction Tj such that TS (Ti) < TS(Tj). If we allow Ti to write such an object, its change should be seen by Tj for serializability, but obviously Tj, which read the object at some time in the past, will not see Ti's change.
> To check this condition, every object also has an associated read timestarnp, and whenever a transaction reads the object, the read timestamp is set to the maximum of the current read timestamp and the reader's timestamp. If Ti wants to write an object O and TS(Ti) < RTS(O), Ti is aborted and restarted with a new, larger timestamp. Otherwise, Ti creates a new version of O and sets the read and write timestamps of the new version to TS(Ti).

# Design Data-Intensive Applications book. 

>   Isolation is the property that provides guarantees in the concurrent accesses to data. Isolation is implemented by means of a concurrency control protocol. The simplest way is to make all readers wait until the writer is done, which is known as a read-write lock. Locks are known to create contention especially between long read transactions and update transactions. MVCC aims at solving the problem by keeping multiple copies of each data item. In this way, each user connected to the database sees a snapshot of the database at a particular instant in time. Any changes made by a writer will not be seen by other users of the database until the changes have been completed (or, in database terms: until the transaction has been committed.)

[知乎link][acdlink]
> ACD三个特性是通过Redo log（重做日志）和Undo log 实现的。 而隔离性是通过锁来实现的。  
> 重做日志用来实现事务的持久性，即D特性.  
> Undo log (MySQL):  
>   1. 实现事务回滚
>   2. 实现MVCC

[repeatable read vs read committed][repeatableread]  
> 在innodb中(默认repeatable read级别), 事务在begin/start transaction之后的第一条select读操作后, 会创建一个快照(read view), 将当前系统中活跃的其他事务记录记录起来;
> 在innodb中(默认read committed级别), 事务中每条select语句都会创建一个快照(read view);
> 其中INSERT操作在事务提交前只对当前事务可见，因此产生的Undo日志可以在事务提交后直接删除（谁会对刚插入的数据有可见性需求呢！！），而对于UPDATE/DELETE则需要维护多版本信息，在InnoDB里，UPDATE和DELETE操作产生的Undo日志被归成一类，即update_undo
> 在回滚段中的undo logs分为: insert undo log 和 update undo log
>   *   insert undo log : 事务对insert新记录时产生的undolog, 只在事务回滚时需要, 并且在事务提交后就可以立即丢弃。(The new record is stored in the table itself. No need to serve reads from undo log. Undo records for insert only serve for transaction rollback. log only needs to tell when that new record was inserted so that we can determine when to undo it (remove it during rollback).)
>   *   update undo log : 事务对记录进行delete和update操作时产生的undo log, 不仅在事务回滚时需要, 一致性读也需要，所以不能随便删除，只有当数据库所使用的快照中不涉及该日志记录，对应的回滚日志才会被purge线程删除

[acdlink]: https://zhuanlan.zhihu.com/p/48327345
[repeatableread]: https://segmentfault.com/a/1190000012650596
