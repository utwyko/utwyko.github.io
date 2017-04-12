---
layout: post
title:  "Rate-limiting API-calls with Reactor 3"
date:   2017-02-14 21:40:35 +0100
categories: Dev
tags: [reactive, reactor]
---

I've been working with [Reactor 3][reactor] recently. Reactor is a Reactive library similar to RxJava, but developed for Java 8 by the guys at Spring.
 
I ran into a situation where I had to call an external API for all items in a stream. That external API only allowed four requests a second, so I had to rate-limit the outgoing calls. Luckily, using Reactor, this is easily implemented.
 
We can use the [delayElements][delay-elements] method on the stream to limit the amount of emitted items per second.

{% highlight java %}
Flux.just("ID1", "ID2")
    .delayElements(Duration.ofMillis(250))
    .map(id -> apiClient.doSomething(id));
{% endhighlight %}

#### Update 12th April 2017 ####
This post has been updated to use the new [delayElements][delay-elements] method instead of the now deprecated [delayMillis][delay-millis-ref]. 

[reactor]: https://projectreactor.io/
[delay-millis-ref]: https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#delayMillis-long-
[delay-elements]: https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#delayElements-java.time.Duration-
