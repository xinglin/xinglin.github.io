# Key Value Systems


* [LSM-tree survey][survey]
* WiscKey: key/value separation
* HashKV: store cold keys separately.
* LSM-trie
* PebblesDB
* KVell
* [Skip-Tree][skiptree]: put kv items directly to non-adjacent layers (skipping some intermediate layers) to reduce write amplification from level-to-level data compaction.
* TRIAD: separate hot keys from cold keys. hot keys are stored in memtable for as long as possible. Use multiple memtables and flush them at once to disk. 
* [Pipelined Compaction Procedure][pcp]: A compaction contains three phases: read phase, merge-sort phase, and write phase. Pipeline these three phases, to overlap CPU and disk IO. 
* [cLSM][clsm]: LSM-tree for multi-core processors. It exploits multiprocessor-friendly data structures and non-blocking synchronization. cLSM supports a rich API, including consistent snapshot scans and general non-blocking read-modify-write operations.
* [Towards Accurate and Fast Evaluation of Multi-Stage Log-Structured Designs][fast16-lim]
* [Dostoevsky][dostoevsky]: evaluated against well-tuned LSM-trees. Extends the design space of LSM-trees with a new merge policy by combining leveling and tiering. very useful for certain workloads that require efficient writes, point lookups, and long range queries with less emphasis on short range queries.
* [Mutant][mutant]: decide which type of cloud disks to place SStables, based on access pattern.
* [diff-index][diffindex]: use LSM-tree as secondary indexes.

[survey]:https://arxiv.org/pdf/1812.07527.pdf
[diffindex]:https://openproceedings.org/2014/conf/edbt/TanTTF14.pdf
[mutant]:https://ymsir.com/papers/mutant-socc.pdf
[skiptree]: https://ieeexplore.ieee.org/document/7569086
[pcp]: https://www.computer.org/csdl/proceedings-article/ipdps/2014/06877309/12OmNvBIRP6
[clsm]:https://dl.acm.org/doi/abs/10.1145/2741948.2741973
[fast16-lim]: https://www.usenix.org/system/files/conference/fast16/fast16-papers-lim.pdf
[dostoevsky]:https://nivdayan.github.io/dostoevsky.pdf
