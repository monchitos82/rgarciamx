### Local containers shenanigans (08/25)

During this week I got curious about other distro options. While my _Ubuntu_ installation always felt trustworthy, it was still a bit clumsy here and there, so I did what anybody like me, with too much time in their hands would do: experiment.

I started with some _VMs_, but you don't get to experience things properly in a _VM_, especially figuring out if things _"just run"_ or you need to put in the hours. So I got some live _ISOs_ and started my mess.

I started with _Manjaro_. I think it was super neat, the _UI_, the feel was snappy, `pacman` (or `pamac` in this context) is very, very complete. I think however it still required a lot of pull here, snatch there (especially messing with the _BIOS_, I understand their point, but manufacturers don't) so I didn't delve further, I'll go back to it later though. I tried _PopOS_ next, and well, as any _Debian_ based, it was easy going, but the same pains I had with _Ubuntu_ were there. Finally I tried **_Fedora_**, by the time I installed it, I was already tired of this. So I left _Fedora_ as my distro.

I think this was no bad decision though. My last _"living only with a Linux machine"_ experience, about 8 years ago had _CentOS_ and it wasn't perfect, but was good enough to get me started into doing more dev work, more often. _Fedora_ felt similar, except it was way nicer with my hardware, so I didn't feel like going back to _Ubuntu_ and setting up a myriad of accounts and settings... again.

But this hasn't been all _"peachy"_, quite the opposite, I forgot about how painful it is to work with _SELinux_. And _Fedora_ was prompt on bringing the experience back. Not the _OS_ fault at all, my bad hygiene creating services out of any process worked fine in _Debian_, but here was different, it demanded some discipline from me and well, it took me a few hours and some trial and error and then it hit me: I should build these things as containers, especially when they aren't going to be used thoroughly or when you will go over this _setup-configure-run_ pain again and again. Since I spent some time in _podman_ not long ago, well, I had the commands still fresh in my mind.

I don't remember all the syntax of everything, but I found that _ollama_ and _openweb-ui_ are actually available in containers. Well, time to try this new goal, which by the way, broke my laptop once the Nvidia drivers were installed, but we know how things are with Nvidia and _Linux_. Having fixed it (using those neat pages from _podman_ itself), I got these running in no time. And it hit me again after installing **_Qwen_**: This is as good as we need our generative _AI_ to be. It took a few seconds to give me an answer, but this and some reading through _API's_ got me running 30 different services, all at the same time, with no memory bottlenecks. Yes, for my use case, which is simple: my own development, my own monitoring, my own _CI_, my own _DBs_, my own learning, it is more than enough, but that's the thing, my use case is already complex, my hardware is quite good for 2020, but 5 years later is good enough as well to run an _LLM_ and get me spitting code and configurations as fast as I can. I had broken some configurations, had to wrestle with _airflow_ (please somebody tell me what that darn _**Microsoft** directory_ is?) and figure out ways to edit volumes before mounting them as read only, but in the end, all my infra, was replicated beautifully.

<div align="center">
  <p><sub>阿里巴巴的Qwen很棒!</sub></p>
</div><br/>

I know this post is quite shallow since I don't cover many technicalities here and instead dropped names here and there, but honestly all the work is already in the docs from most services, I cannot remember many lessons other than perhaps _flink_ configurations are better set using `-D` (dynamic arguments), or that _alpine_ is super handy when you screw up a config and plan on editing it within the volume (useful for _airflow_ and _trino_ to name a few), or that building _Go_ images is a pain if the `workdir` is not accessed correctly, but the nice thing is that this is a once only kind of pain when done right.

A good one liner for posterity:
<pre>
  for CONTAINER in $(podman ps --all --format "{{.Names}}"); do podman inspect ${CONTAINER} | jq '.[0].Config.CreateCommand | join (" ")' | sed -e 's/^"//g' | sed -e 's/$"//g'; done > podman_container_commands
</pre>

A good set of lessons for me:

* Don't use home directories to run services
* Get comfortable using volumes and `:z` (a lot)
* Get familiar with `host.containers.internal` and mind those `localhost` configurations
* Read the docs, especially those from _podman_ and `Dockerfile` (last time I read them was 2015!)
* There's always a way around `docker-compose`
* Always add volumes for data and prefer variables over configuration files (mind the risk that poses, well, secrets exist, right?)
* Don't mess with _SELinux_ configurations when you're missing packages (this one broke my boot, my containers, _plymouth_ and _systemd_)
* Backup `/etc` from time to time, sometimes it is a big pain to write configurations all over again