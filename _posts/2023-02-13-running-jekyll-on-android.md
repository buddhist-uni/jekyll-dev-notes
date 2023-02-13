---
layout: post
title: Running Jekyll on Android
date: 2023-02-13 07:58 +0700
---

My primary dev machine (odd as this may sound) is my Android phone.
It's always with me, and it's plenty capable for jotting down quick notes like this one.

For any development on Android, you'll of course need to [install Termux](https://termux.dev/en/).

To run a Jekyll build (or even serve!) on Termux, I just needed to do a couple things:

1. `pkg install ruby`
2. `gem install bundler`
3. Add the following two lines to your `Gemfile`:

{% highlight rb %}
gem "jekyll-sass-converter", ">= 2.0", "< 3.0"
gem "webrick", "~> 1.7"
{% endhighlight %}

The first locks jekyll-sass-converter to v2 (as v3 uses Dart, which I haven't yet gotten to work on Android) and the second line explicitly asks for a hidden dependency.

And that's it!
Happy Jekylling on-the-go!
