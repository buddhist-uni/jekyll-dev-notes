---
layout: post
title:  "Lessons from One Year of Using GitHub-Pages"
date:   2021-05-18 11:12:54 +0700
---
Jekyll also offers powerful support for code snippets:

{% highlight liquid %}
{% raw %}
{% assign hello = "world" %}
{% endraw %}
{% endhighlight %}

{% assign mykeys = "me,you,us,them" | split: "," %}
{% assign myvalues = "Alice,Bob,World,Mars" | split: "," %}
{% include ArrayDict_lookup.liquid values=myvalues keys=mykeys key="us" %}
**Hello {{ value }}**

