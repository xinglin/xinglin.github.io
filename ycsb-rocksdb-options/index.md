# Use Options File for Benchmarking RocksDB using YCSB


In YCSB, we can also use the options file (`-p rocksdb.optionsfile=/tmp/ycsb-options.ini`, e.g.,) to configure the options for RocksDB. 
However, for a new user, it is actually not easy to figure out what needs to be configured
for RocksDB for YCSB. Here is a quick way to get it start. 

Use the ycsb tool to load a rocksdb without specifying the options. 

    $ ./bin/ycsb load rocksdb -s -P workloads/workloada -p rocksdb.dir=/tmp/ycsb-rocksdb-data

Then, when we look into the rocksdb datadir, we can actually be able to find the option file that is generated automatically: OPTIONS-00008 or with a different number.
We can copy out this options file to somewhere else, and delete all files in the rocksdb dir.
Next, we can modify the options file. 
Specifically, we should modify the CFOptions and TableOptions/BlockBasedTable 
for the `usertable` sections, as YCSB will use this as the database (or column family) for loading data and running tests.

Hope this helps.
