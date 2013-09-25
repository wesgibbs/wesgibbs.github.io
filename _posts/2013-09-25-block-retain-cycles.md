---
layout: post
title: Objective-C block retain cycles
---

There are two parts to this issue. The first is whether to use `self` or a weak reference to `self` (`weakSelf`) within the block. The second is dealing with the possibility that `weakSelf` may become `nil` at any time.

# `self` or `weakSelf`

For the first question, ask: **what kind of reference does `self` have to the object that holds the block?**

If the reference is strong, use a weak reference within the block. Otherwise, a strong reference can be used within the block.

Some considerations when answering this question:

* If that object is defined as a property with a strong reference, then `self` will have a strong reference. Use a weak reference to `self` within the block.
* If that object is declared locally, it has a strong reference until the method exits. If no other strong references to the object are created before the method exits, then use `self` within the block. Otherwise, use `weakSelf`.
* If the block is a parameter to a class method, then use `self` within the block.

# Is `nil` ok?

For the second question, ask: what happens within the block if the `weakSelf` reference is `nil`?

Sending a message to `nil` is ok in objective-c. This may or may not be ok for your particular code though. Also, passing nil parameters may cause problems. If the outcome of `weakSelf` being `nil` is not guaranteed to be safe, you will want to start the block by creating a strong reference to the object pointed to by `weakSelf`. Then check if that reference is not `nil` and use `self` to do your work. When the block exits, the strong reference will go away.

# When in doubt

The safest way to proceed if it's not clear whether a strong retain cycle is going to be created:

* Create a `weakSelf` reference outside the block.
* Start the block by creating a strong reference to `weakSelf`.
* Use the `strongSelf` reference with a conditional that checks that it is not `nil`.

{% highlight objectivec %}
@implementation MyFirstViewController
...
  __weak typeof(self) weakSelf = self;
  MyOtherViewController *viewController = [[MyOtherViewController alloc] init...];
  viewController.doneBlock = ^{
    MyFirstViewController *strongSelf = weakSelf;
    if (strongSelf) {
      [strongSelf dismissViewController];
    }
  };
...
{% endhighlight %}

This is taken from Apple's [Transitioning to ARC Release Notes](https://developer.apple.com/library/mac/releasenotes/ObjectiveC/RN-TransitioningToARC/Introduction/Introduction.html#//apple_ref/doc/uid/TP40011226-CH1-SW4). (Search for `strongMyController` to find the example.)
