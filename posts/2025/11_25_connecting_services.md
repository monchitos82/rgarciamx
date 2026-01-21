### Connecting my services (11/25)

This will sound like a boring technical post, full of configurations and rants, but actually it's about the unexpected realization that I have connected most of the projects I started independently and what each one of those is teaching me.

Last year as I moved back I experimented heavily with clusters in my own infrastructure, although my infrastructure is limited, I wanted to have a good setup to test products and ideas and to centrally store my data. I ended up giving the remote storage idea and dismantling my cluster, but I repurposed some machines to do some non-trivial tasks like block ads, do network analysis and even poll weather stats. So these machines are still hiding at the corners of my place.

From the projects I created and the services I deployed, I basically built an observability stack, imperfect, but it is there. I have said often that I am not a big fan of containers just because poor containerization makes hard to troubleshoot services, but since I own my code, I am using the good side of containers: isolation, so I can put as much as I can to run in my main box. Why? Because it is beefy enough and because I can run datastores there.

So basically I ended up with a bunch of services and images generating datapoints everywhere. So out of curiosity I decided to dig through them and take the chance to learn a bit more about data ingestion and processing. My setup didn't disappoint, I had several data stores already in place and I figured out the services I had could be connected through some pipelines to produce data, not useful data, because I don't do much with it, however, this is a learning opportunity.

So I had my monitoring services inserting data into my datastores and it was not hard to extend them to produce a bit more, so I instrumented them and in no time I had a full _OTEL_ compliant stack. If you instrument to produce to _Jaeger_ you might as well just replace it for the _OTEL_ collector, which I did and I was able to extend to different backends. So now I know about producers, collectors and exporters, on top of all I knew before about instrumentation.

Collecting data also serves me to feed data into ingestors. At this moment I don't write much to _Kafka_ to actually serialize the ingestion properly (I can do that later though), but I could connect the ingestors (using _Flink_) to my read replicas of _PostgreSQL_. This served me to actually learn about creating ingestors (I knew about it using _Samza_, but _Flink_ is less quirky as a cluster) and sinking data, which I took the chance of doing it using _MongoDB_.

Having this setup as I said, serves no purpose, I collect datapoints using _Prometheus_ plugins for the sake of building dashboards, but it is very useful for learning about: a) How to create scalable systems. b) How to create an end to end data pipeline. c) Assess the functionality of every piece I've written.

As result of this _toying with shinny blocks of software_, I ended up migrating some services to _Golang_, shelving other projects, and fully embracing my _Podman_ setup. It has been an interesting journey that it isn't over, where I have had the chance to see diverse disciplines of my industry. I lost the fear of meddling with some languages, I confirmed my disgust for others, I harvested a lot of the power _LLMs_ have to provide useful advice, I learned to ask better questions, I learned to dismiss bad ideas too.

In the end I am not changing much of what I already had in place; I am discovering that my basis is solid and that I could grow a lot having a bunch of great projects as foundation. This deepens my respect of open source products.

I am shying away from creating a roadmap on how to do things and how to setup these projects, because their purpose is to serve my needs. I can however produce a list of the many great projects I used to connect and produce all the things that today compose my stack at home:

