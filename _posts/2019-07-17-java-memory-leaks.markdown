---
layout: post
title:  "Java memory leaks and how to avoid them."
date:   2019-07-17 10:52:45
categories: spring
---
What is memory leak?

When in memory there are objects that are no longer used by application but still Garbage Collector is not able to remove them as there are still references to them in application.
As a result, more and more resources are consumed, significantly slowing down your application, until they are missing and OutOfMemoryError is thrown.
Sometimes the error will not be thrown, and the only signal indicating memory leaks will be a significant and progressive slowdown of applications related to  GC load.
In that cases leaks are hard to find – either during debug or while using heap profilers.

![memory-leak](/memory_leak.jpg)


While reading  Effective Java (Third Edition), I have accumulated some good practices that will avoid memory leaks in the most common situations:

1.  Best practice: define variables in the smallest possible scope. - Why? Variable that is defined only within method body will be removed by GC after each method usage, won’t be living the whole class lifecycle.

2. Unragistered callbacks: When you register a callback, but you do not register it directly, they will accumulate until you perform some action. One way to guarantee that they will be collected by the garbage collector as they are no longer needed is only their weak references (eg WeakHashMap)

3.  Do not declare large blocks of code as 'static', because it will make Garbage Collector never delete them.

4.  Don't use finalizers

5.  Close SQL connections or files.

6.  Consider removing the values ​​from the cache that have been in it for a relatively long time without being used.

7.  In fact - about caching: At one time during the Geecon conference I met with a strong claim with which I agree 100%:

8.  Use WeakHashMap to write the cache (instead of the usual Hashmap), which holds the object in the cache only for the time when there are external references to the key of this object.
Never implement your own cache mechanism.
There are ready-made, great solutions for different applications (about the next blog entry). Libraries such as Guava Cache, Redis or Spring Cache are adapted to prevent memory leaks.

9. Every non-static Inner Class has, by default, an implicit reference to its containing class. If we use this inner class’ object in our application, then even after our containing class’ object goes out of scope, it will not be garbage collected.	If the inner class doesn’t need access to the containing class members, consider turning it into a static class

10. Don’t use unnecessary auto boxing in loops.

OK, but I already have memory leaks (probably, application is sloooooow) what do I do?

1. Enable Profiling
Java profilers are tools that monitor and diagnose the memory leaks through the application. They analyze what’s going on internally in our application — for example, how memory is allocated.
Using profilers, we can compare different approaches and find areas where we can optimally use our resources.
2.  Memory Leak Warnings – for example IntelliJ has them, please don’t ignore them ;)



