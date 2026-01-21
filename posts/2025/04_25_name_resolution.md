### What happens when you open google.com in your browser? (04/25)

This is the first entry of what I will call: Tiny ass cheat sheet (I have a <a href="../../archive/big_ass_cheat_sheet.html" target="_blank">big ass cheat sheet</a> if you're curious), where posts aren't going to be tiny, not reflective of anything going on with my life, yet I am trying to give some closure to some questions where either I bombed or had a _spirit de l'escalier_ moments.


So what happens when you call google.com from either your browser, curl, ping? This is a common systems engineering interview question, where the interviewer will try to delve into your understanding of networking, protocols, name resolution and ultimately try to get off by pointing out how ignorant you are if you miss a thing or two. Yes, those jerks.


So let's go step by step, alright? Your program will call google.com. It will create a socket (`socket()`) to connect to a remote address. There are going to be several steps involved, the first one will focus on name resolution, well, because you are using a name. So `connect()` will first of all require an address. Your _OS_ will likely go to `nsswitch` to seek where the naming info is taken from, either a Berkeley _database_ or a file. Normally, this is a file (`hosts`) so next stop will be reading that file and -in this scenario- finding nothing, it would not stop there, but moving on to where our name resolution resources are configured (`resolv`). Yes, there were already a bunch of system calls already involved in the process of read and lookup, no I will not mention those, I don't even know all of them (`bind()`, `listen()`, `read()`, `fstat()`, etc), so, the next stop is asking the gateway. This will require figuring out what address is the gateway (if not present in a routing table) so you're going to send (broadcast) an _ARP_ call to your router if that one is missing or if the _DHCP_ has not provided you with it (which likely had). So once you call your gateway (I will emcompass the "gateway" part as your router and _ISP_ here for simplicity, IRL the modem/router at home would forward the call to the _ISP_ routers, _POPs_ and so on, and so on... `computer -> router -> isp`) your next call in the path of what `sendto()` would try (at this point the socket creation is still in need of an address, you will try resolution first), here is another call pending to be mentioned `getaddrinfo()` which we need to define where is the request actually going. So let's talk about domain name (_DNS_) now.

Your lookup made it all the way to the gateway and no name to address relation is found yet, you will care about routes later, first you need an IP and that's where _DNS_ will come into play. Chances are, your first _DNS_ will be that one your _ISP_ provided, so if the entry for the address is cached in it, you will have an address. Let's pretend it isn't what it is now? Your call will make the _DNS_ go and ask the root level. There are few servers in this level, all they are going to do is direct the call to the next level in the lookup hierarchy: top level, that `.com` extension thingy. _DNS_ relies in forwarding calls in a recursive manner so the request lands where it is likely to be present in a resolution table, yet most of this work is cached already, that explains why sometimes it takes time (lots of it sometimes), for a change to be consistent with reality... now feels like eventual consistency is not so new anymore right?

So the top level (_TLD_) will search its authoritative server for the domain name (`google.com`) and send the request there. Can I see this in my interface using `dig`?

Kind of, try: `dig +trace -4 google.com` and you see it. If you happen to have your own _Bind9 DNS_ you are going to get a more thorough view. Cherry on top? Add `strace` and see the syscalls by yourself.

So assuming you have an address (in this case, likely an _API_ gateway or a point of presence that will get you content via _CDN_), now what? And wait we were sending _ARP_ broadcasts, was this part of it? Nope, these calls were _UDP_ calls to a known path (yes, that routing happened behind scenes and you were basically talking to only one _DNS_ machine), and unless the response was so large or _DNSSEC_ was involved or simply you used `-tcp`, the call will always use UDP.

Once you have your _IP_, your `connect()` call has a place to go. Now the routing tables in the network including your _ISP_ will direct your packet to the machine with that address. _BGP_ will give devices the optimal route for your request to go, wanna see where that is going? `traceroute` can help.

So what goes on after? It depends, are you using ping? (if _ICMP_ is enabled) you have an echo request. If on the other hand your call uses _HTTP_ and _TLS_, then the call is slightly more complicated, your call will be processed by an _HTTP_ server. If _TLS_ is enabled you will have a _TLS_ handshake. If the server handles more than one _nameserver_, it will need to find the appropriate certificate too (_SNI_), but the dance is quite the same: `ClientHello`, `ServerHello`, you had a negotiation for mechanism, supported extensions, keys exchange and start data transmission. In a nutshell:

`ARP->UDP->TCP->TLS->HTTP`

This is an imperfect way to paint it as a whole, yet it can be more complete than those infographic diagrams you find at LinkedIn if you care to read through. Jerks will still find a way to tell you you're not good enough, but now you can say, at least I tried to get it deeper.

---
<div align="center">
  <a href="../../indexes/2025.md" title="Back to year's index">📖</a>
</div>