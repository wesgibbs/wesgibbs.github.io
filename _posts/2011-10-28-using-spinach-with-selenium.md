---
layout: post
title: Using Spinach with Selenium
disqus-url: http://wesgibbs.me/post/12036950233/using-spinach-with-selenium
---

When using Cucumber, I like the ability to switch Capybara drivers on a per scenario basis using tags. I can run with the Rack::Test driver for the default cases but switch to the Selenium driver for those scenarios that require javascript support.

I've been trying out [Spinach](https://github.com/codegram/spinach) lately and wanted the same ability. Here's what I did.

Add hooks in featues/support/hooks.rb to switch the Capybara driver based on a scenario tag.

{% highlight ruby %}

require 'capybara/dsl'

Spinach.hooks.before_scenario do |data|
  next unless data["tags"]
  tags = data["tags"].map{ |tag| tag["name"] }
  if tags.include?("@javascript")
    Capybara.current_driver = :selenium
  end
end

Spinach.hooks.after_scenario do
  Capybara.use_default_driver
end
{% endhighlight %}

Tag the features that require a javascript driver just like you would in Cucumber.

{% highlight ruby %}
Feature: Sign up


  @javascript
  Scenario: With correct credentials
    Given I am on the sign up page
    ...
{% endhighlight %}
