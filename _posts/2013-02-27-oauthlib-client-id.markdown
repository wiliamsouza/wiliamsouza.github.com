---
layout: post
title: "OAuth 2.0 and client ID"
date:   2013-02-27 12:22:05 -0300
categories: python OAuth2
category: python
---

OAuth 2.0 spec [RFC 6749](http://tools.ietf.org/html/rfc6749#page-71) define
that `client_id` should be in the format `%x20-7E`.

What this mean?
---------------

Let me explain using some Python code.

{% highlight python linenos %}
>>> chr(int(0x20)), chr(int(0x7E))
(' ', '~')
{% endhighlight %}

But wait it is a range of characters.

{% highlight python linenos %}
>>> characters = []
>>> for char in range(int(0x20), int(0x7E)):
...     characters.append(chr(char))
... 
>>> characters
[' ', '!', '"', '#', '$', '%', '&', "'", '(', ')', '*', '+', ',', '-', '.', '/', '0', '1', '2', '3', '4', '5', '6', '7', '8', '9', ':', ';', '<', '=', '>', '?', '@', 'A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L', 'M', 'N', 'O', 'P', 'Q', 'R', 'S', 'T', 'U', 'V', 'W', 'X', 'Y', 'Z', '[', '\\', ']', '^', '_', '`', 'a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', 'k', 'l', 'm', 'n', 'o', 'p', 'q', 'r', 's', 't', 'u', 'v', 'w', 'x', 'y', 'z', '{', '|', '}']
>>> ''.join(characters)
' !"#$%&\'()*+,-./0123456789:;<=>?@ABCDEFGHIJKLMNOPQRSTUVWXYZ[\\]^_`abcdefghijklmnopqrstuvwxyz{|}'
>>>
{% endhighlight %}

So you can use the following code to create `client_id` for your `RESTful` API.

{% highlight python linenos %}
>>> import random
>>> rand = random.SystemRandom()
>>> length=30
>>> ''.join(rand.choice(characters) for x in range(length))
'*?>tN7a00_FMu{DKMxI<!->&I4GuU7'
>>> 
{% endhighlight %}

This code will create a `client_id` with 30 characters length.
