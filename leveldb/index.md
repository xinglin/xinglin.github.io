# Delete Request in LevelDB


When I searched for how leveldb/rocksdb and a few other KV stores
handle delete requests, the document will usually say ``we insert a special
a tombstone value or a tombstone entry.''. However, for KV stores which
allow storing arbitrary keys and values, I am not sure which value we should 
use as the special tombstone value, as whatever value we pick as a tombstone value, it could be a valid value stored by the user. I looked up leveldb source
code and I think I have figured out how leveldb handles this case. 

In stead of using a special tombstone value, it instead inserts a special
tombstone record which indicates whether this is a new PUT or a delete.
The internal key used in leveldb includes both an increasing sequence number
but also a type field. The type field can have two values: kTypeValue (0x01) or kTypeDelete (0x00). If it is a kTypeDelete, the value field is empty and 
this request is a delete operation.

```
internal_key = user_key + sequence_num (56-bit) + type (8-bit)
```

So, essentially, for each KV pair, from a PUT or a Delete request,
besides user_key/user_value, leveldb also stores a sequence number and
a type which indicates whether this is a PUT or a delete (this KV is deleted). 

```
//db/memtable.h:

  // Add an entry into memtable that maps key to value at the
  // specified sequence number and with the specified type.
  // Typically value will be empty if type==kTypeDeletion.
  void Add(SequenceNumber seq, ValueType type, const Slice& key,
           const Slice& value);
```
