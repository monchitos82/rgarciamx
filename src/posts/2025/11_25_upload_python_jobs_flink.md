### Uploading Python jobs to flink through the JAR API (11/25)

Let me start this post by making some leveling in expectations. This post is not focused on how to build jobs for _Flink_ or what is the right strategy to make these stream tasks. It isn't either a bragging one. I just figured out this information is not available anywhere that I searched for.

I have stated before that I am terrible at _Java_. I remember once a manager jokingly dismissed my work because I delivered a _PHP_ solution rather than a _war_ file because I could not write _Java_. He wasn't wrong. And to this day, I still have a hard time writing good _Java_ code. So when I wanted to do some _Flink_ work, I was happy when I found that it supports _Python_. But the support is far from perfect, and well, _Python_ has its own flaws. Anyway, it took me some hours to get my job running right because I chose the _latest_ build of my container and the driver is not very gentle giving feedback when something goes wrong. A lot went wrong.

My job is simple:

It queries _PostgreSQL_ and performs an aggregation (batch), then sinks the aggregated data into _MongoDB_. Nothing more, nothing less. I know I don't make use of _Flink's_ full potential by not even streaming data, but I have not much to produce for this scenario.

<pre>                                                                     
  ┌──────────────┐                                                             
  │  PostgreSQL  │                                                             
  │   Database   │                                                             
  │              │                                                             
  │ simpleservice│                                                             
  │   requests   │                                                             
  └──────┬───────┘                                                             
         │                                                                      
         │ JDBC Connector                                                      
         │ (Read full table)                                                   
         │                                                                      
         ▼                                                                      
  ┌──────────────────────────────────────────────────────────────┐            
  │              FLINK TABLE ENVIRONMENT (Batch Mode)            │            
  │                                                              │            
  │  Source Table: request_logs                                  │            
  │  ┌────────────────────────────────────────────────────────┐  │            
  │  │ id            INT                                      │  │            
  │  │ ...           STRING                                   │  │                  
  │  │ timestamp     TIMESTAMP_LTZ(3)                         │  │            
  │  └────────────────────────────────────────────────────────┘  │            
  │                          │                                   │            
  │                          │ WHERE timestamp >= CURRENT - 1hr  │            
  │                          ▼                                   │            
  │  ┌────────────────────────────────────────────────────────┐  │            
  │  │          AGGREGATION (GROUP BY something)              │  │            
  │  │                                                        │  │            
  │  │  • COUNT(*) as request_count                           │  │            
  │  │                                                        │  │            
  │  └────────────────────────────────────────────────────────┘  │            
  │                          │                                   │            
  │                          ▼                                   │            
  │  Sink Table: agg_requests                                    │            
  │  ┌────────────────────────────────────────────────────────┐  │            
  │  │ endpoint       STRING (PRIMARY KEY)                    │  │            
  │  │ request_count  BIGINT                                  │  │            
  │  └────────────────────────────────────────────────────────┘  │            
  └──────────────────────────┬───────────────────────────────────┘            
                             │                                                 
                             │ MongoDB Connector                               
                             │ (Upsert by endpoint)                            
                             │                                                 
                             ▼                                                 
                  ┌──────────────────┐                                        
                  │     MongoDB      │                                        
                  │                  │                                                                            
                  │  agg_requests    │                                        
                  │   collection     │                                        
                  └──────────────────┘   
</pre>

I could get this working in v.1.19 by running the _Flink CLI_, but this requires using the container rather than doing this remotely, so if I wanted to trigger this from something like an _Airflow DAG_ it would just not work. I had to find a way to get this working. The solution I found (and where I made great use of _Claude_) was wrapping things in a _JAR_. But this required learning _Maven_, doing some _Java_ work and trying to get things to run. Well, I found a way to get the _POM_ ready and put all my dependencies in the libs directory, but even then all I could trigger were exceptions and my container died.

I had to do some introspection into what my job was doing wrong and if that's fixed in newer versions, and even if there is a proposal for an `artifacts` endpoint, _Python_ support is limited and my job could not run.

After some back and forth, I tried wrapping the _Python_ code in a _JAR_ without using _Maven_, but it still failed:

<pre>
# Basically wrote the MANIFEST to point to the python dir and wrap the .py file, then pack and send
jar -cfm python-app.jar META-INF/MANIFEST.MF python/
</pre>

This produced nothing useful but exceptions. Then I had a Bingo! moment. My initial exceptions were around the driver not being able to find my script, because somehow, the driver assumes the file is in a `/tmp` location and it copies it before invoking `Flink`, so what if there is no need to copy?

<pre><code>
2025-11-28 22:31:54,061 WARN  org.apache.flink.client.python.PythonEnvUtils                [] - Create symbol link from /tmp/pyflink/4cab4606-4d40-423c-9da2-110f4d7858b9/ff3fe4a6-f89a-4fec-a849-5ececc23caee/flink-job-2466975472410535843.py to /tmp/flink-job-2466975472410535843.py failed and copy instead.
�java.nio.file.NoSuchFileException: /tmp/pyflink/4cab4606-4d40-423c-9da2-110f4d7858b9/ff3fe4a6-f89a-4fec-a849-5ececc23caee/flink-job-2466975472410535843.py
        at sun.nio.fs.UnixException.translateToIOException(Unknown Source) ~[?:?]
        at sun.nio.fs.UnixException.rethrowAsIOException(Unknown Source) ~[?:?]
        at sun.nio.fs.UnixException.rethrowAsIOException(Unknown Source) ~[?:?]
        at sun.nio.fs.UnixFileSystemProvider.createSymbolicLink(Unknown Source) ~[?:?]
        at java.nio.file.Files.createSymbolicLink(Unknown Source) ~[?:?]