* Apache Airflow (poll and produce weather data, trigger and orchestrate jobs)
* Git (repository for all projects, including this blog)
* Jenkins (Publish this blog, validate configurations)
* MariaDB (Backend for Grafana)
* InfluxDB (Store network metrics)
* Prometheus (Backend for all monitoring)
* Jaeger (Traces analysis of my code)
* Statsd (Monitoring Airflow)
* Suricata (Monitoring network internally)
* Pihole (Blocking tracking traffic and ads, a **must** in a house full of IoT)
* PostgreSQL (Store for all the backends I wrote)
* Apache Flink (Consumers for aggregating data from my datastores. I know, in batch Spark would be just as good if not better)
* MongoDB (Sink for the Flink data)
* Trino (Centralized querying)
* Redis/Valkey (Caching layer for some projects)
* Nginx (You need a web server)
* HAProxy (You need a web server, you don't need a _specific_ one)
* Grafana (Because visualization of data is golden)
* ElasticSearch (Indexing my logs)

I was not expecting to put this post together; it adds no value to my writing, but it is a recap of all the projects I ventured into for the sake of learning and producing useful data like knowing about my network speed, my energy consumption, my weather and helping this writting go live via _Neocities_ 💜.

<pre>
┌─────────────────────────────────────────────────────────────────────────────┐
│                            EXTERNAL TRAFFIC                                 │
└────────────────────────────────────┬────────────────────────────────────────┘
                                     │
                                     ▼
                          ┌──────────────────┐
                          │     HAProxy      │
                          │  (Load Balancer) │
                          └────────┬─────────┘
                                   │
                                   ▼
                          ┌──────────────────┐
                          │      Nginx       │
                          │ (Config Server)  │
                          │  - CSV files     │
                          │  - Prom configs  │
                          └────────┬─────────┘
                                   │
         ┌─────────────────────────┼──────────────────────┐
         │                         │                      │
         ▼                         ▼                      ▼
┌─────────────────┐      ┌─────────────────┐      ┌─────────────────┐
│   Prometheus    │      │    Grafana      │      │     My own      │ 
│  (Metrics DB)   │◄─────│ (Visualization) │      │    Services     │
│                 │      │                 │      │                 │
└────────┬────────┘      └────────┬────────┘      └────────┬────────┘
         │                        │                        │
         │                        │                        ▼
         │                        │                ┌─────────────────┐
         │                        │                │ OTEL Collector  │
         │                        └───────────────►│  (Telemetry)    │
         │                                         └────────┬────────┘
         │                                                  │
         │                                         ┌────────┴────────┐
         │                                         │                 │
         ▼                                         ▼                 ▼
┌─────────────────┐                      ┌─────────────────┐ ┌─────────────┐
│     StatsD      │                      │     Jaeger      │ │Elasticsearch│
│ (Airflow Stats) │                      │    (Traces)     │ │   (Logs)    │
└─────────────────┘                      └─────────────────┘ └─────────────┘


┌─────────────────────────────────────────────────────────────────────────────┐
│                        WORKFLOW & ORCHESTRATION                             │
└─────────────────────────────────────────────────────────────────────────────┘

         ┌─────────────────┐              ┌─────────────────┐
         │   Git Service   │──triggers───►│     Jenkins     │
         │                 │              │  + Builder Node │
         └─────────────────┘              └─────────────────┘

                          ┌──────────────────┐
                          │     Airflow      │
                          │ (Orchestration)  │
                          └────────┬─────────┘
                                   │
                          ┌────────┴────────┐
                          │                 │
                          ▼                 ▼
                   ┌─────────────┐   ┌─────────────┐
                   │   Valkey    │   │  Postgres   │
                   │ (Task Queue)│   │ (Primary +  │
                   └─────────────┘   │ Replica DB) │
                                     └──────┬──────┘
                                            │
                                            │ replication
                                            │
                                            ▼
                                    ┌────────────────┐
                                    │ Postgres Data  │
                                    │  (Services +   │
                                    │   Airflow BE)  │
                                    └────────┬───────┘
                                             │
                                             ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                      DATA PROCESSING & ANALYTICS                            │
└─────────────────────────────────────────────────────────────────────────────┘

         ┌─────────────────┐                      ┌─────────────────┐
         │  Flink Job      │────aggregates───────►│    MongoDB      │
         │ (Stream Proc.)  │                      │(Aggregated Data)│
         └─────────────────┘                      └─────────────────┘
                  ▲
                  │ reads from
                  │
         ┌────────┴────────┐
         │    Postgres     │
         └─────────────────┘


         ┌─────────────────────────────────────────────────────────┐
         │                       Trino                             │
         │              (Federated Query Engine)                   │
         │   Queries: Postgres, MongoDB, InfluxDB, ...             │
         └─────────────────────────────────────────────────────────┘
                   │          │           │            │
           ┌───────┘          │           │            └────────┐
           ▼       │          ▼           ▼                     ▼
    ┌───────────┐  │   ┌──────────┐ ┌───────────┐      ┌──────────────┐
    │  Postgres │  │   │ MongoDB  │ │ InfluxDB  │      │Elasticsearch │
    └───────────┘  │   └──────────┘ │ (Network  │      └──────────────┘
                   │                │Ping Stats)│
                   │                └───────────┘
                   │                       ▲
                   │                       │ stores
                   │                       │
                   │                ┌──────┴───────┐
                   │                │ Network Ping │
                   │                │   Monitors   │
                   │                └──────────────┘
                   ▼
         ┌─────────────────┐
         │    MariaDB      │
         │ (Grafana Data)  │
         └─────────────────┘


┌─────────────────────────────────────────────────────────────────────────────┐
│                        SHARED INFRASTRUCTURE                                │
└─────────────────────────────────────────────────────────────────────────────┘
         │                  │                  │                     │
         ▼                  ▼                  ▼                     ▼
  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐      ┌─────────────┐
  │    CUPS     │    │     NFS     │    │  Suricata   │      │ Ollama      │
  │  (Printing) │    │(File Share) │    │  (Network   │      │ (Local LLM) │
  │             │    │             │    │  Monitoring)│      │             │
  └─────────────┘    └─────────────┘    └─────────────┘      └─────────────┘
</pre>

Bonus: I tried to replicate this infra by asking Claude to get me a tutorial, I think while imperfect, the results are quite interesting.<br/>
<a href="../../snippets/otel_flask_blog.html" target="_blank" title="Read tutorial">
tutorial for a simple service and its observability stack
</a>

---
<div align="center">
  <a href="../../indexes/2025.md" title="Back to year's index">📖</a>
</div>