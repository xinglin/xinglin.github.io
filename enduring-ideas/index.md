# Enduring Ideas/Techniques that are still in use today


There are a few ideas/techniques that were invented a long time ago
but they are being used/re-implemented again and again, even in today's sytems. Why do people look and reimplement these techniques, even 
after 50 years? Because, even after 50 years, we face largely the same problem when we design new systems. 

Here are a few examples. 

| Ideas | First-time Introduced | What problem is solved? |
| :------: | :--: | :--:|
| Trasactions |    | Make it easy to develop on top of a data store. Modifications are atomic and failures are handled automatically by the system. |
| Log-structure storage/FS | | Turn random writes into sequential ones (good for hard disk drives, original motivation for developing LFS). No in-place overwrites (required for solid-state drives.). |
| Paxos |    | Reach concensus across a number of programs/nodes |

