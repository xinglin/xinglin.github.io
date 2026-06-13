# Detect cycle in a singly linked list


Question: given a singly linked list, detect whether it contains a cycle. 

Solution: use two pointers: one slow and one fast pointer. 

- since fast pointer is moving faster, both pointers can't meet before they enter the cycle. Their meeting point must be part of 
the cycle. 
- In order for them to meet, it must be fast pointer catching up with the slow pointer. 
- They are guarantteed to meet because at every tick, fast pointer is 1 step closer to the slower pointer. 
So, eventually they will meet. By the point they meet the first time, fast pointer travels exactly k more loops than the slow pointer.
- When they meet, fast pointer travels 2T nodes. slow pointer travels T nodes. Their difference is k*L, length of loop. 
So, 2T = T + k*L, which means T = k*L. So, by the time they meet, slow pointer travels exactly k*L nodes. 
- A: The distance from the start of the list to the start of the cycle. 
- B: The distance from the start of the cycle to the meeting point.
```
A + B = k*L. 
A = (k-1)*L + L - B. 
```
- And the difference from the meeting point to the start of the cycle is also L - B. 
  So, both pointers will meet at the start of the cycle at the same time. 



