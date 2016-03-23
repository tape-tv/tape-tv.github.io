---
layout: default
title: "Well: A little BEM helper"
date:   2016-02-29 12:00:00 +0100
categories: ruby
---

At tape.tv, we started using [BEM][bem] over a year ago, to give our
markup some structure and our CSS some much-needed discipline.

One thing that can sometimes be hard to manage, though, is writing
view code without getting lost in a sea of classes and hidden typos.
BEM can be very repetitive.

{% highlight html %}
<section class="content">
  <header class="content__header">...</header>
  <div class="content__byline content__byline--dark">...</div>
  ...
  <footer class="content__signature content__signature--transparent">...</footer>
</section>
{% endhighlight %}

As a quick thought experiment, [Joe][joe] wrote [Well][well], a
Ruby DSL for writing Rails views with BEM-compliant classes.

It provides a couple of methods to `ActionView` which take away
from the user the responsiblity of getting all of the CSS classes
right.

{% highlight erb %}
<%= block(:section, 'content') do %>
  <%= element(:header, 'header') { '...' } %>
  <%= element(:div, 'byline', modifier: 'dark') { '...' } %>
  ...
  <%= element(:footer, 'signature', modifier: 'transparent') { '...' } %>
<% end %>
{% endhighlight %}

[Try it out][well] and let us know if it helps! As usual, we're open to
suggestions for improvements.

{% highlight shell %}
gem install well
{% endhighlight %}

The tape.tv dev team

[bem]: http://getbem.com
[well]: https://github.com/tape-tv/well
[joe]: https://corcoran.io
