---
layout: post
title: English motherf***er! Do you test with it?
---

I tested with Cucumber for years. I was resistant at first, then I drank the Kool-Aid and it was good.

Two years ago I stopped testing with Cucumber. And it was better.

I don't regret my time with Cucumber. I believe it made me a better developer and a better tester. There's one simple reason for that: describing my tests in English<sup><a href='#foot1'>1</a></sup> first.

Cucumber adherents have always been vocal about the value of tests written in plain language: they can be read and understood by stakeholders. Some have gone so far as to say they can be written by stakeholders. The claim is on Cucumber's Github page: tests in plain language can be read by anyone on your team.

They are missing the point. I'm not the first to be critical of Cucumber, of course. But here's why they're wrong.

The value of Cucumber and natural language testing is that it forces you (the developer) to think about your integration tests in the language of your users before you write any code. Nowhere in that statement did I reference a stakeholder or team member.

Once you've done this and written the code to make the test pass, the value of those tests being in plain language falls to near-zero. No stakeholder is going to read them. They certainly won't write any. No one else on your team that is not a developer is going to read them. I worked at a popular Rails consultancy for years and never once did we have a client read — much less write — a test.

So what's the point? Cucumber is not a lightweight testing option, after all. It's a heavy piece of machinery. Embracing the DSL, the support files, your growing library of step definitions<sup><a href='#foot2'>2</a></sup>, all this comes at a price. So if the value of natural language tests doesn't hold up over time, why do it in the first place?

Because thinking about the problem in your spoken language makes your product better. Here's why.

1. It forces you to interact with your product the way your user will.
These are integration tests. They describe features from the point of view of a user. What is their experience using this feature? What is their experience when something goes wrong? Using natural language surfaces bad interaction design. It sheds light on interactions that don't contribute to any business objectives. If typing out the steps a user needs to take in order to accomplish something feels cumbersome to you, there's a good chance that doing those steps will feel cumbersome to your user.

1. It pushes you to a level of interaction your wireframes or design comps likely missed.
You'll find that most wireframes and design comps have an affinity for the happy path. The bulk of a good test suite concerns itself with what's off the happy path. Describing those cases in plain language may be the first time those conditions have been given any real thought. It's good to do that before you start coding.

1. It prevents you from thinking about implementation.
If you can't describe the process from a user's point of view in English without including implemenation details, then you should revisit your design.

Natural language tests have tremendous value, it's just not the value you hear trumpeted so often. And you don't need Cucumber with all of its overhead to realize that value.

I write my integration tests in RSpec these days. I write my tests in Ruby and that's great because that's what I code in. But now my RSpec integration tests have lots of comments in them, and those comments will look familiar to anyone who's spent time with Cucumber.

{% highlight ruby %}
# Given Bo and Luke are caring for Jesse
# And Luke elects to not receive the content of journal entries in his notifications
# When I sign in as Bo
# And I create a journal entry
# Then an email is sent to Luke
# And it does not contain the content of the journal entry
{% endhighlight %}

I start a new test by writing — in plain language — what the preconditions are, what steps the user should take, and what the outcome should be (Given-When-Then). It allows me to think about the interaction like a user. Then I add the Ruby code to make each piece work. The parts that get repeated get extracted to support modules (similar to step definitions, but these are just plain Ruby modules, no Regexp ranking logic to be burned by).

Don't fall for the hype about tests written in natural language being produced or consumed by business people. It's just not that common. It's also a distraction from the real value of natural-language-first integration testing: it's the closest you'll come to interacting with your product the way your real users will without actually building it (short of a functional prototype). It informs the design. It makes you a better tester.

<div class='footnotes'>
  <div class='footnote'>
    <sup><a id='foot1'>1.</a></sup>
    I say English because that is my native language. Gherkin makes the Cucumber DSL available in many languages.
  </div>
  <div class='footnote'>
    <sup><a id='foot2'>2.</a></sup>
    Who tests the tests?
  </div>
</div>
