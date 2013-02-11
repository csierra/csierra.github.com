---
layout: post
title: "Organizing deployments with arquillian and shrinkwrap"
description: "This post shares some of my experiences organizing complex deployments"
category: 
tags: [arquillian, shrinkwrap, deployment]
author: Carlos Sierra
---

Arquillian and ShrinkWrap are undoubtedly great tools for tests automation. However one of the most difficult parts is usually deployment description. 

One of the _promises_ of arquillian is that you can create _real tests_, instead of having to mock services, since it brings the runtime to the test, or more strictly speaking, it runs the tests in a container where you have all these services available. 

Sadly, you normally need to configure the runtime to get it to work properly. That usually means a few files, XML or other, that need to be present in order for the test to run properly. Moreover, this is a task that crosses some virtual lines between development and systems administration. 

From my experience its very handy to have one or more separate classes describing the base deployment. This classes will be extended in the test deployment description in order to specify only the logic we want to test.

{% highlight java %}
public abstract class BaseDeployment {

    EnterpriseArchive ear;
    WebArchive war;
    JavaArchive jar;

    WebAppDescriptor webXml = Descriptors.create(WebAppDescriptor.class);

    public BaseDeployment() {
        ear = ShrinkWrap.create(EnterpriseArchive.class);
        war = ShrinkWrap.create(WebArchive.class);
        jar = ShrinkWrap.create(JavaArchive.class);

        jar.addClasses(String.class, Integer.class);
        war.addAsLibrary(jar);
        ear.addAsModule(war);

    }

    public Archive<?> build() {
        war.setWebXml(new StringAsset(webXml));
        return ear;
    }
}

{% endhighlight %}

Now, in the test, you can do

{% highlight java %}

public class ActualTest {

    
}
{% endhighlight %}


