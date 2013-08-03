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

Tag handler classes are much easier to create now using `SimpleTagSupport`, and using JSP handlers there is almost no need to code markup inside Java. 

But what happens when we need tag collaboration and markup in our tag? Suppose we have a tag that wraps some markup around its body. This tag needs to collaborate with some children tags, for example it needs to know how many of them there are, and these tags also render some markup around their bodies to produce the final result. Below there is a possible use case for this:

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

What I would like to do is create `panel-container` tag using a tag file for the markup. Something like the following:

{% highlight jsp %}
<%-- TAG DECLARATION --%>
<%@ tag body-content="scriptless"%>

<div class="panel-container">
    <jsp:doBody var="output"/>
</div>
{% endhighlight %} 

This is a pretty simple example of a container. But now let's assume that we need to know how many `panel` children we have for our tag to work. We don't want to lose our ability to use markup. I would the want something like the following:

{% highlight jsp %}
<%-- TAG DECLARATION --%>
<%@ tag body-content="scriptless"%>
<%@ taglib prefix="counter" uri="http://bizonos.com/counter" %>

<counter:set var="children" namespace="panel">
    <jsp:doBody var="output"/>
</counter:set>

<div class="panel-container">
    <div>This panel has <span>${children}</span> chlidren</div>
    ${output}
</div>
{% endhighlight %} 

and we can define children `panel` tags as the following:

{% highlight jsp %}

<%-- TAG DECLARATION --%>
<%@ tag body-content="scriptless"%>
<%@ taglib prefix="counter" uri="http://bizonos.com/counter" %>
<counter:increment namespace="panel"/>
<div class="panel">
    <jsp:doBody/>
</div>

{% endhighlight %} 

we rely on a couple of tags that collaborate. We can see how this works. `<counter:set>` initializes a counter with a namespace of its own. `<counter:increment>` increments a counter of the given namespace that needs to be present in a parent tag.

To achieve our purpose we can create a tag pair subclassing `SimpleTagSupport` with the only purpose of collaboration. Then we can make use of these tags inside our tag files to provide the markup. Our _general purpose collaborating tags_ should be configurable so we can use them inside different tag files. We can use tag variables for that. 

The tags that subclass `SimpleTagSupport` cannot collaborate using `findAncestorWithClass`, as they are not going to be descendant of each other strictly speaking (as you can see in the example above) but they can communicate using the JSPContext with a proper scope. 

Below you can find a base class holding the logic for this communication:

{% highlight java %}

package com.bizonos.util;
import javax.servlet.jsp.PageContext;
public class CounterTagSupport extends SimpleTagSupport {

    protected void removeRequestAttribute(String name) {
        getJspContext().removeAttribute(
                name, PageContext.REQUEST_SCOPE);
    }
    protected Object getRequestAttribute(String name) {
        return getJspContext().getAttribute(
                name, PageContext.REQUEST_SCOPE);
    }
    protected void setRequestAttribute(String name, Object value) {
        getJspContext().setAttribute(
                name, value, PageContext.REQUEST_SCOPE);
    }
}

{% endhighlight %}

and the concrete classes that collaborate:


{% highlight java %}
package com.bizonos.util;

import java.io.IOException;

public class CounterTag extends CounterTagSupport {
    
    private String var;
    private Integer parentCounter = null;
    private String namespace = null;
    private String counterVarName;
    
    public String getNamespace() {
        return namespace;
    }
    public void setNamespace(String namespace) {
        this.namespace = namespace;
        this.counterVarName = namespace + "counter";
    }
    public String getVar() {
        return var;
    }
    public void setVar(String var) {
        this.var = var;
    }
    @Override
    public void doTag() throws JspException, IOException {
        parentCounter = (Integer)getRequestAttribute(counterVarName);
        setRequestAttribute(counterVarName, 0);
        getJspBody().invoke(null);
        setRequestAttribute(
                getVar(), getRequestAttribute(counterVarName));
        if (parentCounter != null) {
            setRequestAttribute(counterVarName, parentCounter);
        }
        else {
            removeRequestAttribute(counterVarName);
        }
    }
}

{% endhighlight %}

{% highlight java %}
package com.bizonos.util;

import java.io.IOException;

public class CounterIncrement extends CounterTagSupport {
    
    private String namespace;
    private String counterVarName;
    
    public String getNamespace() {
        return namespace;
    }
    public void setNamespace(String namespace) {
        this.namespace = namespace;
        this.counterVarName = namespace + "counter";
    }
    @Override
    public void doTag() throws JspException, IOException {
        Integer counter = (Integer)getRequestAttribute(counterVarName);
        if (counter == null) {
            throw new Error("No parent counter found");
        }
        setRequestAttribute(counterVarName, counter + 1);
    }
}

{% endhighlight %}

We need a tag library descriptor for the variables to work.

{% highlight xml %}

<?xml version="1.0" encoding="UTF-8"?>
<taglib 
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
xmlns="http://java.sun.com/xml/ns/javaee"
xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-jsptaglibrary_2_1.xsd" version="2.1">
    <display-name>Counter Utils</display-name>
    <tlib-version>1.0</tlib-version>
    <short-name>counter</short-name>
    <uri>http://bizonos.com/counter</uri>
    <tag>
        <display-name>set</display-name>
        <name>set</name>
        <tag-class>com.bizonos.util.CounterTag</tag-class>
        <body-content>scriptless</body-content>
        <variable>
            <name-from-attribute>var</name-from-attribute>
            <variable-class>java.lang.Integer</variable-class>
            <scope>AT_END</scope>
        </variable>
        <attribute>
            <name>var</name>
            <required>true</required>
            <rtexprvalue>false</rtexprvalue>
            <type>java.lang.String</type>
        </attribute>
        <attribute>
            <name>namespace</name>
            <required>true</required>
            <rtexprvalue>false</rtexprvalue>
            <type>java.lang.String</type>
        </attribute>
        
    </tag>
    <tag>
        <display-name>increment</display-name>
        <name>increment</name>
        <tag-class>com.bizonos.util.CounterIncrement</tag-class>
        <body-content>empty</body-content>
        <attribute>
            <name>namespace</name>
            <required>true</required>
            <rtexprvalue>false</rtexprvalue>
            <type>java.lang.String</type>
        </attribute>
    </tag>
</taglib>

{% endhighlight %}

You can find a working example of all this [here](https://github.com/csierra/JSP-2.0-children-counter).

Hope this helps!
