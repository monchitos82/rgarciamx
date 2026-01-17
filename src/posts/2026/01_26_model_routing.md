### Model routing 101... for me (01/26)


Being the first post of 2026, I wanted to make this post something more complete without missing the opportunity to offer some technical advice or some criticism. I think I wanted to take the chance to work with LLMs again; it seems that the trend is catching up and I like to think I already learned a thing or two on the subject during the past few years.


I will start by presenting the scenario where I currently am: As I search for a job here in Mexico, opportunities differ to what you would find in the Bay Area. In comparison, here the scope is narrowed to certain tools and activities... and problems! That's quite why I gave up on doing Site Reliability here, but aiming for a  _Python_ position is not very different in reality, either you do web development in _Django_ (or _FastAPI_ nowadays), or you do **whatever** is perceived as AI work. On the latter, I think one of the "experience" requirements that has drawn my attention more than once is that of "working with _LLM_ implementations". The description is vague at best and many times this only translates to: Writing _AWS Lambda_ functions and routing requests to _AWS Bedrock_, which somehow is an interesting problem by itself, but it means no LLM implementation experience whatsoever. It really makes me wonder if we can consider "an expert" a person capable of reading the _SciKitLearn_ documentation, or we are just masking a bit of _Flask_ with a bit of basic _ETL_ and well, I have no way to validate this, but I am very suspicious of something: Most "experts" have no damn idea about what they do and opt to defer this to whatever _Claude_ spits out (It still has some merit being able to _RTFM_, don't get me wrong). I would do the same in my ignorance, except that my curiosity pushes me to ask questions. So I asked them.


Trying to solve this non-trivial problem, I decided to ask the only "experts" I have around, and I asked the _LLMs_. What was the problem again? Oh yeah:


You have a prompt and a _suite_ of models you can use. Each model has a different specialty and a cost. You want to optimize the decision process to route a prompt to the most fit model in the most cost efficient way.


