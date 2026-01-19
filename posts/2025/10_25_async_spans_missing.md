### When the spans are missing in a trace (10/25)

I wish I could say how to fix what I am going to present here, however, so far I have not solved the problem and I think the only viable solution I can offer since I have not written my own <i>FastAPI + SQLAlchemy</i> implementation, is to defer the operations.

Well, now that my preface offered a shitty warning. Here is the complete scenario:

Observability -well done- is not only about creating a cardinality nightmare in your metrics, or a slob of events choking an <i>ingestor</i>. It is about having or creating the means to look at what the hell is going on within your service when it is processing information. This used to be something doable when we had one machine, access privileges and some time to read through our code and system calls. That's no longer the reality. Still, we have a way to actually inspect our system operations and figure out when, how, for how long something runs so we can answer why it runs and why it fails when it fails. Unlike metrics which basically only offer us trends for us to infer what happens, or logs that can hinder the execution of a process, are hard to navigate or simply are not verbose (since what the developer chooses not to show is not shown), traces are slightly better to present the story before we take the deep dive. (Sorry if I am oblivious to profiling tools using <i>eBPF</i> like <i>Pyroscope</i>, I love profiling, but I am sure it needs its own post)

So this is me ranting on a weird use case that makes me think that not many people are actually looking at things close enough.

<i>FastAPI</i> suddenly became the go-to framework for <i>Python</i> web development. I prefer this a million times over <i>Django</i>, but maturity is no joke when we talk about production scenarios and well, I suddenly found myself in a pickle here, not because the framework is to blame, but because the key technology the framework relies on seems incomplete for my scenario. No framework is perfect and probably I didn't <i>RTFM</i> thoroughly.

For learning purposes, some time ago I created a service that basically serves bogus data, but each time a request comes, it stores it in a backend (a database for this case). I use <i>open-telemetry</i> to track my operations since I defer the initial call to a random endpoint, this because a single-span call-tree is boring. If I am getting too specific into this topic, I am sorry non-technical reader, Web today is complicated.

My initial implementation was working well, since I used <i>Flask</i> first. I know calls in that framework are blocking, but even then, when I plugged <i>locust</i> to load-test my service, I was able to do slightly more than 500<i>QPS</i>. This number is peanuts compared to a high throughput, low latency service, but this is an experiment after all. 

I thought that I could do better, I still think this service can take far more (about 2 to 3 thousand <i>QPS</i>), but it requires <i>HTTP/2</i> and <i>maybe</i> asynchronous calls (Mind that even with the switch and using <i>FastHTTPUser</i> there is no protocol switch, just more concurrency in the requests). That's when I decided to use <i>ASGI</i> instead of <i>WSGI</i>. But even if I changed the server and the execution, I still had a blocking backend.

At this point, every trace would look just fine in <i>Jaeger</i> though. I could send a request and all the flow, including cursor operations were present in the spans. I was happy with this.

Snooping through what is being used lately, suddenly <i>Flask</i> is no longer the go-to tool for this. I think I wrote about this before, so migrating this whole cake of calls required some thought and some work here and there, but much of what I used is present in <i>FastAPI</i> flavors, including <i>SQLAlchemy</i> extensions. So it was not so hard.

What was not so easy, though, was using <i>asyncio</i>, not because the library is hard, but because it requires you to think differently about the order of a request and the objects the request uses. This may sound simple, but when you try migrating the unit tests, it is a mess.

When I got things working, I was happy with the way this went. When I had a look at my first trace, I was not. My trace was missing all database operations. I had the connection, and from there, all went silent.

I think this is in part, because I used a different driver, a different engine and basically a different way to work with the calls and the database; it is not intuitive, it is not meant to be, but I still cannot figure out where the span is and why it is not in my trace. I want to feel stupid here, I think I missed something, reading through <i>PiPy</i>, StackOverflow and Medium, I find nothing off with my code, still this is not there, but well, this is also pointing me to a design flaw in my project:

<pre>
<code class="language-python">
async def log_request(request: Request, latency: float, db: AsyncSession, status_code: int = 200):
    """ Write the request details into the database """
    log_data = RequestLog(
        ip_address=request.client.host,
        endpoint=request.url.path,
        method=request.method,
        response_code=status_code,
        timestamp=get_now_time(),
    )
    db.add(log_data)
    await db.commit()

    REQUEST_LATENCY.labels(request.url.path).observe(latency)
    REQUEST_COUNT.labels(request.url.path, request.method).inc()
</code>
</pre>

Any endpoint "logging" through this function will have to send a call to the database. This operation if deferred in any other endpoint should be returning an accepted response rather than being handled as blocking. When I say I can fix this via architecture, I mean that this write operation is something that needs to be done by an <i>ingestor</i> on a different stage of the workflow. I am paying the price (both in performance and observability) for that mistake. But well, this is feedback for me after all.

I plan on keep trying to get this span fixed, although I suspect the instrumentation is just not there. I am happy to find this, not only because it points me to a design error I made, but because it presents me with a hypothesis: Nobody is actually looking at how things are working under the hood and they don't seem to care, because they are not solving these problems. Whatever the reason is, being the first to hit this, even if driven by my own stubbornness or idiocy is great, some parts of your toys aren't always exposed and you will never learn how they work until you break them apart.

<pre>
How my system works (not ideal):
                    ┌─────────────────────┐
                    │       Hypercorn     │
                    └──────────┬──────────┘
                               │
              ┌────────────────┴────────────────┐
              │                                 │
              v                                 v
     ┌───────────────────┐            ┌───────────────────┐
     │  simple-service   │----------->│  simple-service   │
     │      /data        │            │      /random      │
     └───────────────────┘            └───────────────────┘
              └────────────────┬────────────────┘
                               │
                               v
                        ┌──────────────┐
                        │   Postgres   │
                        │   (async)    │
                        └──────────────┘
     └─────────────────────────┬───────────────────────────┘
                        ┌──────┴───────┐
                        │    Jaeger    │
                        │  Prometheus  │
                        └──────────────┘
</pre>

#### It's all in the middleware! (10/25)

I was not planning to edit this soon, but here we go!
I found the issue, I found why this was all messed up and while the fix is simple; the design flaw is still important because it blocks the service from scaling.

The issue was that using <code>fastapi_sqlalchemy</code> extension would add an unnecessary middleware object. This middleware is probably hijacking the session and doing the database operations isolated from the process. It sounds nice that just removing some session configuration would fix my problem, but it is not so easy, because in my unit tests, I am using <i>sqlite</i>. This creates a different kind of problem: Part of my settings are built around <i>asyncpg</i>, but my <i>Postgres</i> driver has no use for the tests. I can circumvent this problem using a synchronous setting (basically, keeping my old settings from the <i>Flask</i> project), but if I do so, my test is no longer apples to apples for the <code>log_request</code> method.

Regardless, I fixed it! But my service regressed 100<i>QPS</i> 😞