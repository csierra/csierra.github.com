---
layout: post
title: "Loosely comparing maps using Groovy"
description: ""
category: posts
tags: [tests, groovy, hashmaps]
language: en
author: Carlos Sierra
---

Sometimes is useful to be able to compare maps loosely. 
For example you may have some rest services that you want to test. This tests may very likely involve creation and modification of entities. To be able to assert that an operation was performed correctly, you may only want a subset of the map to be checked, for example excluding ids, timestamps or whatever other data that may not be inferred upfront. Besides we will very likely find lists of maps and nested maps. 

We can achieve this comparison very easily using Groovy categories and overriding equals method on Map interface, like so:

{% highlight groovy %}
class Loose {

    def static equals(Map one, Map two) {
        if (!two.keySet().containsAll(one.keySet())) return false
	one.every { it.value == two[it.key] }
    }
}
{% endhighlight %}
Using this category we are making equals to check only those keys that are present on the lefthand side map of the equality. The values of those keys are then compared. If some of those values are indeed maps, Groovy will use this same logic for comparing those nested maps. 

One caveat of using this approach is that equality is a very loaded concept and we are breaking its simmetry here (since a might be equals to b but the opposite will not hold most of the times). Luckily we need to declare the boundaries of our code where we expect this logic to apply. 

{% highlight groovy %}
use(Loose) {
   assert [a: 1, b: 2] == [a: 1, b: 2, c: 3]
}
{% endhighlight %}

Since lists use equals operator as well to compare their values, we can use this same approach to compare lists of maps. Just make sure you sort the lists so you get to compare the pairs you want to compare. 

Hope this helps!

