# Sketch Data Structures


Tonight, I went to attend a local meetup event. The talk was given by a 
faculty from my own department, Jeff Phillips, an assistant professor
working in the area of big data analysis. The topic of his talk is about 
sketch data structure - data structures which uses a much smaller
amount of memory to summarize the represented data.This is a cool
technique and I am very interested to learn more.

Sketch data structures are commonly used in the following three use 
cases.  

- frequent items, like IP addresses  
- distinct items (harder to do)  
- approximate distribution of items   

And it is now also becoming popular to use sketches to represent matrixes and graphs  

- matrix sketch (hot)  
- graph sketch (new)  

Here is a list of sketches Jeff talked in his talk.  

- reservoir sampling: keep k samples over a streaming items  

      keep the first k items;  
      for j_th item where j>k, keep it at probability of k/j.  

Frequent items sketches   
Assume we want to find top k frequent items  
 
- Misra-Gries sketch (under-count)   
  
      initialize k counters = 0  
      for i_th item  
        if (we have a counter that is not 0 for this item)  
          increase the counter by 1  
        elsif (we have a counter that is 0)  
          increase the counter by 1 and assign this counter to this item  
        else  
          decrease all counters by 1  

- spacesaving sketch (over-count)  

      initialize k counters = 0  
      for i_th item  
        if (we have a counter that is not 0 for this item)  
          increase the counter by 1  
        else  
          find the counter with minimal value;  
          increase the counter by 1 and assign this counter to this item  

Two generic sketches to estimate frequency of any item  

- count-min sketch (over-count, report minimal value, support substraction) 

      initialize a two-dimensional t*k array of counters  
      pick t independent hash fuctions  
      for i_th item  
        foreach hash function h  
          increase the counter of h(item)  

      for any item, report the minimal value of h(item) as the estimation of 
      its frequency

- count sketch  

      initialize a two-dimensional t*k array of counters  
      pick t independent hash fuctions h_1,...,h_t, each maps an item to {1,...,k}
      pick t independent hash functions s_1,...,s_t, each maps an item to {+1, -1}  
      for i_th item  
        foreach hash function h_i  
          h_i(item) += s_i(item) 

      for any item, report the median value of h(item) as the estimation of 
      its frequency

Count sketch: "Finding Frequent Items in Data Streams", Moses Charikar,	Kevin Chen and
Martin Farach-Colton, ICALP '02 Proceedings of the 29th International Colloquium on Automata, Languages and Programming.	


