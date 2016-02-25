---
layout: post
title:  "Schrank: A simple YAML config loader"
date:   2016-02-15 12:59:55 +0100
categories: ruby
---

Whenever we set up a new Ruby application, we face the problem of how
to store access credentials and other settings in a way that:

1. Keeps secrets out of version control;
1. Can be changed per application environment;
1. Is not a huge pain in development.

Our solution to that problem is [Schrank][schrank].

It's a tiny gem that loads configuration from any YAML file. You can
provide a hash of defaults in case the file doesn't exist and, in development
only, the defaults are merged with the YAML.

This sounds strange at first, we know. But we like it this way because *there
are two types of configuration – things you are happy to have in version
control and things you aren't*.

In production and staging, our config is managed via Chef. We have data bags
per environment and we don't want to merge them with the defaults. In
development, we have our config split in two.
Values that are safe to put into version control, such as access tokens
that all the developers share, are written in as defaults. Other values,
such as personal AWS credentials, are read from the development YAML file.

Here's what our usage tend to look like:

{% highlight ruby %}
$config = Schrank.load(Rails.root.join("config/#{Rails.env}.yml")) do
  {
    foo: 'bar',
    baz: 420
  }
end
{% endhighlight %}

If you have the same problem and you want a really minimal solution,
we encourage you to check out [Schrank][schrank] and let us know what you
think!

The tape.tv dev team

[schrank]: https://github.com/tape-tv/schrank
