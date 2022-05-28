# Copysets: Reducing the Frequency of Data Loss in Cloud Storage


# Summary
Copysets proposed another approach/perspective for data replication, than the traditional random replication. 
In random replication, *N* nodes are selected randomly from the cluster to store a data chunk.
Random replication provides a strong protection against uncorrelated failures. 
For each individual chunk, since it can be stored randomly at N nodes in the cluster, 
it is quite resilient to data loss. However, when we consider all data chunks, 
any *N* node failure will almost lead to loss of some chunks, as long as these chunks happen to be replicated at these exact N nodes. For certain large scale
deployments, they prefer to have much less frequent data loss and are willing
to achieve that, even at the expense of larger amount of data loss during each data loss.

They introduced the concept of *Copysets*: a set of unique nodes that are used to store
a copy of a chunk. Only when all nodes in a copyset are down will we lose that chunk. 
By limiting the number of copysets in a  cluster, one can reduce the number of 
data loss probability. 

# Terms used in the paper  
* R: replication factor. Also means the number of replicas of each chunk. 
* Copyset: a set of R unique nodes, to store a chunk
* N: the number of nodes in a cluster
* S: scatter width. Each node's data is split uniformly across a group of S other nodes. 
* P: the number of permutations used, to generate the copysets. P = S/(R-1)

# How it works  
* Use permutations, P times, to generate copysets.
    * Each copyset overlaps with another by only one node.
    * The copysets covers all nodes equally, for load balancing.
* Pick a random node in the cluster as the primary node. The other replicas can 
be placed on the nodes of a randomly chosen copyset that contains the primary node.

# My Comments
* This paper is not easy to follow. The scatter width and the copyset are 
not easy to understand simultaneously. 
* The introduction of the abbreviations makes the paper harder to understand. I 
forgot what *P* stands for when reading the paper. 
* The paper cited from their collaborators in large paragraphs, to help motivate
their work. I find this can be done in a more professional approach. 
* A lot of simulation results. Quite surprised to see in a full ATC paper. 
The evaluation on HDFS/RAMCloud was quite light. 

