---
layout: post
title: "Implementing Business Domains models using functional approaches in Java"
description: "In this post I will analyze a functional approach to implementing business domains that are stored in a database"
category: posts
tags: [java, domain, business, models, functional, persistent]
author: Carlos Sierra
---

So in these approach we are not passing model instances between the user code and the services. Actually there are no "offical" model types. There exist types that are handled to the functions passed to the services as arguments and then, these functions, will operate on the types to generate the ouputs needed by the client code.

The main benefit to this approach is that we don''t need to pass state through the interface boundaries.


