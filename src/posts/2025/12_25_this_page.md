### How this page was built (12/25)

I started this page as a blog and I have changed it so frequently that it is hard to describe what it is, and its purpose. I added a <a href="../../changelog.txt" target="blank_">changelog</a> to describe what changed within, but I wanted to take the chance to write about this site itself.

The reason for this, is mostly to document a simple build pipeline with no special tools or artifacts; I mean, I integrated some _CI/CD_ for the sake of automation, but in reality, this whole thing really just needs a script and _Neocities_ to go live on every change.

I didn't pick any shinny tool or platform because I really enjoy the crafting process. Sometimes I write these things in plain _HTML_, although it is not the most efficient way (that's why _Markdown_ is the default format).

I can say the same about other parts of this page. It used to leverage on _Bootstrap_ and some _JavaScript_, even some stats using _G4A_ when I had it hosted at _Github_ pages, but all those are gone, because, I just don't care about adding dead weight to this page. It still uses some _CSS_ and templates, frankly, this site is as vanilla as it gets with modern browsers.

Behind the scenes it is not much different what happens for this page. I might take ideas from here and there, especially for the robots and style files, but that's just me being lazy about trying weird things and breaking the site. Normally, every entry starts with a new _Markdown_ file getting created. From there all I need to to do is the real hard part, write the post.

Once the post is ready, the publishing process just follows a few simple steps on my end: It is all about storing this in a repository and let it go.

When I say "let it go", I mean, the update happens, then the changes go through a very simple build process in a _Jenkins_ container:

<pre>
Repository update
  |
  ∟ Build HTMLs
       |
       ∟ Rebuild Indices
            |
            ∟ Async Validations (broken anchors and foul language)
              RSS build
                    |
                    ∟ Publish site using its API
                        |
                        ∟ Cleanup
</pre>

That's it. Simple as it is, I enjoy doing this and connecting pieces and ideas. I used to build other sections like a _snippets_ one, but I decided not to dedicate a menu for it, or any other than the actual posts. I am experimenting using _Ollama_ calls to add descriptions to the indices, but I think the result is not compelling enough to actually devote time for the models to produce an abstract, it just feels like an overhead. I think this site has evolved both in what I want to show and how I want it to work. Simplicity has been key for me to keep this project alive, and I think this will continue being my companion.
