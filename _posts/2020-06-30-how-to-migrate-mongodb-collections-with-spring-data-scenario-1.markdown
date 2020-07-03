---
layout: post
title:  "How To Migrate MongoDB Documents With Spring Data, Scenario 1"
date:   2020-06-30 17:48:02 +0200
categories: kotlin spring-data
---
### Introduction
Sooner or later you will face with a situation where you need to remove, add or change the attributes of a MongoDB document. 
Unfortunately, there is no migration library for Java/Kotlin for that purpose (well this is not 100% true, mongobee is one 
but is no longer maintained). Moreover, what happens if you have a MongoDB collection of 30M documents? Are you going to execute a query 
to update the whole collection? Probably not, you should do some kind of lazy update.

I will illustrate how to effectively evolve a MongoDB document with Spring Data and do it in the lazy way. I will use 
three scenarios (in three different posts): when you are adding new attributes, when you are removing them and when you are updating them.

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

Another solution is to execute the confirmation logic **after loading an old user from the database.** How to do that? 
Well, first you need to recognize what an old user is, in our case an old user is one that does not have `emailConfirmed` flag (aka null). 
Second, you need some kind of mechanism to execute the confirmation logic when Spring Data loads a user from the database and save it again. Hopefully, 
Spring Data MongoDB >= 3.0 has a new `EntityCallback` called [ReactiveAfterConvertCallback](https://github.com/spring-projects/spring-data-mongodb/blob/master/spring-data-mongodb/src/main/java/org/springframework/data/mongodb/core/mapping/event/ReactiveAfterConvertCallback.java) 
that does exactly this:

{% highlight kotlin %}
@Component
class UserMigrator(private val applicationContext: ApplicationContext) : ReactiveAfterConvertCallback<User> {
    override fun onAfterConvert(entity: User, document: Document, collection: String): Publisher<User> = mono {
        val migratedUser = if (entity.emailConfirmed != null) entity else entity.confirmEmail()
        applicationContext.getBean(UserRepository::class.java).save(migratedUser)
        return@mono migratedUser
    }
}
{% endhighlight %}

### Conclusion

As you can see, it's pretty easy to do the migration with Spring Data, the only thing that could be improved here is the saving, maybe 
you want to decouple the saving from the migration if so, you could publish an event from here with the migrated user and subscribe to 
it in another part of the application, but this is another history. 

You can see the complete working code here: https://github.com/soasada/migrate-mongodb-collections-spring-data