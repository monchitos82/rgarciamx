### Application bottlenecks (08/25)

I suck at _Java_. I have never written production grade _Java_ code. But I have gotten to understand the _Java_ virtual machine (_JVM_), especially because I was mentored by brilliant people who understand the language and the virtual machine inside out. I think one of my best opportunities back at my previous job was around understanding bottlenecks and tuning a _JVM_. Tuning a _JVM_ is not something we should do out of ignorance though; the _JVM_ defaults are quite effective when dealing with regular problems. I will not write a guide on how to profile projects or picking flags to tune the _JVM_, especially when it is something I am not doing on a daily basis, however I wanted to share some advice on getting started with profiling and understanding performance from more than a _CPU_ utilization graph perspective.

I should start by mentioning that utilization of resources is not the same as saturation; if you see a _CPU_ is being used over 80% that is not an indicator of trouble, quite the contrary, however, if you throw in the mix latency concerns and garbage collection, well, your problem (if any) turns interesting.

One of the things I learned when working with _cgroups_ and old _JVM_ versions is that understanding the limits of an application is basically the first and only thing we need to do when fitting a process to its assigned resources. There are interesting reads about _JVM ergonomics_ that will suggest we want to let the virtual machine handle the hairy parts, and it is correct for the most part of it, yet understanding is non trivial when containers are involved. It's also important to understand the architecture of a program to figure out how and why it solves a problem the way it does.

So let's say you have a simple program that scans an array of objects and uses a library to rank them and build a recommendation. This sounds like something that would be _CPU_ intensive but manageable, except when you run it you notice that latency randomly spikes and you don't see how or why. Looking at logs is not effective since you have no errors coming, just latency, then what?

I dare to say, without hesitation, this was one of my favorite parts of being an _SRE_. Figuring out what's affecting a service and how to push it to perform better is not a one man kind of task, especially if you didn't write the code or have access to it for a start. There is a great set of tools at your disposal and understanding when to use them is key.

1. Graphs - It sounds contradictory to mention graphs just after I said that graphs will not get us somewhere in the investigation, however, if well instrumented, graphs are quite useful resources, especially when they can overlay. Performance problems (_ideally_) are non-persistent, but influenced by outstanding factors. A graph is a good way to understand when and where the problem starts, yet these are not flawless, if instrumentation is poor and there are no good metrics, all we may see are spikes at system levels, that's not useful to understand load as a source but as a consequence.
2. Profilers - Profiling a process is a good way to understand a few things in a reasonable time window: workflows, interruptions, processing load and dependencies to mention a few. When doing this in the _JVM_ there is another thing we get to see: the garbage collection (_GC_). This is very important and can be paired with an amazing tool called _flamegraphs_, we get to see duration of a process and the depth of dependencies in a call; this alone may not be very insightful of a problem, but it is very insightful on how resources are utilized and helps us gain context on how a program executes and what a program is calling. In the example above, you may not see a problem in the graph itself, especially when profiling _CPU_ utilization, so how to spot it? I think there are two sensitive ways: a) pay attention to the duration of a method execution, is it short some times and very long another? and b) pay attention to the _GC_ cadence. Is it suddenly too frequent? Too long? is that graph full of yellow bars (or the color scheme you pick for _GC_)? There is your clue.
3. Memory dumps - If you get to understand the flow of a program, you may get to see what is that program doing when a problem arises. A memory dump is especially useful when that happens. Having insight on the amount of objects, their type and their size is especially useful when detecting a bottleneck. Going back to the example above, if your program fails only when scanning objects containing 4K image files when it is designed to scan thumbnails, well, it makes sense that your program struggles; but sometimes it is less obvious than that, sometimes, it boils down to using objects like boxed numbers or `arrayList`; objects that improve the developer's ergonomics (lazy evaluation) at the expense of memory footprint that over time will become wasteful and saturate pages at a staggering speed, sometimes all you have to do is go back to using primitives or just correcting the size of a heap because you can't fit objects properly. Getting insight on that could be hinted by the profiler, but is not coming until you are at this step, then it actually makes sense to think about solutions like tuning the virtual machine or scaling vertically.
4. Profilers (again!) - I mention profilers again. Just like with overlaying graphs, looking at a program behavior under regular circumstances and saturation can be insightful on how the program behaves on the latter. Sometimes a _flame graph_ will show it to us right away.
5. Source code - I think this is especially useful when we think that our resource allocation is correct and that we see that tuning has no effect, why? because performance problems should ideally be solved at the code level. Improving our code performance is key on improving our resource utilization, not the other way around, most experienced engineers understand this, except that sometimes the re-engineering cost is too high, so making room is faster disregard on how ugly it may get.

