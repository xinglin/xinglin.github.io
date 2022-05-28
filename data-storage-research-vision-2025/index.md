# Data Storage Research Vision 2025


NFS sponsored a data storage visioning workshop in 2018. 
The workshop was hosted by IBM Research. From the workshop, 
they produced a report, summarizing the discussion from the workshop. 
[The report][dsrv2025] outlines the future research challenges and opportunities that the attendees recommend to work on. 

The discussion of the workshop was organized in the four groups. 
Below are some interesting points I copied from the report. 

1. Storage for the Cloud, Edge, and IoT Systems
  * More desegregated, composable software architectures are highly desirable. 
  * One common feature of all IoT devices is that they have limited hardware resources. To address this constraint, we should explore how to identify and discard unimportant data in a timely fashion, and how to balance among storage, preprocessing, and communication between IoT devices and cloud.
2. AI and Storage
  * The increasing richness of data (e.g., multiple correlated mixed type streams, images, sound, video) requires more complex ML and Deep Learning (DL) approaches.
  * AI stage awareness. Storage that is aware of the distinct stages or phases of AI processing can optimize AI pipelines via techniques such as caching of intermediate results, tracking of lineage, provenance, and checkpointing [46, 62, 72, 84].
  * Compute architecture and data optimization. AI platforms follow distinct distributed computation architecture patterns (e.g., data parallel and model parallel [57, 95]). Memory hierarchy and data layout design for such computation patterns should also be a focus for future storage research. APIs that express the data access intent of an AI algorithm can also be a powerful tool to integrate memory hierarchy and data layout optimizations with AI computation.
  * Unique characteristics. AI algorithms have unique characteristics that can be exploited to create efficient storage designs. Example characteristics include a tolerance to small amounts of data loss, very structured access patterns, and the ability to use and extrapolate from lossy compression
  * Access and distribution characteristics. Emerging access methods and characteristics associated with AI workloads, such as stream processing [9] or edge storage [12], also create unique challenges that should be focused on in future storage research.
  *  Security and compliance. The use of AI brings a new dimension to data security. As industries and users demand that decisions made by AI algorithms be reproducible, transparent, and explainable, pressure builds on enterprises to put in place data-management mechanisms to govern what data is and how it should be used to generate AI models and consequent insights [8, 10, 11, 35, 48].
  * Performance and placement optimizations. ML algorithms can be applied to predict popular data and application patterns, which help improve various storage techniques, including tiered caching, prefetching, and resource provisioning. Adapting caching policy using online learning can have significant benefits: recent work [93] shows that using ML techniques to select between LRU and LFU replacement policies resulted in a significantly improved total cache hit rate under even smaller memory constraints. The key takeaway is that ML techniques are valuable for solving online optimization problems such as caching with the caveat that the primary knobs of control have to be orthogonal. We believe that ML can be applied with great success for other problems such as non-datapath server-side caches [36, 54, 58, 60, 70, 80, 83], distributed storage caches such as in hyper-converged systems, dynamic multitiered and hybrid storage systems [50, 64, 86, 94], and hybrid systems comprising DRAM and persistent memory.
  * Intelligent storage devices within storage systems. Computing-enabled storage devices refer to storage devices with ample internal computing capability, typically also including some AI capabilities. On one hand, computing storage devices assume part of the storage-related computing functionality so that the running storage system is alleviated from excessive overheads. Therefore, the storage system can deliver improved performance. On the other hand, with more intelligent devices, researchers need to determine what level of intelligence is appropriate to offload to the device and propose techniques at the storage system level to achieve the best synergy.
3. Evolution of Storage Systems with Emerging New Hardware  
Below are research challanges and opportunities related with byte-addressable NVM. 
* Operating System and Application Development Support
    * Security and integrity: How to efficiently support encryption and low-latency access control at the granularity of memory accesses? How to avoid corruption of persistent data from malicious or buggy applications using lightweight mechanisms? Are language-level mechanisms appropriate or sufficient?
    * Abstractions for persistent data: B-NVM can provide a single-level memory with a single namespace for both volatile and persistent data. What are the appropriate naming abstractions and required OS support? Are relative pointers the appropriate addressing mechanism? How should garbage collection be organized in this environment?
    * Consistency and transaction support: Research is needed to transition the most efficient techniques into actual systems, and to build tools for building/migrating applications for direct memory access.
* Distributed Shared NVM-Based Storage
    * Software support. A strong research effort is needed in identifying the mechanisms and abstractions necessary to support future distributed, shared B-NVM. The problems are challenging and solutions are necessary to meet the scalability, reliability, usability, correctness, and latency requirements of applications. Many of these issues (listed below) have been previously examined in the context of single-server B-NVM systems, while distributed versions of these problems have been studied in the context of traditional block storage and TCP/IP networks. However, the distributed version of these problems deals with a far larger design space than server-attached B-NVM, and the micro-second-level latencies of B-NVM and direct-access RDMA-like protocols changes the nature of the problem qualitatively, requiring vigorous new research to explore the design space effectively. Some of the issues needing new research and lightweight solutions in the distributed B-NVM context are as follows:
        * Transparent global naming: How should shared data be named, distributed, and accessed? What are the overheads for metadata management?
        * Transaction management: How should transactions spread across multiple B-NVM hosts be orchestrated? How can RDMA-like protocols or atomics be exploited? What is the appropriate form of distributed logging?
        * Reliability and availability: Whether replication or erasure coding be employed? How to support wearleveling across nodes?
        * Consistency and coherence: What are the appropriate consistency models for balancing application requirements and performance?
        * Crash resilience and durability: How does one handle failures of nodes holding TBs of B-NVM data?
        * Metadata management: How does one handle the size and overheads of metadata management?

[dsrv2025]: https://www.fsl.cs.stonybrook.edu/docs/nsf/a1-amvrosiadis.pdf
