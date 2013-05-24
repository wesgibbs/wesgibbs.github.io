---
layout: post
title: Heroku BUNDLE_WITHOUT config variable wants colons
disqus-url: http://wesgibbs.me/post/1506018375/heroku-bundle-without-config-variable-wants-colons
---

It appears that Heroku recently changed the syntax for the BUNDLE_WITHOUT config variable.

The cached version of this [page] (http://docs.heroku.com/bundler) still shows the old syntax where the environment names are separated by spaces:

{% highlight ruby %}
heroku config:add BUNDLE_WITHOUT="development test"
{% endhighlight %}

The current version shows them separated with a colon:

{% highlight ruby %}
heroku config:add BUNDLE_WITHOUT="development:test"
{% endhighlight %}

I remembered this error when I initially deployed my app to Heroku. linecache19 was being installed as a dependency of ruby-debug19 which was declared in the bundler development group. I added the BUNDLE_WITHOUT config variable to Heroku and it solved the problem.

The problem came back this morning and I noticed the change above in the Heroku docs.
