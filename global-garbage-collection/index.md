# Coordinating Garbage Collection for Arrays of Solid-State Drives


This is the start of a series of blogs I plan to write about existing work related with garbage collection for SSDs. 

The main idea of this paper is to run garbage collection during idle times to minimize the impact on foreground workloads. Garbage collection is scheduled to run simultaneously at all SSDs, to maximize the time window during which there is no garbage collection, and thus higher application performance. Figure 5 shows their approach and Figure 10 shows the effects. 

![](./fig1.png)
![](./fig2.png)

I do have a few reservations about this paper. 

* Most of the evaluations focus on workloads with 40% or more write requests. If a workload has 40% or more write requests, I wonder how many of the write requests will be sequential and how many are random. If a workload's write requests are mostly random, write requests should not be as high as 40%. On the other hand, if a workload contains 40% or more write requests, I would assume most of these write requests will be sequential. Sequential writes however are much easier to handle. 

* The paper assumed it is a solved problem to predict idle periods. But what if the system is busy or has some small work all the time?  

* The paper only looked at average response time. They should look at tail latencies as well. Their approach might experience extremely long tail latencies, because they schedule to do garbage collection simultaneously at all SSDs.  

> We note a 60% reduction in response time for the HPC\(R\) read-dominated load and a 71% reduction for the 
> HPC(W) write-dominated load. 

71% reduction for HPC(W) is fine but I don't understand why HPC\(R\) also received 60% reduction. HPC\(R\) is read intensive and for read intensive workloads, Figure 1 in their paper show SSDs are able to provide consistent high performance. Besides, read intensive workloads should not trigger lots of garbage collection and thus the reduction should be much smaller. 

> Interestingly the baseline requires eight SSDs to provide a response time equivalent
> to that delivered by two devices using GGC. Even with
> 18 devices, the baseline performs 184% worse than GGC
> using only 4 devices.

Such results probably mean something was not right...

> It is mainly because (i) HPC workloads have much higher I/O demands than Enterprise
> workloads (refer to Table 8), and (ii) large requests in
> the HPC workloads are more frequently conflict with GC
> invocation of drives, increasing the I/O response times.

The first reason is simply flawed. Openmail has a 2x higher request rate and TPC-C's request rate is almost as high as HPC workloads. 
