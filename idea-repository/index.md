# Idea Repository


List of ideas to explore.

* Interaction between a filesystem and a computational SSD that does transparent compression. 
    * Problem: How to manage SSD space in this case? What API should be use?
    * Can use SSDSim to explore the design space.

* Key prefix compression in k/v store
    * Problem: in levelDB, it uses fixed-size key prefix compression. Can we do better than this? 
    * In Rockset/Foundationdb, secondary indexes are stored as <key, null> key/value pairs. This creates an opportunity that significant storage saving can be achieved if we can do a better key compression.

* [Zoned namespace SSDs][zonedns]

[zonedns]: http://zonedstorage.io/introduction/

* VectorBtree, VectorBloomFilter, VectorKV, VectorDB, VectorFS
    Investigate how to levarage AVX extension instructions to accelerate a B/B+ tree index, certain operations in a KV store, a DB or a FS.

    Execution plan: 
    
    * Understand how lookup works in an indexing data structure; determine whether we can easily do batch lookups; 
    * find an open-source implementation; modify and add support for batch lookup;
    * measure the batch lookup speedup compared with point lookup -> best result.
    * Integrate batch lookup into some systems, such as a KV store or a DB.

* VectorZip  
    Use AVX instruction to accelerate compression (zstd/lz4)
