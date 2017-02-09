---
layout: post
title: 'Working with nvie/gitflow: Considerations [Part 1]'
author: soutoner
---

If you search the internet looking for a successful git branching model 
you probably (very very likely) will find [Vincent Driessen's 
successful branching model](http://nvie.com/posts/a-successful-git-branching-model/).
 
This is a very simple (yet powerful) branching model that will fit 
the majority of your software projects. It has been proven to be very 
adaptable and that is the main reason why it is very extended along
the software community.

Are we appreciating enough Vincent's knowledge? I don't think so,
as he also built a [git extension (nvie/gitflow)](https://github.com/nvie/gitflow)
so we can adopt his flow easily in our everyday work.

The idea of this entry (or serie of entries) is not to write an 
extensive documentation of how to use this tool (this is much better explained in 
[Jeff Kreeftmeijer's blog](http://jeffkreeftmeijer.com/2010/why-arent-you-using-git-flow/))
but to comment everyday considerations that may not be so explicit when
you understand the basic branching model.

> Please note that a minimum understanding of the model is necessary
in order to understand the following words.

## Perishable branches

If you have already adopted this model, you would be dealing with a great deal
of branches on your daily basis.

Let's review, for example, what happens when we are doing a `release`.

{% highlight bash %}
git flow release start 0.1.0
Switched to a new branch 'release/0.1.0'

Summary of actions:
- A new branch 'release/0.1.0' was created, based on 'develop'
- You are now on branch 'release/0.1.0'

Follow-up actions:
- Bump the version number now!
- Start committing last-minute fixes in preparing your release
- When done, run:

     git flow release finish '0.1.0'
{% endhighlight %}

As in this case, almost every movement in this **branching** model will
create new branches. For example imagine that as a company rule, you
must upload this branch to the remote repository.

It's not any madness to think that branches must be removed when
they are no longer necessary. What is more, this is the default 
behaviour of this extension. Let's check what happens when we finish
our `release`. 

{% highlight bash %}
$ git flow release finish 0.1.0
Switched to branch 'master'
Merge made by the 'recursive' strategy.
 foo.txt | 0
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 foo.txt
Switched to branch 'develop'
Already up-to-date!
Merge made by the 'recursive' strategy.
Deleted branch release/0.1.0 (was fe5e000).

Summary of actions:
- Release branch 'release/0.1.0' has been merged into 'master'
- The release was tagged '0.1.0'
- Release tag '0.1.0' has been back-merged into 'develop'
- Release branch 'release/0.1.0' has been locally deleted
- Release branch 'release/0.1.0' in 'origin' has been deleted
- You are now on branch 'develop'
{% endhighlight %}

Looking at the **Summary of actions** please note what is happening
in the 4 and 5 steps.

{% highlight bash %}
[...]
- Release branch 'release/0.1.0' has been locally deleted
- Release branch 'release/0.1.0' in 'origin' has been deleted
[...]
{% endhighlight %}

Those branches, specifically `release` branches, will disappear locally,
even from the remote repository if they have been pushed.

Imagine now that you are a devops of your company, and you are trying
deploy this release. In our imagination exercise, we reach the point
where once that everything has been successfully deployed (and that means
that we have done a `git flow feature finish` before) we detect an horrible
 bug in production and we have to revert the state of the system. 
No problem at all, `git revert` (good name, isn't it?) will help us!

Once that we successfully reverted the release, or more specifically,
we have reverted the merge commit of the release branch, we can check
astonished how our release branch has disappear both locally and remotely.

No problem at all, we can continue with our chain of reverts (revert of 
a `git revert` commit) o even to do a cherry-pick of the merge commit.

What if the branch `release/0.1.0` was (at least) in the remote repository?
That would give us a lot of flexibility to handle this situation.

So a evident question arises: when is a branch *no longer valid*?

I can not answer that question as maybe some politics in your company
would override my thoughts. What if we never delete branches during this process?

That was previously thought by Vincent, so if we check the `help` of the
command, we can find two insanely useful flags that will save us a lot of 
headaches:

{% highlight bash %}
$ git flow release finish --help
usage: git flow release finish [-h] [-F] [-s] [-u] [-m | -f] [-p] [-k] [-n] [-b] [-S] <version>


    Finish a release branch
    
    [...]
    
    -k, --[no]keep        Keep branch after performing finish
    --[no]keepremote      Keep the remote branch
    --[no]keeplocal       Keep the local branch
    -D, --[no]force_delete
                          Force delete release branch after finish
    [...]
{% endhighlight %}

Any of those options will help us in our journey through this awesome
branching model without taking into account the flexibility of preserving
 "old" branches.

## Summary

At the end, we can check that the extension is very polished, so even
when this behaviour may not be so explicit in the branching model,
it has many flag that can help us to have a very personalized experience.

