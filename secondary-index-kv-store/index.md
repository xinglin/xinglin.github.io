# Secondary Index Built on top of a Key-value Store


Key-value stores such as RocksDB have been used as the storage engine in several databases (MySQL/Mongo/Rockset/FoundationDB). A database typically requires support of secondary indexes. To support secondary index, a common approach is to store the secondary index as yet another key-value pairs. Let's take a look at the following example. 

Employee table  
| userid (primary key) | salary | age |
| :------: | :--: | :--:|
|Alice | 40000 | 30 |
|Bob | 20000 | 23|
|Charlie | 30000| 30|
|David | 20000| 25|


To store the user table, we can store each record using `tableName,primary_key -> salary,age` as key/value pairs.

| key | value |
| :------: | :--:|
| Employee,Alice| 40000,30 |
| Employee,Bob| 20000,23 |
| Employee,Charlie| 30000,30|
| Employee,David| 20000,25|


To support secondary index based on the salary column, one could store `salary,primary_key` as the key with an empty value into the key/value store (or set the value to be something that would be interesting for the use case). This will ensure efficient search for employees with a particular amount of salary, as the key/value pairs are sorted by keys.

| key | value |
| :------: | :--:|
| 20000,Alice| |
| 20000,David| |
| 30000,Charlie| |
| 40000,Alice| |

* For both cases, the key/value pairs are sorted based on keys. Because keys are sorted, similar keys will be grouped together. Thus, **prefix compression**  is important to reduce space consumption when storing keys, as done in LevelDB (in LevelDB, for every N key-value pairs in an SSTable, the key for the first key-value pair is stored in complete while the keys for the rest key-value pairs are prefix-compressed based on the key before it. Each time a complete key is stored, it is called a restart point in LevelDB. Restart points enable quick binary search within an SSTable.). 
* Since keys are sorted and nearby keys are similar, could we use a data structure such as trie to organize data within an SSTables? Maybe make sense when values are small. 

Reference
* [LSM-trie][lsm-trie]

[lsm-trie]: http://ranger.uta.edu/~sjiang/pubs/papers/wu15-lsm-trie.pdf
