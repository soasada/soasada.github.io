---
layout: post
title:  "Maintain Backward Compatibility"
date:   2021-04-20 19:07:11 +0200
categories: good-practices
---
### Introduction

Sometimes we forget about our clients, by clients I mean the other "party" that is using our code/database/message broker/HTTP API 
or whatever, and we change something in a particular feature that breaks their system. Also, is typical to have two (or more) services/modules/classes/components 
that depend on each other and have different release chains, who will be the first one? If you release one after the other 
it will start failing, and you can't release at the same time, for this reason you **must ensure backward compatibility**. I 
will talk about it through this post.

### The New Feature

When a new feature comes and is related with an existing one, we have to know if implies changes in the fields/API/methods. 
If implies updating or removing something, **we should keep the old one as it is** and create a totally new feature 
(in the case of updating) or tell to our clients that this specific feature won't be available anymore (in case of removing). 
After that, we must deprecate the old one to let everyone knowing about it.

### The Time Window

One of the bad things, about maintaining backward compatibility, is that in some point in time you will have to maintain two versions 
of the feature that you are changing: the old one and the new one. In order to get rid of the old one as soon as possible, you have 
to **set a time window between you and your clients** to let them update, and after it, you will be able to remove the old version.

Bear in mind, the possibility of being your own client. In such case, you don't need the time window, and you will be the 
responsible of updating to the new version. Right after the update is done, you will be free to remove the old version.

### The Removal

Once everyone is updated, you have to remove the old version of the feature. In compiled languages it's an easy task, you 
remove the feature and something won't compile, but in interpreted ones the only thing that ensures that the old feature is 
removed are tests. Well, also you could have a bunch of TODO comments to remind you or your coworkers to remove the things. 
In both cases, you should have some tests failing that will help you on remove things.

This step could be skipped, but if you do, **your codebase will start to rot** at lightning speed.

### Conclusion

As you can see, the process of maintaining backward compatibility is always the same:

1. Add the new feature and deprecate the old one.
2. Set time window for your clients (if any) or update to the new feature.
3. Remove old feature.

This process could be use when evolving an RESTful API, a database table or an event schema. **But as usual, use it when makes 
sense**.

### References

1. [The Limited Red Society (2010)](https://www.infoq.com/presentations/The-Limited-Red-Society)
2. [API Expand-Contract Pattern (2014)](https://martinfowler.com/bliki/ParallelChange.html)