---
layout: post
title: "Organizing deployments with Arquillian and Shrinkwrap"
description: "This post shares some of my experiences organizing complex deployments"
category: posts
tags: [arquillian, shrinkwrap, deployment, reusable, organize]
author: Carlos Sierra
---

Arquillian and ShrinkWrap are undoubtedly great tools for tests automation. However one of the most difficult parts is usually deployment description. 

One of the _promises_ of arquillian is that you can create _real tests_, instead of having to mock services, since it brings the runtime to the test, or more strictly speaking, it runs the tests in a container where you have all these services available. 

Sadly, you normally need to configure the runtime to get it to work properly. That usually means a few files, XML or other, that need to be present in order for the test to run properly. Moreover, this is a task that crosses some virtual lines between development and systems administration. 

From my experience, sometimes its very handy to have one or more separate classes describing the base deployment. This classes will be extended in the test deployment description in order to specify only the logic we want to test.

{% highlight java %}

import org.jboss.shrinkwrap.api.Archive;
import org.jboss.shrinkwrap.api.ShrinkWrap;
import org.jboss.shrinkwrap.api.asset.StringAsset;
import org.jboss.shrinkwrap.api.spec.EnterpriseArchive;
import org.jboss.shrinkwrap.api.spec.JavaArchive;
import org.jboss.shrinkwrap.api.spec.WebArchive;
import org.jboss.shrinkwrap.descriptor.api.Descriptors;
import org.jboss.shrinkwrap.descriptor.api.webapp30.WebAppDescriptor;

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

        webXml.version("3.0");
        //A lot of reusable descriptions here
    }


    public Archive<?> build() {
        /* Finally assemble the deployment so we allow web application description extension
         * from the test */
        war.setWebXML(new StringAsset(webXml.exportAsString()));
        
        // return the actual deployment
        return ear;
    }
}

{% endhighlight %}

Now, in the test, you can do

{% highlight java %}
import org.jboss.arquillian.container.test.api.Deployment;
import org.jboss.arquillian.junit.Arquillian;
import org.jboss.shrinkwrap.api.Archive;
import org.junit.Test;
import org.junit.runner.RunWith;

@RunWith(Arquillian.class)
public class ActualTest {
    
    @Deployment public static Archive<?> getDeployment() {
        return new BaseDeployment() { {
            war.addClasses(Number.class);
            webXml.createServlet().servletClass(Number.class.getName());
            //Etc...
        } }.build();
    }
    
    @Test public void test() {
        //Test code
    }
}
{% endhighlight %}

The design of Arquillian and ShrinkWrap allows us to build our deployments in this way, thus hiding a bit the internals of our deployment and making it a bit more reusable. 

We could even go further and parameterize the deployments or allow only some _extension points_ playing with the visibility of the fields and method we want to expose.


