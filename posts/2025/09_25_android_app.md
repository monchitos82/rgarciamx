### Developing Android apps using LLMs help (09/25)

This sounds like a very straight forward process after all the buzz <i>AI</i> companies make. In reality, making anything barely functional using an <i>LLM</i> to create code is an iterative process full of bumps and potholes where experience is a big differentiator in building total crap or something useful. Now, I must warn that this effort is not my first rodeo and this was a simple enough project where toying with an <i>LLM</i> actually made sense, but using a tutorial would get me there too.

#### The need

So let's begin with what I wanted here. Calculating carbs out of a label is a big pain in the ass. If you have been into calorie count or have to deal with a condition like diabetes, you know the info not only can be flawed, but you need to be a math wizard to actually come up with the right number. Labels have all the information you need, except that here in Mexico they calculate in carbs per 100 grams. In the USA you get portions, but sometimes you eat less than a portion or more than that because packages rarely come by portion.

Trying to build a simple calculator is a quick job that we can throw in any application. I wanted this, however, for my phone, since I need to have this info with me anywhere. Because of this it made more sense to have this as an app than as a service (I would then need at least a script and an address to access it). I could just put out the phone calculator and call it done, but where's the fun in that?

#### Attempt number one

Until recently, I had never created a phone app. I wanted this for an <i>Android</i> phone (since I could test straight in <i>Waydroid</i> or my <i>Graphene</i> phone). I thought it would not be that complicated, well, I was wrong. As usual, <i>Java</i> is not a terribly complicated language, but its ecosystem is <b>a <i>ffffffreaking</i> nightmare</b>. I could spend hours learning <i>gradle</i>, <i>maven</i> and trying to get this to run or ask an <i>LLM</i>. I chose the latter, but the result didn't really change. I wrote a prompt as detailed as I could, specifying things like limits, dependencies, actions, forms, transitions and other constraints. I got a bunch of files, which upon quick reading, seemed acceptable. So I started adapting the response into my project. Of course the first build was a mess.

I tried to troubleshoot this for a couple hours until I came to a conclusion: I cannot write <i>Android</i> apps, but neither <i>LLMs</i>.

This was no relief, I wanted to do something useful rather than ranting about my incompetence amplified by a model. The model offered an alternative: a native web application. I have always seen these <i>electron</i> apps as an acceptable (sometimes even better) way to do things, they are as portable as it gets, disregard how bloated they seem. I know people have many reasons not to want to use this approach, but my app is simple enough to be stored as a <i>JavaScript</i> file in a <i>CDN</i>.

So after some thought, anger and meditation, I decided to give it a try.

#### Attempt number two

The <i>LLM</i> already had my prompt, I built context around it, I knew what I wanted, I knew what I didn't, I had zero clue on how to build <i>React</i> apps, and I couldn't care less, but this and some clues on <i>JSX</i> were important. Again, the <i>LLM</i> knew more than I did, so using it was very, very helpful.

Once I got some files created, I started the journey on building a <i>NodeJS</i> application. The <i>LLM</i> did a lot of the lifting, but here comes where it really pays to know how to build projects before the <i>Generative AI</i> era: You know how to model a product, you know how to model your data, and you know what it means to have a good enough result. I could let the <i>LLM</i> take the wheel and spit code out of control, the result would be disastrous. How do I know? I let it run a couple iterations and it made a sloppy joe sandwich of the starting code.

So one trick that we must use here is: Pass your code on each prompt and make sure the model sticks to it. Letting a model drift away from a function or method that actually works and has been properly crafted is a recipe for chaos, remember, the model will assume it is always right, even when it is not. You <b>must</b> do the same when dealing with dependencies, especially for technologies like <i>NodeJS</i> where supply chain attacks are not strange; don't lose sight on your advantage: Time. The model knows what it has been fed during training, but nothing after that point.

People <i>vibe-coding</i> tend to expect to get a model to prioritize correctness over ability to deliver. This creates garbage not to say it introduces bugs to otherwise functional code. Remember, <i>LLMs</i> are the equivalent of a know-it-all person, including their arrogance. You really need to expect the result to be wrong, because the <i>LLM</i> will assume it is always right.

So where did I end up after this attempt? Well, my <i>NodeJS</i> application definitely worked, but I had to use a couple tricks to get it to work on my phone. Again, the <i>LLM</i> was my guide and I asked for instructions on the tooling to be used after.

#### Wrapping up things

When I started with this whole project I had installed <i>Android Studio</i>. Using the advice from the model, I installed <i>capacitor</i>. This combo basically wraps up the <i>NodeJS</i> application and sets up all the dependencies to get this built into an app. The build and packaging is done by the <i>IDE</i> and the result is a debug <i>APK</i> file I could use both in my emulator or my phone. This sounds complicated, but tooling is quite sophisticated because all I needed to do was run:

<pre>
npm run build
npx cap init
npx cap add android
npx cap sync
</pre>

And then have <i>Android Studio</i> take care of the rest.

#### Wrapping up (my post)

In the end, I could not write a native <i>Android</i> app using an <i>LLM</i>. But it is fine, I think the complexity around <i>Java</i> or <i>Kotlin</i> is just obnoxious and creates a problem beyond the language itself. That's something I praise in <i>Go</i> regardless of what critics say, tooling feels adequate for the task.

As I said before, I think <i>LLMs</i> are great intellect amplifiers, not a replacement. Knowing how to ask things and building a context around a task is no small feat. Just as forcing the analysis to stick to existing code is. Those expecting a great outcome from a prompt no matter how well thought it is, are in for a nasty surprise; this requires context awareness on the topic, the requestor needs to build a world of concepts and constraints where the knowledge of a model will make sense out of it, this by itself is a skill that needs practice and reinforcement. I think the progress in these models is tangible and while hyped, <b><i>Generative AI</i> does have the potential</b> to improve our productivity, but we cannot slack on it, we must use it as it is: a tool, just as we should use books.

---
<div align="center">
  <a href="../../indexes/2025.md" title="Back to year's index">📖</a>
</div>