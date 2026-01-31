### Postmortems: How and why? (05/25)

One of the best parts of being in an _SRE_ team **definitely** is: running a _postmortem_. It is also one of the hardest though. I was terribly sloppy when I started taking care of the task, and it was not until I was mentored by great engineers while running _postmortems_ that I understood both the value and how to actually do them right (kind of learned by doing and got scraped in the process, to anyone who I disappointed during my learning process, I am still sorry about the bad experience). There is a great shortcut to get there though, and it is actually documented: Read engineering blogs, play the [wheel of misfortune](https://sre.google/sre-book/accelerating-sre-on-call).

A _postmortem_ goes beyond trying to find the ugly truth and following a template, although having a good template and a good _postmortem_ document is a non-trivial task. A memorable _postmortem_ is that one where you leave the room with conviction of: a) now you know way more than what you did when you wrote the document draft, b) your respect for your collaborators and their work has a new high and c) you're convinced you are going to make things better from that point. Truth is, this is not always achieved and especially point _c_ is a pain in the ass, because pressure on releasing new things is always pushing flaws downward in everyone's queue.

So why are _postmortems_ hard anyway? Where is the process turning into something difficult?

_Postmortems_ are a blameless process (or they should be). Keeping it that way is a titanic effort. Nobody likes to hear their work is dogshit and their sloppiness causes an outage, regardless, those things happen and the process of figuring out what happened in detail, when done right, brings out all the dirt from under the rug and sometimes, it gets ugly.

_Postmortems_ require a fair amount of effort, both because they require focus, and they require to do an investigation fast. It serves no good to schedule a _postmortem_ meeting a month after an incident happened, by then the world has moved on. So, whoever oversees gathering people needs to do both: a thorough investigation and organize a meeting with a similar sense of urgency to what it took to solve an incident that triggered a _postmortem_.

_Postmortems_ derail a ton of work. Incidents happen but making sure they don't come back requires revisiting a lot of work and disrupting the release process of features, it demands commitment from engineering and product parties to prioritize site up above everything else. As with any important decision, it is not coming for free.

I will steer a little bit here and talk about what is a _postmortem_ anyway, how it is expected to go and what made a _postmortem_ memorable in my experience. My experience is limited though, so this should be brief.

When systems and infrastructure fail, failure brings a learning opportunity. Organizations that follow an incident management practice will identify the steps to follow from the moment an incident is detected until it is resolved, however, the root cause of an incident is not always crystal clear and while in the old _ITIL_ world closure would come after investigation, a thorough root cause analysis and long term remediation may remain unclear; probably that's why _ITIL_ themselves formally introduced the process.

The blameless _postmortem_ process as a practice was popularized by _Google_ when they introduced _SRE_, and it is a powerful tool to understand what has gone wrong rather than figuring out "who" dropped the ball. As I mentioned before it is easier said than done, egos and power dynamics (a.k.a. red tape / office politics) often play against. I think that one of the best practices I witnessed when trying to make this blameless and a learning endeavor is **keeping your organization values present**, I think the reason why, is because you already have a common goal which is honor those values and of course you have that problem that's stealing away your sleep, and hell, you want it gone!

But how do things flow exactly?

<pre>

your system shits the bed
  |
  |____ you get alerted and you start an incident resolution process
          |
          |____ you find the source of the problem, you fix the problem
                  |
                  |____ you want to make sure this is not reoccurring, you start investigating what happened indeed
                          |
                          |____ you document your findings and establish a timeline
                                  |
                                  |____ you bring together all stakeholders of the system | business
                                          |
                                          |____ you unearth the details of the incident and make sure you answer:
                                                   - what went wrong
                                                   - what went right
                                                   - where luck was on your side
                                                   - where it wasn't
                                                   - what could have been done to prevent it
                                                  |
                                                  |____ you identify all the broken parts of your system and process, agree to a fix, set a timeline and a priority
                                                          |
                                                          |____ you follow up the fixing process
                                                                  |
                                                                  |____ you recover your sleep... or maybe not

</pre>

_Postmortem_ as a process, peaks when a meeting occurs. Memorable meetings are not those where people point fingers, but those where people recognize things are broken and agree to fix them properly. Basically, a memorable _postmortem_ is one where you agree to **make things right**. Sadly, not all leaders and top engineers want to go this way, tech debt is often what comes in exchange for moving fast, other people with dubious morals will do what they can to go unscathed or simply suggest sweeping things under the rug. But even in the ugliness of these situations, great engineers arise, people calling out this as a deviation from your core values or simply owning the problem. When this happens, you realize not everything is rotten. Is this it? Just the old good vs evil? No advice here? Well, I can't spill the secret sauce, but basically try to run a meeting like this:

- Come prepared. Your document is clear, the details are there
- Have people read your doc
- Walk through your timeline, root cause analysis and start asking for gaps
- Have people question what they read, fill in the blanks, improve the picture
- Have the experts chime in, this is where their wisdom helps
- Ask the questions from the diagram:
    - what went wrong
    - what went right
    - where luck was on your side
    - where it wasn't
    - what could have been done to prevent it
- Set action items
- Find owners, if nobody wants to **own the problem**, ask more questions, call for accountability
- Set a timeline, document again
- Follow up your document, empty promises bring problems back and you as the person in charge of preventing this from happening again, should oblige
- If the problem rises again, your investigation went wrong, it will happen, sometimes it is just symptomatic, sometimes it really is a bad fix, learn from it!

Make sure that no matter what, there is nobody pointing fingers or just ranting, you want a problem fixed, this is your duty.

But wait, who runs the meeting after all?

In my brief experience, incident management leads (the person who coordinated the fire fighting) is the right person, however, there's nothing wrong with passing on the lead to a more experience engineer or a manager during the meeting, after all this is not an exercise to take credit, you want things fixed above everything else, your site stability, your system, your alerting, your rest and your sanity are at stake here.

What happens after this?

Well, if you write a good _postmortem_ not only you helped solve a problem, you have good material for your engineering blog, a good tale for your nerd friends or a justification for your eye twitching.

---
<div align="center">
  <a href="../../indexes/2025.md" title="Back to year's index">📖</a>
</div>