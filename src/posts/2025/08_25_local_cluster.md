### Local cluster access (08/25)

I wanted to add a post with a lot of details and little rant for some time about how complicated is to do something that should just work, but well, this is our reality, regardless how much I like it or not. So if you ever wonder how to access something like _argo_ in a _kubernetes_ cluster managed by _podman_ without any `port-forward` voodoo and you do this for experimentation purposes, well, I think I found a formula.

I'm not gonna lie, I spent countless hours trying to find the right approach. I even asked _AIs_ for help and, of course, still failed. In the end, after tinkering, the solution is simpler than what you might think.

Adding a _kubernetes_ cluster in your local machine, to me had 2 common flavors: _minikube_ and _k3d_ (this is _k3s_ optimized for _docker_). I knew nothing about _kind_ when I started this, I know _minikube_ is a pain in the ass (too raw), and I read _k3d's_ warning: _podman_ is not supported. So I had to opt for an alternative and well, _podman_ suggests using _kind_.

The first challenge was easy to overcome, when you try to start a cluster (either using _k3d_ or _kind_), _podman_ will croak because it wants to use a rootless access, well, _podman_ replaces _docker_ and if you were smart enough to install `podman-docker` extension (mimicking _docker_ commands), you just need to give it access to the `podman.sock` socket. To do this, just mind adjusting permissions or adding whatever runs _podman_ to the same group that owns `/run/podman/podman.sock`. First I tried adjusting this using env variables, but the results was poor, the easiest way to get _kind_ to start a cluster is doing this, just be wary on the permission settings.

As I moved on, I found pretty simple to start and add services to my cluster by using _helm_, however, exposing ports is _nightmarish_, the reason for that is that _kind_ runs its _control-plane_ in a container. You cannot just change a container without recreating it, so you need to give some thought on what you will expose, that's not something coming as default and will feel uncomfortable from the _GUI_. This however, is key to forwarding traffic, so if you mean to let your services be accessed without using port-forwarding, you need to at least expose a port through your config (`config.yaml`):

<pre>
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  extraPortMappings:
  - containerPort: 80
  ...
  - containerPort: 30000
    hostPort: 8088
    listenAddress: "0.0.0.0"
</pre>

Adding this to the list of your port-mapping will spare you a tons of headaches. Adding it from the _GUI_ was not possible for me though (I tried using the config field and it did nothing!), so you will need to delegate this to `systemd`. Why `systemd`, what did I miss? Well, rootless is a premise of kubernetes in _podman_, something that I think, they strived for when _podman_ was offered as the alternative to _docker_ (_docker_ has evolved since, but well...) so in order to start a service, you will need to do it at user level scope:

`/usr/bin/systemd-run --scope --user -p "Delegate=yes" kind create cluster --name my-kindiest-cluster --config config.yaml`

Once you have this mapping, basically you defined that whatever that is bound to port `30000` will be accessed through `8088` in the host machine. This is necessary for your service which you will later patch.

From here on, installing a service, is just a matter of running `helm` and installing things:

<pre>
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
helm install argocd argo/argo-cd
</pre>

In a perfect world you would be done here. Maybe, you would want to add an ingress to forward traffic to a path so your node can serve more services (this **never** worked for me), but one thing you will learn, is that there are no happy paths in _kubernetes_. So having no ingress and being unable to access your service, what then?

If you ask _Gemini_ or _ChatGPT_ the solution is using the `port-forward` subcommand, but do you really want to hijack a terminal with a command output? Well, forwarding should be permanent, but it isn't, however, you can patch the mapping, in this case, `8088` is already set to get traffic from port `30000`; this port is in the range of `NodePort` and we want to map our service to it, this is needed, since we will direct our application to what is accessible according to the _kubernetes API_:

`kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort", "ports": [{"port": 443, "targetPort": 8080, "protocol": "TCP", "name": "https", "nodePort": 30000}]}}'`

I gotta be honest here, I despise that services and orchestrators alike think that ports `80`, `443` and `8080` are just there available. This is the part of the work where development seems just idiotic, but well, in this case, patching will tell my service that whatever it's configured to use port `8080` (which I don't want it to use, but it is its default) will get traffic from port `30000`, this mapping will allow me to use the other mapping I defined (container level mapping), thus, accessing my browser at `https://127.0.0.1:8088` will get me into my service landing page.

There's a lot of juggling in this and well, it could be worse, we could add more services and then find that we need a longer list of port mappings in the control-plane. We could add proxying and an ingress to give it some sense of order, but that's just naive, because, as I said, there is no happy path.

Asking this to _LLMs_ feels like asking this to reddit's _kubernetes_ evangelists: _"this is better because excuse, excuse, excuse..."_. I did some further reading, and I learned that in the "real" world, where this is run in multiple nodes, my setup would have less friction, or at least less mapping here and there. Would it? I stated it at the beginning: I don't mean to rant, **my service is accessible!** However, I think all these stunts could just be replaced by running a bunch of services and an _nginx_ instance without the _yaml_ and layers non-sense, or even better, by letting what has been proved to work for many decades take over (yes, I mean _DHCP_, _DNS_, _NAT_, which, by the way, have had their working equivalents in the public cloud for close to 2 decades already).

However, these systems will come with their own inconveniences, right? Yes! We could add proxy clusters and health-checks, define a load balancing strategy and autoscaling mechanisms and well, end up with the same kind of problem; I think _kubernetes_ is fixing an inconvenience and creating another one, but I understand its purpose, a noble one, that could however, be fixed in a well controlled manner, e.g. from the application layer itself. I admit, I would be blindsided by evaluating _kubernetes_ just by a single experiment on a single host for a single instance of a single application, so just be mindful on your needs before you embark into this journey. Call it and place it wherever you want, layers and complexity are a trade-off that, at least in my opinion, should not be adopted before the need arises. 

**_tl;dr_ - Don't mind having a container orchestrator infrastructure when you don't need more than one instance, focus instead on configuring things properly.**

I don't see the value in convoluting systems with the lame excuse of scale; most of our foundation services can handle a fair volume of traffic just fine and doing things for a scale that never comes is just foolish, but I think that's a conversation for another day.