�       at org.apache.flink.client.python.PythonEnvUtils.createSymbolicLink(PythonEnvUtils.java:240) ~[flink-python-1.19.3.jar:1.19.3]
        at org.apache.flink.client.python.PythonEnvUtils.addToPythonPath(PythonEnvUtils.java:294) ~[flink-python-1.19.3.jar:1.19.3]
�       at org.apache.flink.client.python.PythonEnvUtils.preparePythonEnvironment(PythonEnvUtils.java:226) ~[flink-python-1.19.3.jar:1.19.3]
�       at org.apache.flink.client.python.PythonEnvUtils.launchPy4jPythonClient(PythonEnvUtils.java:487) ~[flink-python-1.19.3.jar:1.19.3]
        at org.apache.flink.client.python.PythonDriver.main(PythonDriver.java:92) ~[flink-python-1.19.3.jar:1.19.3]
�       at com.example.PythonJobLauncher.main(PythonJobLauncher.java:33) ~[de9b8c19-aa32-4320-9852-2a6ed7f3abae_flink-python-job-1.0-SNAPSHOT.jar:?]
        at jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[?:?]
        at jdk.internal.reflect.NativeMethodAccessorImpl.invoke(Unknown Source) ~[?:?]
        at jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(Unknown Source) ~[?:?]
        at java.lang.reflect.Method.invoke(Unknown Source) ~[?:?]
        at org.apache.flink.client.program.PackagedProgram.callMainMethod(PackagedProgram.java:355) ~[flink-dist-1.19.3.jar:1.19.3]
�       at org.apache.flink.client.program.PackagedProgram.invokeInteractiveModeForExecution(PackagedProgram.java:222) ~[flink-dist-1.19.3.jar:1.19.3]
        at org.apache.flink.client.ClientUtils.executeProgram(ClientUtils.java:108) ~[flink-dist-1.19.3.jar:1.19.3]
�       at org.apache.flink.client.deployment.application.DetachedApplicationRunner.tryExecuteJobs(DetachedApplicationRunner.java:84) ~[flink-dist-1.19.3.jar:1.19.3]
�       at org.apache.flink.client.deployment.application.DetachedApplicationRunner.run(DetachedApplicationRunner.java:70) ~[flink-dist-1.19.3.jar:1.19.3]
�       at org.apache.flink.runtime.webmonitor.handlers.JarRunHandler.lambda$handleRequest$0(JarRunHandler.java:108) ~[flink-dist-1.19.3.jar:1.19.3]
        at java.util.concurrent.CompletableFuture$AsyncSupply.run(Unknown Source) [?:?]
        at java.lang.Thread.run(Unknown Source) [?:?]
</code></pre>

Well, this is where the _LLM_ saved my ass. I decided to stream the code into a location using the _JAR_ rather than copying it extracting it and moving it, but I didn't know how to do it in Java. This is totally a hack and requires awareness on the stream size and what is actually tolerable for the process (don't you dare to try loading a whole package in it!). For my case, however, this worked great, and I could right away trigger jobs using a _CURL_ call to the _API_:

<pre>
# Send my JAR
curl -s -X POST http://localhost:8081/jars/upload -F "jarfile=@flink-python-job-1.0-SNAPSHOT.jar"


# List my JAR ID
curl http://localhost:8081/jars | jq

# Run my Python Job
curl -s -X POST "http://localhost:8081/jars/<JAR ID>_flink-python-job-1.0-SNAPSHOT.jar/run" -H "Content-Type: application/json"
</pre>

So if you find yourself in the same pickle, here's the class to get this whole mess fixed. Long term fix: Improve my _Java_ knowledge.

<pre><code>
package com.example;

import org.apache.flink.client.python.PythonDriver;
import java.io.InputStream;
import java.io.OutputStream;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.nio.file.StandardCopyOption;

public class PythonJobLauncher {
    public static void main(String[] args) {
        try {
            // Extract Python script from JAR resources
            InputStream pythonScript = PythonJobLauncher.class
                .getResourceAsStream("/main.py");
            
            if (pythonScript == null) {
                throw new RuntimeException("main.py not found in JAR resources");
            }
            
            // Write to a predictable location that won't be moved
            // Use the working directory instead of /tmp
            Path pythonFile = Paths.get("job.py").toAbsolutePath();
            
            System.out.println("Extracting Python script to: " + pythonFile);
            
            // Write the file
            try (OutputStream out = Files.newOutputStream(pythonFile)) {
                byte[] buffer = new byte[8192];
                int bytesRead;
                while ((bytesRead = pythonScript.read(buffer)) != -1) {
                    out.write(buffer, 0, bytesRead);
                }
            }
            
            System.out.println("Python script written successfully");
            System.out.println("File exists: " + Files.exists(pythonFile));
            System.out.println("File size: " + Files.size(pythonFile));
            
            // Build arguments for Python driver
            String[] pythonArgs = new String[] {
                "-py", pythonFile.toString()
            };
            
            System.out.println("Calling PythonDriver with: -py " + pythonFile);
            
            // Launch Python job
            PythonDriver.main(pythonArgs);
            
            System.out.println("PythonDriver completed");
            
        } catch (Throwable e) {
            System.err.println("Failed to launch Python job: " + e.getMessage());
            e.printStackTrace();
            throw new RuntimeException("Python job failed", e);
        }
    }
}
</code></pre>
