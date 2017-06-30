---
layout: post
title: More about JVM memory management
description:
headline:
modified: 2016-01-16
category: tech
tags: [update, java, jvm]
image:
---
One of the things that bothers me the most about Java is its memory management system, and by extension how Java developers approach problems when they hear about how memory is managed.  Several of the Java developers I’ve discussed Java memory management with have said, in not so many words, that Java memory management is “magic”.  To those of you who believe this, you're the worst.  Nothing is magic, there are only tools which make software development easier.  While you don’t have to manage specific memory addresses with malloc() and free(), as a developer you still have to be aware of what lives in memory, how much memory you are using, and what you want on tap all the time as well as what you don’t need.  Admins who manage the JVM for long enough tend to understand the gist of that, since they’re usually the ones restarting the service when it runs out of heap, but they don’t always understand the costs of the JVM doing malloc()s to grab heap space on the fly, rather than allocating your whole heap on service initialization, or what conditions should *really* trigger an FGC.  Much of this information really only applies to reactive programs; scripts and batch jobs don’t matter nearly as much since the speed of a batch job doesn’t necessarily impact how fast users perceive your app to be.  Most admins have a firmer grasp on garbage collection in the JVM since it’s frequently their job to tune it, but I’ll reiterate how the various garbage collectors work, and share my opinions on what’s the best for what type of application.

For an introduction to the various garbage collectors at your disposal, the JVM tuning guide puts it most concisely:


> Unless your application has rather strict pause time requirements, first run your application and allow the VM to select a collector. If necessary, adjust the heap size to improve performance. If the performance still does not meet your goals, then use the following guidelines as a starting point for selecting a collector.


> * If the application has a small data set (up to approximately 100 MB), then
select the serial collector with the option -XX:+UseSerialGC.
> * If the application will be run on a single processor and there are no pause time requirements, then let the VM select the collector, or select the serial collector with the option -XX:+UseSerialGC.
> * If (a) peak application performance is the first priority and (b) there are no pause time requirements or pauses of 1 second or longer are acceptable, then let the VM select the collector, or select the parallel collector with -XX:+UseParallelGC.
> * If response time is more important than overall throughput and garbage collection pauses must be kept shorter than approximately 1 second, then select the concurrent collector with -XX:+UseConcMarkSweepGC or -XX:+UseG1GC.

> These guidelines provide only a starting point for selecting a collector because performance is dependent on the size of the heap, the amount of live data maintained by the application, and the number and speed of available processors. Pause times are particularly sensitive to these factors, so the threshold of 1 second mentioned previously is only approximate: the parallel collector will experience pause times longer than 1 second on many data size and hardware combinations; conversely, the concurrent collector may not be able to keep pauses shorter than 1 second on some combinations.

Pretty much covers all common scenarios.

Most of these GC types are pretty self explanatory, and only apply to full GCs, not minor GCs.  The serial garbage collector pauses code execution, goes through the objects in heap memory one by one to see if they can be purged.  Once it goes through all of the objects, code execution resumes.  The benefit of this collector is that there’s very little overhead, because it’s very simple.  The parallel garbage collector does something similar, but threaded to run better on multiprocessor systems.  Then we get to my former, Concurrent Mark and Sweep.  The reason it’s my favorite is because it’s got the shortest GC pauses of the bunch.  The rest of the methods pause when they’re looking for stuff to prune, and continue to pause while they do the actual pruning.  The short version is that concurrent mark and sweep pauses to mark objects still in use, and then unpauses to actually get rid of the old objects.  You can still end up with some pretty gnarly pause times, but the same collection with CMS is shorter than parallel or serial.  The reason CMS is my old favorite is because they introduced the G1 collector in Java 7 Update 4.  G1 ( which stands for Garbage First) allows you to set the max pause time you want to allow, and the collector goes through as much memory as it can in that time.  This allows reactive applications to stay within their response time SLAs, even if they’re going through a full garbage collection.  Unfortunately, there’s still a pause, but that’s the price you pay for not having to allocate and deallocate memory references yourself.  Whether that’s an acceptable trade-off in your environment is up to you.

None of this is an exact science.  Tuning the JVM involves a fair amount of fiddling.  There are drawbacks to every configuration, it’s just a matter of what you value most for your use case.  For more comprehensive advice, check out the tuning guide for the HotSpot JVM.