Once upon a time, I was tasked with translating a bunch of decision rules for some machine learning (supervised) models. I know this task by itself is cumbersome and non-trivial, but is this what the position is all about? I think this is just part of the problem. When bouncing ideas with the **models** (From this point onwards, I'll use bold when I refer to large language models), I concluded that if we could put this into a funnel, the early classification based on some rough heuristic is maybe the top layer. When done right, this can mean saving a lot of time and processing considering this would enable using less sophisticated machine learning models to do the classification work, but I am no expert, it just makes sense to me.


Based on my own observation and what I got as a first response, many AI companies rely on some hard rules including the ability of the user to pick a model (or the assumption that most will never touch that dropdown menu), whatever it is, classification _could_ start from the user, so if the input is somehow influenced by the user interface, maybe we have that top layer figured out.


This could be implemented by using base modeling, considering we have the task, source, prompt and some specifics that can be easily mapped using _Pydantic_ (for reasons, in all clarity you might as well just use _Dataclass_), but I think this early classification is just good for a tracing purpose later down the funnel. At some point the **models** suggested using another _LLM_ to do the classification based on patterns, while this could work at a small scale, I dare to believe that this would have problems scaling to higher volumes of either words or prompts. What can be done then? If the filtering is good enough (this concept by itself requires a whole different section, we will get there) we could implement a second layer of classification based on different approaches: Regressions or decision trees. At this point, just knowing _Python_ is still good enough, but we need to start figuring out the differences between these machine learning techniques for classification. _SciKitLearn_ documentation is fair, but it won't have an _ELI5_ section, so let me try to help:


Regressions could perform better (in time) at the cost of accuracy when the classification training data and features are poor at best. The decision trees will cost both in time and resources (`Random Forest -> XGBoost -> LightGBM` in that order), but offer a greater degree of confidence when properly configured. Here knowing _Python_ is not very useful anymore, understanding when we are no longer tuning and start over-fitting or simply adding more complexity without gains in the classification requires a good grasp on how these models work and more importantly, on both the feedback loop and the training data.


This, by the way, is just a second/third layer down the funnel, and it really is just part of the design decision that we might call out later and just opt to rebuild. Here is where feedback and experimentation come into play, which I would say, becomes a blurry second layer. There is no confidence in the accuracy of a machine learning model implementation if the classification at the first layer is not good enough or the training data is insufficient. With this in mind, it is a good idea to implement _A/B testing_ to validate the effectiveness of the scoring. This idea, however, is imperfect if the volume of requests is limited or inconsistent (we can opt to replay requests, but this is not a decision we can make free of charge).


The **models** correctly suggested to do some feature extraction at layer one, where we can estimate tokenization, this task however is not simple because there are considerations to be made, from the breakdown of sentences to more complex conditions, like understanding that in some languages a single character is a word and could cost more if we just try to break down the sentence. Again, knowing _Python_ just won't cut it, e.g. "你好" would mean 4 tokens compared to the equivalent 1 token word: "Hello".


At this point I start noticing a pattern, each layer is going to be influenced by the feedback loop. So here's where we can start to delve into the cost allocation. I believe that budgeting the experiments and tracking the consumption of the budget vs. the effectiveness of the funnel layers is not a trivial task or an easy thing to do, basically you need to instrument and trace the decisions and the cost-effectiveness of every decision. This also requires a way to classify the feedback, e.g. if a prompt is being rephrased twice because the first response is not of good quality, will we be able to mark it as a duplicate and more important, to route it effectively?


At this point I believe the _problem_ has no clear solution. Even if we use cross-validation (which we must) at the classification level, we still have no _defined_ way to do cross-validation at _LLM_ level. Well, the **models** agree, there are companies creating their own implementations and algorithms, trying to solve this problem, and while _Claude_ was exhaustively creating code snippets of what the implementation would look like, I believe this is not a programming language problem anymore (if it ever was), and it is more of an architecture one; definitely _Python_ is loaded with _APIs_ and packages, but the tooling definitely is not the _hard to get right_ part of the problem.


I bet you are curious about the layers of the funnel, so let me share a diagram:


<pre>

                                                                                     [------]---- Feedback analysis is done at every layer, this improves the classification
\==========================================================================================/      through training data quality improvement and effectiveness measurement.
 \                                                                                  /     /       This can help improve implementation of signal the need for a different 
  \                                     User request                               /     /        approach.
   \                                                                              /     /         All data produced here is meant to be used for retraining down the funnel.
    \----------------------------------------------------------------------------/     /
     \                               Input validation                           /     /
      \                                                                        /     /
       \                    Pre-processing and early classification           /     /
        \                                                                    /     /
         \                      Caching and metric creation                 /     /
          \----------------------------------------------------------------/     /
           \                         Feature extraction                   /     /
            \                                                            /     /
             \                       Experiment routing                 /     /
              \--------------------------------------------------------/     /
               \                                                      /     /
                \               Classification model routing         /     /
                 \                                                  /     /
                  \------------------------------------------------/     /
                   \                Infrastructure routing        /     /
                    \                                            /     /
                     \                   Execution              /     /
                      \----------------------------------------/     /
                       \                                      /     /
                        \               Evaluation           /     /
                         \                                  /     /
                         |=================================/     |
                         |                                       |
                         |              Retraining               |
                         |                                       |
                         |=======================================|

</pre>


So this is kind of an imperfect way to represent the funnel idea and of course it oversimplifies the whole set of concepts behind, the best part of it: I am not certain about this to be accurate, but hey, nobody does! At least I'm not pretending that I know.

I am intentionally skipping on more advanced concepts (that I don't master either) like neural networks or the use of tensors and torch libraries to encode and train models because, as I said, I am no expert, and I believe that if we can't grasp on the basics, trying to blindly navigate through, what is indeed, a black box will not be of greater help. If we are stubborn and try to force the skills regardless, well, I just hope the profile being recruited no longer is just _Python_ programming, but data science, because as I said, this is not a language kind of problem (if it ever was).
