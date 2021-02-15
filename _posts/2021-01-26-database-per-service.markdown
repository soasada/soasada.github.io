---
layout: post
title:  "Database Per Service"
date:   2021-01-26 08:06:36 +0200
categories: databases distributed-systems
---
### Introduction

If you have ever worked in a big company with small teams, you may have had the need to be as much isolated as possible 
from the others. Things like: releases, changes to a shared data model, high coupled code, scaling issues, etc... 
soon or later will be problems caused by the dependencies between teams. In this blog post I will explore some ideas that 
I used before, and we will see the possible problems (and solutions) that you could face if you do the same.

### Separated repository (Idea 1)

The easier one, get your code, cut it and paste it in a separated repository, but **with a shared database**, in the end all of your 
entities are related with others from other teams and departments. This seems pretty straightforward at the beginning, 
but you have to think about your use case and the trade-offs carefully.

* Are your services being used for thousands or millions of users? 
  * If the answer is no, maybe this approach could work for you.
* How many teams will be working on the database? 
  * If the answer is more than 1, maybe this approach is not for you.

So your team start working on the new approach and realize that your code will be duplicated in your new repository and 
in the old one, you and your team decide to do some shared libraries to put common things together 
and available for everyone: entities, repositories and even services are going to be shared. You don't want to 
duplicate code right? Well... in the end no, but you could create a worse situation than duplicate code.

Ok ok, you have your new shine repository, your new CI/CD pipelines, you have created two or three libraries and 
then your team start developing services but a problem arises, one of your core libraries have a bug, what are you going to 
do with those fifteen services you have now? You could update one by one the dependencies or create a special job 
to make an update for every service (believe me that this is not easy), and you fall into the dependency hell but more problems appears:

* What happen if someone is using entities and repositories that shouldn't be used? 
  * You could move them to the service that belongs or don't accept the PR of that team. Set the boundaries of your libraries 
  is crucial.
* What happen if someone has to create an index in a 10 million records table/collection?
  * You have to create some sort of communication or ticket system to tackle this and create the index in the background. This 
  problem could potentially put down all services depending on the same database.
* What if one or two of your services is having a lot of success?
  * You could scalate horizontally those services, but your database now can't handle that traffic.
* Are your libraries cohesive and low coupled? 
  * This is something hard to change once you have implemented a lot of code in them. But the solution is to refactor.

Even with all the possible problems that you could have with this approach, it's still a good solution if your use case is 
small, but if you are in another case it's worth to mention that there are other possible solutions.

### Separated services and database HTTP (Idea 2)

Your services doubled their traffic, a new team comes to the company and problems are growing, you and your team start thinking 
about the idea of be self-contained, then database separation is the way to go. Be completely isolated makes a lot of sense 
if you want to scale your services and release them apart from the others. 

What are the trade-offs here? One could say duplication of data (it could be a problem if you have thousands of GB of data), 
another could say maintenance of another database (this could be true if you create the database in a separate database server), 
another says incremental of costs (development costs and more iron maybe it's needed), but the hardest trade-off here is the data synchronisation.

Knowing who will be the master and the slave is the first step, the following one is to think about the communication protocol 
between these services: HTTP or message broker, for this second idea we will choose the HTTP flavor, and the third step is to 
choose the information to be needed by the slave service, usually is the minimum required, a subset of the master data (e.g. 
email and address from a customer data table/collection).

In every operation to the data that change it, you must replicate this change to the slave service. Using an async HTTP client with a 
retry policy could work, but if your services have dependencies between more than one service this will be a mesh. Also, 
if one of the services is down, and you fail to sync your data (even with the retries), you need some kind of mechanism to 
continue in the same point you were at the beginning of the failure. On the other hand, due to the asynchronicity of the 
data sync process you will face another problem: eventual consistency, so you have to choose between synchronous 
data sync process or async one, depending on your use case.

Additionally, you have to maintain the retro-compatibility in the APIs, in the **Idea 1** this is done by communication 
channels between teams and telling them the change on the database, here is the same, and you could follow two ways, the first 
one is done in three steps always:

1. If you have to add a new field you don't have to do anything, if you have to update or remove one field, add a new field (with the updated info) and/or deprecated old one.
2. Communicate to other teams to let them update.
3. When they are done (or a due date is reached), remove the deprecated field of step 1.

The other way is to version the APIs, this is usually used for big changes in the API.

### Separated services and database Message Broker (Idea 3)

Finally, you want to avoid the data loss, failure in HTTP calls between long service chains and the scale problems of the 
**Idea 2**. A message broker comes to the table, instead of doing the synchronisation with HTTP calls you could create the 
channel with a publisher/subscriber tool, all the fancy features like failure tolerance, high availability and scalability 
are covered in such tools and will free you about all of your problems... but all of this is priceless? Sure not, the first 
thing that you have to know is that this solution is the most advanced one and the most complex one.

If you still think that this approach is for your use case, ask yourself the following question:

- Do I need Pub/Sub solution or queue one? 
  - To answer this you need to know what are the differences between the two approaches and then choose a tool for it. 
    A Pub/Sub tool is able to produce messages from more than one producer, and are consumed by more than one consumer, a 
    queue tool is more for a point-to-point communication. There are more differences but in general this is the most 
    important one, also there are differences between tools but this is another story.
    
When you have chosen the approach you need to follow the same path of **Idea 2**:
- Know who will act as master and who as slave.
- Maintain retro-compatibility with the flow Add > Communicate > Delete.
- Create the communication link between master and slave doing producer and consumer in both sides.

The possible problems that could appear could be:
- **Duplicate messages consumed:** What happen if your producers have a problem (network issue or a bug in the code) and 
  are sending duplicated messages? If your use case can't tolerate this situation you should build your consumers idempotent. 
  You could implement idempotent consumers by using idempotent code (I mean doing or use functions that are idempotent) or 
  implementing a cache giving a unique ID for each produced message, and when a message arrives a consumer you could check 
  against the cache if the message was consumed previously with ID comparison.
- **Producer failure:** If your application that is producing messages is down for a while, your consumers will be idle for 
  a period of time, you could monitor this situation and fix it as soon as possible.
- **Consumer failure:** The same happens if consumers are down, this will create a lag in the message consumption and 
  also should be monitored.

### Conclusion

Everything in this field is a trade-off, before choosing an idea or approach you must have a deep knowledge of the domain 
and the problem you are trying to solve, the only way of improving in this regard is to try and experiment but sometimes this 
is not possible. I had been working in projects where the **Idea 1** was a totally success and in others where it was a 
big big mistake, remember to not be drive by the hype of new tools and keep things simple.

### References

1. [Cohesion and Coupling: the difference](https://enterprisecraftsmanship.com/posts/cohesion-coupling-difference)
2. [Eventual Consistency In Plain English](https://stackoverflow.com/questions/10078540/eventual-consistency-in-plain-english)
