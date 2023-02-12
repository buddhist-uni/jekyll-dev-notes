---
layout: toc_post
title: Advanced Liquid Recipes
date: 2023-02-12 20:17 +0700
---

## Introduction

After using Jekyll for a number of years, I've had to discover or invent a number of Liquid "recipes" (cocktails?) for doing common tasks that _should_ be easy in a sane language, but which are not obvious how to do in Liquid.

Since these (anti-?)patterns are under-documented elsewhere, I thought I'd collect them here in case this list is useful for others, especially those who are starting to use Jekyll / Liquid for serious development.

## Format

Each entry below will show off some Liquid code and what that code outputs:

{% highlight liquid %}
{% raw %}
{% assign hello = "world" %}
{{ hello }}
{% endraw %}
{% endhighlight %}

> world

It assumes you're already familliar with the fundamentals of writing [Liquid in Jekyll](https://jekyllrb.com/docs/liquid/).


## The Recipes: How to...

###  Make a literal string list

You cannot directly instantiate a list in liquid.
To make a list of literal strings, you'll have to use "split":

{% highlight liquid %}{% raw %}
{% assign mylist = "hello,world,you're,looking,great!" | split: "," %}
{{ mylist[0] }}
{% endraw %}{% endhighlight %}

> hello

### Filter a list to elements containing a substring

Don't be discouraged! `contains` works as you'd expect inside a `where_exp`:

{% highlight liquid %}{% raw %}
{% assign mylist = "hello,world,you're,looking,great!" | split: "," %}
{% assign filtered = mylist | where_exp: "item", "item contains 'e'" %}
{{ filtered[2] }}
{% endraw %}{% endhighlight %}

> great!

### Filter a list to elements containing a _variable_ substring

The above is all well and good, but what if the substring you're looking for is contained in a variable?

In that case, use {% raw %}`{% capture %}`{% endraw %} to build the filter expression dynamically![^eval]

[^eval]: Yes, this is basically a hidden `eval` function. With great power...

{% highlight liquid %}{% raw %}
{% assign mylist = "hello,world,you're,looking,great!" | split: "," %}
{% assign lookingfor = "r" %}
{% capture filterexp %}item contains '{{ lookingfor }}'{% endcapture %}
{% assign filtered = mylist | where_exp: "item", filterexp %}
{{ filtered[0] }}
{% endraw %}{% endhighlight %}

> world

### Write test cases using `capture`

Yes, the `capture` tag is super powerful!

One of my favorite uses is to write test cases!
You can simply `capture` the output of an `include` and then make sure it `contains` the expected string:

{% highlight liquid %}{% raw %}
My test:
{% capture output %}{% include my_include.md value=5 %}{% endcapture %}
{% if output contains 'expected string' %}Passes!{% else %}Fails!{% endif %}
{% endraw %}{% endhighlight %}

> My test: Passes!

### Strip your includes

When you call `{% raw %}{% include something.md %}{% endraw %}`, newlines will be automatically added to your include---even if they aren't in the original file!
Since this breaks certain markdown patterns, you may want to have a `strip`ped include.

You could could `capture` the output, then call `{% raw %}{{ captured_output | strip }}{% endraw %}` but this is cumbersome and ugly for the caller.
To make the include self-stripping, just make sure to put some whitespace-erasing {% raw %} `{%-`{% endraw %}s at the beginning and end of the include:

`_includes/inline_bold.md`:

{% highlight liquid %}{% raw %}
{%- comment -%}This include can safely be used in an inline manner{%- endcomment -%}
**{{ include.text }}**
{%- comment -%}Just make sure the include ends in a %- comment like this one!{%- endcomment -%}
{% endraw %}{% endhighlight %}

`_somewhere/else.md`:

{% highlight liquid %}{% raw %}
- my
- {% include inline_bold.md text="amazing" %}
- list
{% endraw %}{% endhighlight %}

> - my
- {% include inline_bold.md text="amazing" %}
- list


### Make a map

Sometimes you need a more advanced data structure than just a list.

