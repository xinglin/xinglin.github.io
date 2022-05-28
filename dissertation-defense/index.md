# Reflection After Dissertation Defense


After six years in the PhD program, since August 2009, I finally defended my dissertation today. I feel grateful that I finally make it. I do think I am one of the luck ones: I have a really nice adviser and he gives me strong support to do research in areas which I am interested in. I feel grateful for all the help I have received from my advisor, committee members, office mates and co-workers. The journey would be much challenging if I did it alone. Thank you all! 

**_August 7, 2009: From China to United States_**  
I started as a fresh PhD student in August 2009. During that semester, we had 6 people coming from China to join the department as PhD students. Five of us negociated online
and we actually took the same flight to come to United States. We stayed overnight 
at the Los Angeles International Airport before we arrived at our destination: Salt Lake City at Utah. Everyone felt excited when we first arrived. Salt Lake City is beautiful: it has very clear and blue sky and it is surrounded with mountains. After we settled down, our school life started. 

**_Year 1-2: Taking Courses_**  
During the first two years, we were mainly taking courses. The idea is to finish course
requirements as soon as possible so that we can focus on research in our later years. 
From these courses, we did build more solid understanding about basic concept
in computer systems. I had the chance to take the operating system course from Matthew Flatt, the network security course from Sneha Kasera, Computer Architecture from Rajeev Balasubramonian, Data Mining from Jeff Philips,
Database Kernels from Feifei Li. Matthew used to write code when he was giving 
his lectures. I did not like that way by that time (I do appreciate that way now).
It was really great to build some relationship with
these professors when we were taking their classes. For example, I worked closely
with Rajeev when I was taking the Advanced Computer Architecture course and by the
end, we were able to submit a paper to a workshop. That paper became the first
 publication during my PhD study and I got the chance to give the talk and attend
ISCA in 2011. During the interaction with these professors, you can also tell which professor you are interested to work with. This is probably the most important issue
to figure out when you are in your early stage of PhD study. 

 
At the same time, I was also looking for RA positions and Eric Eide did some interviewswith me during an informal lunch, together with my current adviser. Since I had some hacking experience in the Window kernel, they let me to try RA for one semester. For the first semester, they assigned a project for me. It was to use some data collection library so that a user can send their experiemnt data from distributed compute nodes to this data collection node. We were trying to figure out how this data collection library scales, mainly how many nodes can it support. It seemed that this library worked pretty good and we decided to use it by the end. After the first project, I started to discuss the next project with my advisor. I had some experience in process migration and wanted to do some work with virtual machine migration. So, I read lots of papers about virtual machine migration at that time and also worked on OpenVZ migration in Emulab. I also started to write a survey paper about process and virtual machine migration. But we never tried to submit it somewhere. 


I still remember that it was quite challenging for me to understand a paper by that time. Our group is running a paper reading semester and in most cases, we read one paper every week. Given nearly no research experience in this field, it is really difficult to
understand what a paper is about. I still remember that for every week, I spent the whole thursday night to read a paper once, to get farmilar with its work. During the morning on 
the next day, I need to read it again. This time is to digest this paper and write summary about that paper. This paper reading exercise took almost 8 hours already every week. But the cost eventually pays off: the more papers I read, the more I become familar with that field and it has become easier to read papers nowadays.  

**_Year 3_: Started to Work on Deduplication**  
During the first two years, I shared the office with an indian Master student. He was working on using deduplication to store disk images for Emulab. When I got to know about deduplication, I found it was a pretty cool idea. He presented a poster for his project for new graduate visit. I had some discussion with him for his project and found I actually had a better way to improve his system. He was busy with his Master thesis writing at that time while I was focusing on implementing my design. In 2011, I went to attend the best conference in file and storage systems. I found I really liked the papers presented there. When I came back from that conference, I continued to work on this deduplication project and planed to submit this to FAST 2012. I worked full-time on this project in summer 2012 and we managed to submit this work to FAST in time (though we missed two data points by that time). I was busy with experiments by that time and Eric Eide did most of the writing (Thanks, Eric). When the reviews came back, the paper was rejected. Some reviewers liked the idea while others did not. We tried a re-submission to USENIX ATC in 2013. I did some substantial re-writing and we also improved the paper. The results were even
worse: no reviewrs liked the idea and they all gave rejects (some are weak rejects) and complained that our work was lacking of innovation. Partially I agreed (the contribution was probably not substantially enough for a FAST or USENIX ATC paper). After this rejection, I showed no interests in continuing improving this work and switched to work on solving fragmentation problem for deduplicating storage systems. 

**_Feb. 2013_: the Turning Point**   
At that time, deduplication and Flash are two hot topics for storage research. I continued to attend FAST conference every year. At FAST 2013, I turned in my resume to HR from EMC and on my resume, I said that I was working on solving fragmentation problem for deduplicating storage systems. That catched the eyes of one important person for my PhD study: Fred Douglis. He was also working on a similar project and he was looking for interns. We had a phone interview and exchanged what we were working on and found a perfect match. Then, I started to work as a summer intern at EMC for the summer in 2013 and that is the turning point for my research. 

**_Year 4--5_: Two Internships with Fred Douglis**  
During the summer in 2013, I went to New Jersey, to work with Fred as an intern. With Fred's mentoring, we managed to submit a paper to FAST in 2013. It was not a strong submission since I failed to fix some bugs in my prototype implementation with DataDomain Filesystem (DataDomain Filesystem is a complex system and what makes it worse is it is written in C and there is no easy way to know how a function calls the other.). This paper did not get in but at the same time, a very similar work from HP labs got published. EMC has very strict policies regarding access control and I was not able to continue to work on this while I came back to school (can only access their system when I am employed by them). The next summer, Fred invited me again to do another internship. I saw that was a positive sign and I did not even ask what project I would work on before I started my internship. 

During the second internship, I worked on Migratory Compression which transforms an input into a more compressible format. The good part of this project is that I did not need to hack DataDomain FileSystem anymore. It only requires some coding in perl which is much easier. This project was quite successful: I made significant progress in the first month and implemented most required function. During the seond month, I had already started to do evaluation and fine-tuning. Before I left, we had the first draft of our paper ready. We submitted it to FAST 2014. When the nofication of acceptance came out,
I was really happy: I had not imaged that this could happen and it is true now! I had a paper at the best storage conference! This can be used as a significant part for my PhD dissertation.

**_Final Year_: Wrap-up**  
Since this is my final year, we decided we need to submit the deduplication work which I worked on in 2012 at school somewhere. No one will continue to work on that once I leave. We decided to submit it to a testbed specific conference: TridentCom. My advisor had a few paper published there and he was confident with our submission to there. Finally, our paper was accepted there! I also did some work with GNU tar and initially, I was planning to submit a short paper to MSST. The idea of this work was very similar as Migratory Compression and I thought I should check with Fred to see whether he is interested to be co-author. What's surprising is that he told me that, within EMC, they had done something similar and we could combine our work together. We decided that we could possibly write this as a positioning paper and send it to HotStorage. We managed to pull it off and put together a paper in just two weeks. It got accepted! With all these papers published, I felt confident to defend my dissertation. With the demands from my advisor, I put quite a lot of efforts in combining four pieces of my work into a conherent dissertation. 

The defense went well. Just before my defense, I gave two talks at USENIX ATC and HotStorage respectively and that helps me to gain more experience in giving talks to the public. I practiced my defense multiple times and when I presented it to the audience, it seemed to work well. I only received a few questions and each committee member said I did a great job in presenting my dissertation. 

**_Mission Completed!_**

