---
layout: post
title:  "Database Per Service"
date:   2021-01-26 08:06:36 +0200
categories: databases distributed-systems
---
### 1. Introduction

If you have ever worked in a big company with small teams, you may have had the need to be as much isolated as possible 
from the other teams. Things like: releases, changes to a shared data model, high coupled code, scaling issues, etc... 
soon or later will be problems caused by the dependencies between teams. Then your team starts brainstorming about how 
to get rid of these dependencies.

### 2. Separated repository (Idea 1)

Someone proposes to have a separated repository with all of your code but, with a shared database, in the end all of your 
entities are related with others from other teams and departments. This seems pretty straightforward at the beginning, 
but you have to think about your use case, and the tradeoffs of this approach.

Are your services being used for thousands or millions of users? If the answer is no, maybe this approach could work for you.

So your team starts working in the new repositories and realize that your repositories and entities will be duplicated in 
your new repository and in the old one. So you and your team decide to do some shared libraries to put common things 
available for everyone, things like database entities, repositories and even services are going to be shared. You are not 
duplication code right? Well... in the end no, but you are entering in another situation that could be even worse than 
duplicate code: the famous dependency hell. 

Ok ok, you have your new shine repository, your new CI/CD for that repository, you have created two or three libraries and 
then your life is easy, time passes and you have ten, fifteen or even more services that are build on top of those libraries, 
what happen if some of your core libraries have a bug? what happen if someone is using entities and repositories that shouldn't 
be used? what happen if someone have to create an index in a 10 million records table/collection? For sure, 
you will have some of these problems without taking into account things like scalability because your use case is not for 
that big amount of user usage.

Another caveat, be sure that the ones that are going to do the libraries use a low coupling and high cohesion design. 

Even with all the possible problems that you could have with this approach, it's still a good solution if you do it right, 
but if you are in another case, where you need full isolation (for releases, to unblock your team, etc...) and you are 
experimenting scalability issues and your bottleneck is the shared database, maybe you should consider a second approach...

### 3. Separated services and database (Idea 2)

Hablar sobre el approach de hacer servicios y comunicarlos por HTTP

### 4. Separated services and database with a message broker (Idea 3)

### Conclusion

### References

1. [OVH disable cloud-init for hostname](https://docs.ovh.com/gb/en/public-cloud/changing_the_hostname_of_an_instance/)
