---
layout: post
title:  "Combining Streams using Reactor 3 + Kotlin"
date:   2017-08-22
categories: Dev
tags: [reactive, reactor, kotlin]
---

I spend a day migrating a service that does a lot of aggregation from [RxJava 1][rxjava] to [Reactor 3][reactor]. The migration went pretty smooth, with the Reactor API being more elegant compared to RxJava in most cases. 

However, I struggled migrating the large `zip()` operations, where we merge async data from various sources into one big Observable. RxJava has `zip` that work like this:

{% highlight java %}
public static <T1,T2,T3,T4,T5,R> Single<R> zip(Single<? extends T1> o1,
                                   Single<? extends T2> o2,
                                   Single<? extends T3> o3,
                                   Single<? extends T4> o4,
                                   Single<? extends T5> o5,
                                   Func5<? super T1,? super T2,? super T3,? super T4,? super T5,? extends R> zipFunction)
{% endhighlight %}

While I think from a maintenance perspective, this if far from ideal, as a user, the resulting code becomes quite readable:

{% highlight java %}
Single.zip(singleA, singleB, singleC, singleD, singleE, 
           (a, b, c, d, e) -> new Aggregate(a, b, c, d, e))
{% endhighlight %}

In Reactor, the API unfortunately isn't as user-friendly. In the combinator function, you lose all type information which results in a long, ugly Function where we're forced to do manual casting.

{% highlight java %}
Mono.zip(array -> {
    A a = (A) array[0];
    B b = (B) array[1];
    C c = (C) array[2];
    D d = (D) array[3];
    E e = (E) array[4];
    
    return new Aggregate(a, b, c, d, e)
}, monoA, monoB, monoC, monoD, monoE)
{% endhighlight %}


There is also the `when` approach, which returns a `Tuple` that you can then transform. This approach is slighly more maintainable as it doesn't require manual casting, but is not that readable:

{% highlight java %}
Mono.when(monoA, monoB, monoC, monoD, monoE)
    .map(tuple -> new Aggregate(tuple.t1(), tuple.t2(), tuple.t3(), tuple.t4(), tuple.t5()))
{% endhighlight %}

In [Kotlin][kotlin], we can improve this by using the Tuple Extension Functions for Kotlin provided by Reactor. These [Kotlin extension functions][extensions] provide `component()` methods to the Reactor Tuples which allow us to use [Kotlin's Destructuring Declarations syntax][destructuring]. We need to manually import these extension functions, in my experience IDEA does not do it for you.

{% highlight kotlin %}
import reactor.util.function.component1
import reactor.util.function.component2
import reactor.util.function.component3
import reactor.util.function.component4
import reactor.util.function.component5

Mono.when(monoA, monoB, monoC, monoD, monoE)
    .map {(a, b, c, d, e) -> Aggregate(a, b, c, d, e)}
{% endhighlight %}

[reactor]: https://projectreactor.io/
[rxjava]: https://github.com/ReactiveX/RxJava
[kotlin]: https://kotlinlang.org/
[extensions]: https://kotlinlang.org/docs/reference/extensions.html
[destructuring]: https://kotlinlang.org/docs/reference/multi-declarations.html