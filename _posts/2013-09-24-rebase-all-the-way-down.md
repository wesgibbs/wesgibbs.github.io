---
layout: post
title: Rebases all the way down
---

You have a long-running topic branch off of master. Every day or two you rebase the topic branch on top of master to keep it up-to-date.<sup><a href='#foot1'>1</a></sup> You have another branch off of topic in which you are doing incremental work that periodically gets committed to the topic branch. When you rebase topic on master, that incremental branch gets "orphaned". You want to rebase it cleanly back on top of the topic branch.

Now with pictures. Here's what you start with.

{% highlight ruby %}
  A---B---C  master
       \
        D---E  topic origin/topic
             \
              F  incremental
{% endhighlight %}

Your `topic` and `incremental` branches are missing commit `C`. Use `rebase --onto` to correctly rewrite the history<sup><a href='#foot2'>2</a></sup> for `topic` and `incremental`.<sup><a href='#foot3'>3</a></sup>

<div class="highlight"><pre><code class="text">
<b>(topic)$ git rebase master</b>

  A---B---C  master
      |    \
      |     D'--E'  topic
       \      
        D---E  origin/topic
             \
              F  incremental

<b>(incremental)$ git rebase --onto topic origin/topic</b>

  A---B---C  master
           \
            D'--E'  topic
                 \ 
                  F  incremental

</code></pre></div>

You may be thinking that rebasing `topic` on `master` then rebasing `incremental` on `topic` would accomplish the same goal, like this:

<div class="highlight"><pre><code class="text">
<b>(topic)$ git rebase master</b>

  A---B---C  master
      |    \
      |     D'--E'  topic
       \      
        D---E  origin/topic
             \
              F  incremental

<b>(incremental)$ git rebase topic</b>

  [UH-OH]

</code></pre></div>

UH-OH means that you're going to rewind to `E'`, then try replaying `D`, `E`, and `F` on top of `E'`. That's probably not what you want, and in all but the most trivial cases you're going to wind up with messy conflicts. What you want is to permanently drop `D` and `E`. `D'` and `E'` have replaced them. Using `rebase --onto` accomplishes that.

<p>&nbsp;</p>

<div class='footnotes'>
  <div class='footnote'>
    <sup><a id='foot1'>1.</a></sup>
    And you're using <code>rerere</code> to avoid reliving the same conflicts each time.
  </div>
  <div class='footnote'>
    <sup><a id='foot2'>2.</a></sup>
    Assume all the usual caveats about rewriting git history.
  </div>
  <div class='footnote'>
    <sup><a id='foot3'>3.</a></sup>
    In these examples, note the branch from which these commands are run in the prompt. There are alternate invocations of these commands that take that branch as an argument and so don't care what branch is current when the command is run. I prefer the context provided by switching branches.
  </div>
</div>
