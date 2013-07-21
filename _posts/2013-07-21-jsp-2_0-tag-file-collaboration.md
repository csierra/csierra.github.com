---
layout: post
title: "Tag collaboration using JSP 2.0 tag files"
description: ""
category: posts
tags: [jsp, 2.0, java, tag files, collaboration, count, children]
language: en
author: Carlos Sierra
---

One of the best things of JSP 2.x are, without any doubt, tag files. With tag files we can very easily reuse our view logic without having to embed any markup inside any Java class, or having to resort to different tricks to include other JSPs.

Tag handler classes are much easier to create now using _SimpleTagSupport_, and using JSP handlers there is almost no need to code markup inside Java. 

But what happens when we need tag collaboration and markup in our tag? Suppose we have a tag that outputs some markup to the browser. This tag needs to collaborate with some children tags, for example it needs to know how many of them there are, and these tags also render some markup around their bodies to produce the final result. Below there is a possible use case for this:

{% highlight jsp %}
<body>
    <h1>Hello World!</h1>
    <bizonos:panel-container>
        <bizonos:panel>a</bizonos:panel>
        <bizonos:panel>b</bizonos:panel>
        <bizonos:panel>
            <bizonos:panel-container>
                <bizonos:panel>1</bizonos:panel>
            </bizonos:panel-container>
        </bizonos:panel>
        <bizonos:panel>c</bizonos:panel>
        <bizonos:panel>
            <bizonos:panel-container>
                <bizonos:panel>x</bizonos:panel>
                <bizonos:panel>y</bizonos:panel>
            </bizonos:panel-container>
        </bizonos:panel>
        <bizonos:panel>d</bizonos:panel>
    </bizonos:panel-container>
</body>

{% endhighlight %} 

