# NV-Tree: Reducing Consistency Cost for NVM-based Single Level Systems


This paper introduces a new B+ tree variation that is optimized for NVM 
by reducing consistency cost from memory fence and cacheline flush operations.

1. Consistency is guaranteed only for leaf nodes. Internal nodes are re-built
from leaf nodes. 
2. Elements in leaf nodes are not sorted. A log-structure is used for leaf nodes to remove the need for frequent synchronization. Only appends are added
to a leaf node. This also increases concurrency. When a leaf node becomes full, a GC is triggered for cleaning and consolidation. 
3. Internal nodes are managed using arrays, rather than pointed based data
structures. 

