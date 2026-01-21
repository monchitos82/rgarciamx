### Spirals (06/25)

I wish I had something interesting to say here, especially something unrelated to _LLMs_. Unfortunately, I am doing some prep work. One would think that _LLMs_ would be very helpful with preparations by giving us problems, guidance and feedback. I thought the same. The results are catastrophic.

I get good explanations and good problems from my conversations with _ChatGPT_ (and _Claude_ too), but when I try to get insight into things and even solutions, I am the kind of person who tries to take an extra step, that extra step is what any congruent person would do: validate for correctness. Well, the results are demoralizing.

I wish I could write down all my findings and prompts, but I would resume everything with one image:

<pre>
0 0 0 0 0 0 0
0 X X X X X X
0 X 0 0 0 0 X
0 X 0 X X 0 X
0 X 0 0 X 0 X
0 X X X X 0 X
0 0 0 0 0 0 X
</pre>

This is a "simple" problem. You get a matrix of size N (`NxN`). Then your program:

  - Finds the center of the matrix
  - Fills the matrix with a spiral that emerges from the center and replaces `0` by `X`
  - Solves the problem using an O(n²) time complexity
  - Bonus points if the spiral direction can be switched

Well, I got these beautiful responses:

<pre>
# Claude 4 and GPT 4o
X X X X X X X
X 0 0 0 0 0 X
X 0 X X X 0 X
X 0 X X X 0 X
X 0 X X X 0 X
X 0 0 0 0 0 X
X X X X X X X

# Claude 4
X X X X X X X
X X X X X X X
X X X X X X X
X X X X X X X
X X X X X X X
X X X X X X X
X X X X X X X

# GPT 4o
X X X X X X X
X 0 0 0 0 0 X
X 0 X X X 0 X
X 0 X 0 X 0 X
X 0 X X X 0 X
X 0 0 0 0 0 X
X X X X X X X

# Claude 4
0 0 0 0 0 0 0
X X X X X X X
X X X X X X X
X X X X X X X
X X X X X X X
X X X X X X X
X X X X X X X
</pre>

In all outputs the models were confident they solved the problem correctly. But I am not here to throw shit at the _LLMs_. I want to document how these models become actually useful... Well, the results greatly changed when I provided the solution. Then my own inefficiencies were corrected.

I started by presenting a problem and the expected results. Then I tried to tighten it a little bit by giving constraints, expected input and expected output. Then I provided a hint of what the code could be (provided a non-working solution). Once the model started working on the code I started asking about inefficiencies. Here the workflow broke because as I have mentioned before, models tend to stick to the context so if you provide a non-working example, it will try to improve that example without understanding it. So I provided a working example with a plethora of inefficiencies. At this point the models’ focus shifted from trying to fix my code to trying to make it less wasteful; it removed expensive lookups, it added guardrails (not needed, but tried), it reduced the code complexity. I presented both models with challenges on their decisions and both provided insight. They were not "smart" enough to steer away from the _"You are totally right!"_ answer, but they were actually trying to solve what they could.

_LLMs_ can be helpful and should be considered helpful. They cannot show you the way, that's all on you. But they can help you look back and figure out where you could have planned better. So no vibe coding, no intellect replacement, no hallucinated solutions, simply what they do with any other document: Process, simplify, explain.

<sub>_(2025-08-12) Update: I tried this again in GPT-5, Opus, LeChat, R1, Qwen Code, Perplexity, Gemini and CoPilot, all, **no exceptions**, kept failing miserably._</sub>

<a href="../snippets/spirals.md" target="_blank" title="Read snippet">
snippet with my solution
</a>

---
<div align="center">
  <a href="../../indexes/2025.md" title="Back to year's index">📖</a>
</div>