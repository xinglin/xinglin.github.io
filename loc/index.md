# Lines of Code for Various Software


Just out of curiosity, I wanted to know how many lines of code for various software that I am interested in (mostly, file systems and key-value stores). ext4 is the newer version for ext2, mainly added with the journaling mechanism to provide data consistency for crashes. The number of codes for ext4 is 5 times more than ext2. f2fs is a new file system that is optimized for flash devices while nova is a new file system that is optimized for persistent memory. Both of them are in the order of 30,000 lines of code. I also looked at leveldb and rocksdb, which are key-value stores. rocksdb is derived from leveldb and has gained great popularity in recent years. While rocksdb focked from leveldb, as we can see, the number of lines of code in rocksdb is >10 times more than leveldb. Maybe, we should really consider rocksdb as a different key-value store. While key value stores are supposed to be more performant and provide simple APIs, it is interesting to see that rocksdb has grown to be even more complex (4 times more in lines of code) than an actual relational database: postgreSQL. 

| Software | Type |Lines of Code |
| :------: | :--: | :-----------: |
| ext2     | file system | 10,000        |
| ext4     | file system | 52,000        |
| nova     | file system | 21,000        |
| f2fs     | file system | 27,500        |
| leveldb  | key value store | 29,000        |
| rocksdb  | key value store | 443,000       |
| postgreSQL | relational database | 113,000     |

