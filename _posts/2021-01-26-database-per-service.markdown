---
layout: post
title:  "Database Per Service"
date:   2021-01-26 08:06:36 +0200
categories: databases distributed-systems
---
### 1. Introduction

If you have ever worked in a big company with small teams, you may have had the need to be as much isolated as possible 
from the others. Things like: releases, changes to a shared data model, high coupled code, scaling issues, etc... 
soon or later will be problems caused by the dependencies between teams. In this blog post I will explore some ideas that 
I used before, and we will see the possible problems (and solutions) that you could face if you do the same.

### 2. Separated repository (Idea 1)

The easier one, get your code, cut it and paste it in a separated repository but, **with a shared database**, in the end all of your 
entities are related with others from other teams and departments. This seems pretty straightforward at the beginning, 
but you have to think about your use case and the tradeoffs carefully.

* Are your services being used for thousands or millions of users? 
  * If the answer is no, maybe this approach could work for you.
* How many teams will be working on the database? 
  * If the answer is more than 1, maybe this approach is not for you.

So your team starts working in the new approach and realize that your code will be duplicated in your new repository and 
in the old one, you and your team decide to do some shared libraries to put common things together 
and available for everyone: entities, repositories and even services are going to be shared. You don't want to 
duplicate code right? Well... in the end no, but you could create a worse situation than duplicate code: the famous dependency hell. 

Ok ok, you have your new shine repository, your new CI/CD pipelines, you have created two or three libraries and 
then your team start developing services, but a problem arises, one of your core libraries have a bug, what are you going to 
do with those fifteen services you have now? Well you could update one by one the dependencies or create a special job 
to make an update for every service. But more problems appears:

* What happen if someone is using entities and repositories that shouldn't be used? 
  * You could move them to the service that belongs or don't accept the PR of that team. Set the boundaries of your libraries 
  is crucial.
* What happen if someone has to create an index in a 10 million records table/collection?
  * You have to create some sort of communication or ticket system to tackle this and create the index in the background.
* What if one or two of your services is having a lot of success?
  * You could scalate horizontally those services, but your database now can't handle that traffic.
* Are your libraries cohesive and low coupled? 
  * This is something hard to change once you have implemented a lot of code in them. But the solution is to refactor.

Even with all the possible problems that you could have with this approach, it's still a good solution if your use case is 
small, but if you are in another case it's worth to mention that there are other possible solutions.

### 3. Separated services and database (Idea 2)

Hablar sobre el approach de hacer servicios y comunicarlos por HTTP

### 4. Separated services and database with a message broker (Idea 3)

### Conclusion

### References

1. [Cohesion and Coupling: the difference](https://enterprisecraftsmanship.com/posts/cohesion-coupling-difference)
