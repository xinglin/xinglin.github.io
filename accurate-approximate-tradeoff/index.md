# Trade-off Between Accuracy and Runtime Overhead


During the weekly technical forum within the NetApp ATG group, a team member presented the trip report for 
OSDI '14 this year. In his talk, he mentioned a paper that pushed me to think more in this line. The paper 
is about how to estimate the working set of a workload, using less memory and runs much faster. 
The only related work that dealt with the same problem was proposed more than 10 years ago and in that
work, the authors tried to get the exact working set size, with a very high demand for memory
and runtime. So, in their work, they tried to estimate it, with a goal to reduce memory demand and runtime.
They got very impressive results. Together with other papers, I suddenly realized that it is actually
quite common in compute science community, to take this approach: **when it is too expensive to 
get the accurate result, it is usually a good time to think about how to approximate it with much lower overhead.**

There are a few examples, pop up in my mind. One is the use of bloom filter in tracking unique chunks 
for deduplication. The memory consumption is too high to keep track of every unique chunks. 
So, bloom filters are used, to keep track of memberships, with a low probability of introducing duplicates.  
[Sparse Indexing][sparse-index] is another example: a full chunk index is not scalable as data size increases. 
Thus, instead of deduplicating against the full chunk index, they only deduplicate new data blocks with a few 
data blocks that are most similar. There are probably more such projects that I can not remember all of them.

12/16/2014: found another example - [BlinkDB][blinkdb]. It trade-offs query accurary for 
response time, and thus enables interactive queries over massive data by running
queries on sampled data and presenting results with annotated error bars.

02/21/2015: add another example - [MRC estimation][mrc-fast15]. This is another piece of work, trying
to approximate miss rate curve, with low memory consumption and runs quickly. The key idea is again
to do sampling! It samples a subset of IO requests and uses re-use distances from this subset as an approximation
for an application.

[sparse-index]: https://www.usenix.org/system/files/conference/osdi14/osdi14-paper-wires.pdf
[blinkdb]: http://blinkdb.org/
[mrc-fast15]: https://www.usenix.org/conference/fast15/technical-sessions/presentation/waldspurger

