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
data class User(
    @Id
    val id: String,
    val email: String
) {
    constructor(email: String) : this(UUID.randomUUID().toString(), email)
}
{% endhighlight %}

Then, a new requirement comes to you and now users only can log in if they confirm the email. So far so good, you add 
a new attribute to your document to handle that:

{% highlight kotlin %}
@Document
data class User(
    @Id
    val id: String,
    val email: String,
    val emailConfirmed: Boolean?
) {
    constructor(email: String) : this(UUID.randomUUID().toString(), email, false)
    
    fun confirmEmail() = this.copy(emailConfirmed = true)
}
{% endhighlight %}

In principle this could work, but **only for new users**. What should you do with old ones? Your old users might not be 
able to log in into your application because they didn't confirm the email. Now you have a problem. 

A solution could be run an update query that adds this new attribute with a true value for each document in the collection. This solution 
only works with collections with few users, but if you have a very big collection, the query could take hours or even days. 
Are you gonna block the login for all those users during that time? No, you are not. 

Another solution is to execute the confirmation logic **after loading an old user from the database.** What is an old user? An old 
user means that the `emailConfirmed` comes with null

1. If the attributes could be populated, you have to implement the logic somewhere. Hopefully, Spring Data MongoDB >= 3.0 
has 
2. Otherwise, you return the loaded document to the caller and is the caller the responsible to update this attributes.

In our case, we could do the first approach with the  interface from 
Spring Data MongoDB >= 3.0, if not we should follow the second approach.

