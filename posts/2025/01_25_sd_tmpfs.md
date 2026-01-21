### Saving my SDs using tmpfs (01/25)

I won't delve here into `tmpfs` because I don't know where it started beyond what <a href="https://en.wikipedia.org/wiki/Tmpfs" target="_blank">_wikipedia_</a> has. However, if you ever worked with a raspberry, you know _SD_ card lifetime is limited and I come to find that `tmpfs` has been a time and money saver since I minimized the writing to my storage by using it. Yet as you may expect, this is ephemeral (since it is saving data into a block in the _RAM_) so yes, this has two drawbacks actually, you lose memory for the sake of having a temporary place to write. Is it worth it? IMO yes. Any permanent (and important) storage I have deferred to external drives, but external drives are slow. Things like logs (which in these boards I don't care about), go to the `tmpfs` for sure.

So how easy is it to get this going?

Actually it is pretty simple. To the extent to say it comes out of the box, so pretty much all I needed is:

1. Add the entry to `/etc/fstab`
2. Add the dirs/files (this is a minor ache) that need services like _nginx_ back to the _filesystem_ in order not to have startup failure upon restart; file: `/etc/tmpfiles.d/var.conf`

The entries of each file are quite simple:

<pre>
# fstab
# in this case, we specify we mount /var/log as tmpfs with 400MB size
tmpfs  /var/log  tmpfs  defaults,noatime,nosuid,nodev,mode=0755,size=400M  0  0

# var.conf <- this would go for any filesystem that actually is mounted
# in this case, we specify we want a dir for nginx with owner www-data in the path
d /var/log/nginx 0755 www-data www-data -
</pre>

And just like that, I have kept my _SD_ cards alive for close to two years. Probably with a v.5 this is not needed if you just use a _NVMe_, but that's not a cheap option when dealing with more than one board.

---
<div align="center">
  <a href="../../indexes/2025.md" title="Back to year's index">📖</a>
</div>