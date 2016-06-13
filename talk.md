The voice line to the presentation
======

# Step-1
* customer speak only about runtime performance, because they don't really care about memory
* **as long as** the software
 * don't crash
 * can run on target hardware
* allocating and freeing memory also need time

# Step-2
## Kiss Principle
* easy code can be
 * understud really fast
 * can be tested
 * can be optimized
 * is usually fast code too
* see [Clean Code Developer - GERMAN](http://clean-code-developer.de/die-grade/roter-grad/#Keep_it_simple_stupid_KISS)

## **R**oot**C**ause**A**nalysis
We all know one colleague, if something crashes because of NullReferenceException who will simply add if (x != null) condition. But this is only a good choice if the object is allowed to be *null*, in every other case it is better to figure out, why it is null.  
* even more true while talk about performance
 * **crasher** usually not the cause of a performance problem
  * maybe the memory is fragmented 
  * maybe the window handles are already out
  * ...

## 50k theorem
Developer of small applications / Apps can now think about something nice. All other listen carefully ;)
* As professional software developers you write code that work (is testable... is well designed .. is easy to understand and so on) but
 * think about input data/objects for at least 50000 elements, does your alogirthm really perform in that case? 
 * maybe you shouldn't traverse the collection for each object
 * maybe you can avoid expotential running time
 * maybe you can avoid creation of data container for each element, if only 10% of the elements are relevant

# Step-3
## Don't Optimize up front
There is no contrast to the 50k theorem, because here I like to point out that each higher level performance optimization should have a measurement which is showing the problem.
* around 50% of caches I saw in daily work where either
 * way to complex
 * don't really work in each cases (error prone!)
 * wasn't faster than the code without caching
* around 20% of the caches, where **slower** because of caching management and software bugs

## Investigate only if necessary
Points to topic above, only investigate performance problems if you  have identify a performance issue. And this mean **measure, measure, measure**

## Learn you language
"To be a good developer has nothing to do with the used language", a sentence which you can hear often. It is true in case of object orientation, in case of design principles or quality but **not necessary** for performance because:
* .NET CLR is different from version to version (ArrayList vs Generics, Inlining, etc.) 
* if using higher level functionality IEnumerable, yield, LINQ you should understand what CLR does
* in case of doubt
 * ILDasm, ILSpy, dotPeel, Reflector 
 * WinDBG ^^
 
#Step-5
Microsoft learning book suggest to increase object size, to be bigger than 85K, because you "save" time to promte objects thru different garabage collector generations because it is directly allocated on Large Object Heap. 
Today we know you should avoid LOH... maybe next version we can use it again?!
There are a lot ob know-how on different websides which compare different source code, which achive the same goal.
Please retest with current version, it is may not true any longer (due to compiler/clr optimization).

#Step-6
To decide if there is some performance problem you need an expectation how long it should take.
* Categorize the implementation into an category like
 * implementation should need same amount of time for each input objects
 * implementation should need constant time indpendend from input objects
 * implementation should need smae amount of time for each relevant object, in average only 10% of input objects are relevant
* measure for 1 object the time
* create expected value list for 1, 10, 100, 1.000, 10.000, 100.000 input objects
* measure the real timings for 10,100, 1.00, 10.000, 100.000 input objects
* compare expectation and results --> does it match?
* **of course you should do the same for memory consumption** 

#Step-7
--> explain measurement "frame" (left side on slide)
*Question:* Are both way's of matrix traversion equal or is one faster (which one?)

#Step-8
*Answer:* Row traversation is fast, because memory preload (less CPU L2 cache misses)

#Step-9
*Question:* Concating strings, how fast the different versions?

#Step-10
*Answer:*
1. StringBuilder
2. IEnumerable + Join
3. concating in loop 
4. by Hand

#Step-11
--> explain the sample
Question: Is there an performance problem?
hint: struct is value type and is allocated on stack

#Step-12
Answer: Yes there is a big bug, structs in hash collections are really slow
--> default GetHashCode implementation causing a lot of hash collusion 

#Step-13
Garbage Collector split comlete memory into tree parts
##Gen0: 
* all objects are allocated here
* for short living objects
* Gen0 garbage collection promote objects to Gen1 or free

##Gen0:
* all objects which survive one garbage collection
* Gen1 garbage collection promote to Gen2 or free
* approximate Gen0 collections are 10 times more often than Gen0 collections

##Gen2:
* for long living objects
* LOH exist here

#Step-14
* Large Object Heap contains all objects bigger than 85KB
* it will no defragmented (compacted automatically)
 * there are examples that you can have OutOfMemory with only 4 MD memory referenced (on 32Bit) based on fragmentation
* before .NET 4.5.1 --> you're in trouble
* since .NET 4.5.1 you cann call defragmentation by hand (see link)
 
#Step-15
* GC works with mark & sweep algorithm
* mark phase
 * GC traverse from each root down to all accessable objects and mark them
* sweep phase
 * depending on GC generation objects which are marked
  * will be promoted to the next higher GC Generation
* not marked objects will be
 * add to finalizer queue if they have finalizer implemented
 * or removed from memory
* maybe the collected generation will be compacted

#Step-16 
Problem: object clouds
* garbage collection for complexe object sturcture are working fine, until there is one (or more) root chains.
* it happens offen that a bundle of objects can't be collected because on object register to
 * Language-Change-Setting
 * dynamic (static) collection
 * or something like that
* key message: GC will only work, if you think about object lifetime  --> **when should I unsubscribe from events/callbacks and so one?!**

#Step-17
* memory management is also runtime consuming, the GC works asynchronous as much as possible but it have to freeze the working threads
* with more garbage collections (at least Gen2 collection) the working threads will be stopped
* this will directly causing higher runtime to execute the program
* compacting the Generation need even more time (up to minutes) where your application freezes
 * especially if compacting the LOH
 
#Step-18
Example two ways of implementing read only object with ObjectChanged event (which will never called)
*Question:* Which implementation is better?
Hint: heading is a link to [Codeproject](http://www.codeproject.com/Tips/876878/Calculating-Optimistic-Memory-Footprint-of-Managed), implementation to calculate object size at runtime

#Step-19
*Answer:* empty add/remove block
*Question:* Did you know that each event approximate need 90bytes of memory? It is not only the four byte pointer most (c/c++) people expect

#Step-20
In memory we see the reason why string concation with += is so slow, **5212 MB** of temporary memory was used (and need to be allocated and garbage collected)
* string are immutable objects, each change to a string causes
 * allocating memory with size of new string
 * copy old content to new memory address
 * *do* the change

#Step-21
in contrast StringBuilder only uses **2MB** of memory

#Step-22
Thanks for attention and time for Q&A session ;)

#Step-23
full list of links