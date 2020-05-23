---
layout: post
title: "[TIL] A new way to set debug break point in Python"
date:   2020-05-22 23:48:35 -0300
categories: python debug ipdb ipython
category: python
---

Always used `ipdb.set_trace()` to instruct my code to stop at desired breakpoint:

{% highlight diff %}
+++ b/tests/grpc/test_server.py
@@ -103,10 +103,11 @@ def test_server_create_order(database, tracer):
     request = CreateOrderResponse(
         code=str(order_code).encode(),
         identity="03303441965",
         amount_cents=100,
         status="validating",
         date=date,
     )
+    import ipdb; ipdb.set_trace()
     result = servicer.CreateOrder(request, None)
{% endhighlight %}

I already know about `breakpoint()` but used a sortcut(`F8`) in my `.vimrc`[1] to add the above code.

Today I decide to give `breakpoint()` a try. 

{% highlight diff %}
+++ b/tests/grpc/test_server.py
@@ -103,10 +103,11 @@ def test_server_create_order(database, tracer):
     request = CreateOrderResponse(
         code=str(order_code).encode(),
         identity="03303441965",
         amount_cents=100,
         status="validating",
         date=date,
     )
+    breakpoint()
     result = servicer.CreateOrder(request, None)
{% endhighlight %}

It start a `pdb` debug session.

{% highlight python %}
104             code=str(order_code).encode(),
105             identity="03303441965",
106             amount_cents=100,
107             status="validating",
108             date=date,
109         )
110         breakpoint()
111  ->     result = servicer.CreateOrder(request, None)
112
113         assert database.query(Order).one()
114         assert result.status == "validating"
(Pdb)
{% endhighlight %}

I whould like to user `ipdb` on my debug sessions.

Just set the following environment variable:

{% highlight shell %}
export PYTHONBREAKPOINT=ipdb.set_trace
{% endhighlight %}

Now it start an `ipdb` debug session.

{% highlight python %}
> /home/wiliam/Development/promotion/tests/grpc/test_server.py(111)test_server_create_order()
    110     breakpoint()
--> 111     result = servicer.CreateOrder(request, None)
    112 

ipdb>
{% endhighlight %}

[1] https://github.com/wiliamsouza/dot/blob/master/.vimrc#L105
