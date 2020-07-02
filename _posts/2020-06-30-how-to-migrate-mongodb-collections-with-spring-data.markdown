---
layout: post
title:  "How To Migrate MongoDB Documents With Spring Data"
date:   2020-06-30 17:48:02 +0200
categories: kotlin spring-data
---
### Introduction
Sooner or later you will face with a situation where you need to remove, add or change the attributes of a MongoDB document. 
Unfortunately, there is no migration library for Java/Kotlin for that purpose (well this is not 100% true, mongobee is one 
but is no longer maintained). Moreover, what happens if you have a MongoDB collection of 30M documents? Are you going to execute a query 
to update the whole collection? Probably not, you should do some kind of lazy update.

In this post I will illustrate how to effectively evolve a MongoDB document with Spring Data and do it in the lazy way. I will use 
three scenarios: when you are adding new attributes, when you are removing them and when you are updating them.

### Scenario 1: Add new attributes to a document

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

In principle this could work, but **only for new documents**. What should you do with old ones? This is where the 
problems begin. 

The proper way to do it lazily, is executing some logic **after loading an old document from the database.** Usually, when you 
do that the attributes are unavailable, so you have two options here:

1. If the attributes could be populated, you have to implement the logic somewhere. Hopefully, Spring Data MongoDB >= 3.0 
has 
2. Otherwise, you return the loaded document to the caller and is the caller the responsible to update this attributes.

In our case, we could do the first approach with the  interface from 
Spring Data MongoDB >= 3.0, if not we should follow the second approach.

