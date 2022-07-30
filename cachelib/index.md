# The CacheLib Caching Engine: Design and Experiences at Scale

> CacheLib is a general-purpose caching engine, designed based on experiences with a range of caching use cases at Facebook, that facilitates the easy development and maintenance of caches.

> CacheLib was first deployed at Facebook in 2017 and today powers over 70 services including CDN, storage, and application-data caches.

> All of these systems process millions of queries per second, cache working sets large enough to require using both flash and DRAM for caching, and must tolerate frequent restarts due to application updates, which are common in the Facebook production environment.

> systems built on CacheLib achieve peak throughputs of a million requests per second on a single production server and hit ratios between 60 and 90%.

> Because workloads at Facebook are less cacheable than is generally assumed, caches at Facebook require massive capacities to achieve acceptable hit ratios.
Caching systems at Facebook often comprise large distributed systems where each node has hundreds of gigabytes of cache capacity. This makes the use of flash for caching attractive. 

> CDN focuses on serving HTTP requests for static media objects such as photos, audio, and video chunks from servers in user proximity. 

> Churn refers to the change in the working set due to the introduction of new keys and changes in popularity of existing keys over time. The popular YCSB [24] workload generator assumes that there is no churn, i.e., each key will remain equally popular throughout the benchmark.

> In Facebook production workloads, we find a high degree of churn. We define an object to be popular if it is among the 10% of objects that receive the most requests. Across all workloads, over two-thirds of popular objects in a given hour fall out of the top 10% after just one hour. Such high churn applies independent of which hour we use as the baseline, for different percentiles (e.g., top 25%), and with different time granularities (e.g., after 10 minutes, 50% of popular objects are no longer popular). 

> For Storage and CDN , we find that 64KB and 128KB chunks, respectively, are very common, which result from dividing large objects into chunks. 

> production caches frequently restart in order to pick up code fixes and updates. For example, 75% of Lookaside caches and 95% of CDN caches have an uptime less than 7 days. Even systems such as Storage and Social-Graph which have longer uptimes on average follow a regular monthly maintenance schedule which requires restarting cache processes. 

> For example, under LRU, popular Items frequently compete to be reset to the most-recently-used (MRU) position, leading to lock contention and limit throughput. 
CacheLib adopts a simple solution to reduce contention: Items that were recently reset to the MRU position are not reset again for some time T [9, 87]. T is set to 60 seconds in most deployments.


