---
title: "Garbage collectors and Memory Leaks in Nodejs - V8 [re-up]"
datePublished: Wed Nov 09 2022 17:00:00 GMT+0000 (Coordinated Universal Time)
cuid: clq6xe5mo000409l67wfq4d5y
slug: garbage-collectors-and-memory-leaks-in-nodejs-v8-re-up
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1702661492780/1d46069f-4557-4854-b3bf-536b1bd8f259.png
tags: javascript, nodejs, typescript, v8, v8-engine, garbagecollection

---

On 10 Nov 2022, I hosted a workshop to share about "Garbage Collection and Memory Leaks in Node.js V8" at [TIKI](https://www.linkedin.com/company/tiki-vn/)  
  
Slide URL: [https://www.slideshare.net/LyLuongThien/garbage-collectors-and-memory-leaks-in-nodejs-v8](https://www.slideshare.net/LyLuongThien/garbage-collectors-and-memory-leaks-in-nodejs-v8)  
Youtube Video (Vietnamese) on Tiki Developer: [https://youtu.be/u3Dvh-ofzNg](https://youtu.be/u3Dvh-ofzNg)  
  
The detailed agenda is:  
1\. V8  
\- about V8 and comparing V8 with alternatives  
\- V8’s compiler pipeline  
2\. About V8 Garbage Collection:  
\- 2 Collectors: ▶ Major GC (Mark-Compact) & Minor GC (Scavenger)  
\- Hidden class  
\- Inline Cache  
\- Hot function  
3\. Some Best practices with GC with my real-world experimentation  
  
4\. Practising Memory debugging  
\- Profiling frontend app memory with Memlab framework from Facebook  
\- Profiling backend app memory with Chrome Devtools and node-inspector  
\------------