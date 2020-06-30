---
layout: post
title:  "How To Migrate MongoDB Documents With Spring Data"
date:   2020-06-30 17:48:02 +0200
categories: kotlin spring-data
---
### Introduction
Sooner or later you will face with a situation where you need to remove, add or change the attributes of a MongoDB document. 
Unfortunately, there is no migration library for Java/Kotlin for that purpose (well this is not 100% true, mongobee is one 
but is no longer maintained). 

In this post I will illustrate how to effectively evolve a MongoDB document with Spring Data. 

### Scenario

Imagine that you the have the following document: 

{% highlight kotlin %}
@Document
data class Person(
    @Id
    val id: String,
    val name: String
) {
    constructor(name: String) : this(UUID.randomUUID().toString(), name)
}
{% endhighlight %}

Then, a new requirement comes to you and you have to add a couple of attributes to audit this document. So far so good you add 
them:

{% highlight kotlin %}
@Document
data class Person(
    @Id
    val id: String,
    val name: String,
    @CreatedDate val createdAt: Instant = Instant.now(),
    @LastModifiedDate val updatedAt: Instant = Instant.now()
) {
    constructor(name: String) : this(UUID.randomUUID().toString(), name)
}
{% endhighlight %}

The problem that you have now is