To implement a Map, I usually use two parallel lists of keys and values.

{% highlight liquid %}{% raw %}
{% assign mykeys = "foo,bar" | split: "," %}
{% assign myvalues = "baz,bat" | split: "," %}
{% assign mykeys = mykeys | push: "new key" %}
{% assign myvalues = myvalues | push: "new value" %}
{{ mykeys[2] }} => {{ myvalues[2] }}
{% endraw %}{% endhighlight %}

> new key => new value

See the next recipe for how to access `my_map[some_key]`

### Return a value from an include

Sometimes you find yourself doing some messy calculation in multiple places, and (being a good programmer) you want to [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) out your code.
But Liquid `_includes` don't have `return` statements... so, can you actually write an `_include` which just performs a calculation?

Of course!

Your first thought might be to use `{% raw %}{% capture returnvalue %}{% include my_func.liquid param="foo" %}{% endcapture %}{% endraw %}` and to just output your return value inside the include.
And that will work!
But that's limited to returning strings, and usually it'll come with lots of added whitespace.

To return arbitrary variables, leverage the fact that **your local variables aren't actually local!!**

`_includes/ArrayDict_lookup.liquid`:
{% highlight liquid %}{% raw %}
{% assign value = nil %}
{% for k in include.keys %}
  {% if k == include.key %}
    {% assign value = include.values[forloop.index0] %}
    {% break %}
  {% endif %}
{% endfor %}
{% endraw %}{% endhighlight %}

`_somewhere/else.markdown`:

{% highlight liquid %}{% raw %}
{% assign mykeys = "me,you,us,them" | split: "," %}
{% assign myvalues = "Alice,Bob,World,Mars" | split: "," %}
{% include ArrayDict_lookup.liquid values=myvalues keys=mykeys key="us" %}
Hello {{ value }}
{% endraw %}{% endhighlight %}

> Hello World

Note how in the above, `value` was set in the `_include` but then _used_ by the calling template!
This means (among other things) that you should **never `{% raw %}{% {% endraw %}assign content = `** in an `_include` as this will nuke the parent page's `content`!

### Recurse in an include

You might think that local variables always leaking their scope would make recursive includes impossible.
It certainly makes them tricky!

While general, "local" variables are a no-go in a recursive include, you **can** use:
- `include.parameter` variables (as the include objects are indeed put on a stack)
- `for` loop variables (as the loop tracks its own state)
- Temporary variables (who are assigned and used on the same side of a recursive call)

For example, this `_include` takes a list of lists and outputs them as nested markdown:

`_includes/recursive_lists.md`:

{% highlight liquid %}{% raw %}
{% for item in include.list %}{% if item[0] %}{{ include.prefix }}Sublist:
{% assign p = '  ' | append: include.prefix %}{% include recursive_lists.md list=item prefix=p %}{% else %}{{ include.prefix }}{{ item }}
{% endif %}{%- endfor -%}
{% endraw %}{% endhighlight %}


`_somewhere/else.md`:

{% highlight liquid %}{% raw %}
{% assign list = 'Z,Y,X' | split: ',' %}
{% assign list = '1,2,3' | split: ',' | push: list | push: '5' %}
{% assign list = 'a,b,c' | split: ',' | push: list | push: 'e' %}
{% include recursive_lists.md list=list prefix="- " %}
{% endraw %}{% endhighlight %}


{% assign list = 'Z,Y,X' | split: ',' %}
{% assign list = '1,2,3' | split: ',' | push: list | push: '5' %}
{% assign list = 'a,b,c' | split: ',' | push: list | push: 'e' %}
Outputs:
```
{% include recursive_lists.md list=list prefix="- " %}
```

**Check your understanding**: What do you think would happen if that `{% raw %}{%- endfor -%}{% endraw %}` at the end of the above `_include` was instead `{% raw %}{% endfor %}{% endraw %}`?

## Conclusion

Liquid can be a frustrating language at times, but you can usually convince it to do what you want.
And if you can't, you can always write a `_plugin`!



**Footnotes:**