This feels like I am just scratching the surface and want to speed run through the topic. I realize this conversation is not easy because not everyone starts at the same level and depth into the conversation really depends on the problem and the tools. So I would like instead to offer some resources and advice if this is the first time you are dealing with something like this:

1. Use the <a href="https://sre.google/books/" target="_blank">_SRE book_</a> to get familiar with concepts like what has been mentioned here, saturation? observability? it is all in there
2. Understanding the <a href="https://docs.oracle.com/en/java/javase/21/gctuning/ergonomics.html" target="_blank">_JVM ergonomics_</a> is a good read
3. Understanding the <a href="https://docs.oracle.com/en/java/javase/21/gctuning/available-collectors.html" target="_blank">_GC types of collectors_</a> and tuning flags is non-trivial but it is one very important thing to do. Memory management and _GC_ frequency are _knobs_ that you can play with, but ideally you need to understand well first. When in doubt, fall back to the defaults, understand the _GC_ periodicity as a necessary sacrifice of performance in exchange of throughput
4. Understanding profilers like <a href="https://github.com/async-profiler/async-profiler/blob/master/docs/GettingStarted.md" target="_blank">_async-profiler_</a> and <a href="https://www.brendangregg.com/flamegraphs.html" target="_blank">_flamegraphs_</a>. to me was a great investment of time. I used to advice using **_Minecraft_** to learn how to do profiling, I still find it quite useful; sadly, Mojang has made quite cumbersome understanding the flamegraphs (too much obfuscation in their classes nowadays, thank you Microsoft 😔).
5. Using <a href="https://eclipse.dev/mat/" target="_blank">_MAT_</a> the first time can be intimidating, yet it is your best **free** option. I was fortunate enough to get <a href="https://jxray.com/" target="_blank">_J-Xray_</a> taught by its author, however it is not a free tool. Regardless, you may want to read their blog and articles, that amount of wisdom for free is a bargain!
6. Understand how to write performant code. Understand the _API_ of your language and **never** overlook simplicity

I think the list above could rapidly grow the more we delve into specifics, my advice, read as you go and try to be one step ahead in the knowledge. Many people focus on finding tools to rightsize containers and forget the basics like _Big-O_ notation or design principles. This of course is not the end of the conversation, and for the sake of brevity I am shying away from discussing off-heap allocation or _JNI_ invocations, but I hope you get the gist, we need to look at our performance journey through a funnel:

<pre>
\=======================================================================/      ---
 \=====================================================================/        |
  \                          Resource definition                      /         Environment adjustments (e.g. full memory allocation) and vertical scaling
   \                         (Container settings)                    /          |
    \---------------------------------------------------------------/          ---
     \                       System metrics                        /            |
      \                      (Monitoring)                         /             |
       \---------------------------------------------------------/              |
        \                    Application configuration          /               |
         \                   (Settings)                        /                |
          \---------------------------------------------------/                 Performance bottlenecks identification
           \                 Application metrics             /                  |
            \                (Monitoring)                   /                   |
             \---------------------------------------------/                    |
              \              CPU utilization &#8635;            /                     |
               \             (Profiling)                 /                      |
                \---------------------------------------/                      ---
                 \           Memory footprint          /                        |
                  \          (Dump analysis)          /                         |
                   \---------------------------------/                          Design and implementation bottlenecks identification and optimizations
                    \        Code analysis          /                           |
                     \                             /                            |
                      \---------------------------/                            ---
                       \     Running environment /                              |
                        \    tuning             /                               JVM and kernel fine tuning
                         \=====================/                                |
                          \===================/                                --- 
</pre>

Final, and not less important advice I can share: Measure. Load testing and metric analytics are cornerstones of a good service health, not paying attention to this, will not spare you the above, but **it will save you from executing it with high urgency**.

---
<div align="center">
  <a href="../../indexes/2025.md" title="Back to year's index">📖</a>
</div>