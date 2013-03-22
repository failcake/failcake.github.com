---
layout: post
title: "A hazelnut in every bite"
date: 2012-04-19 15:20
comments: true
author: JHR
categories: [post-hack rationalisation,proving a point for spite,epic handwave]
---

An obvious question is 'Why on earth bother with all this message-wanging gubbins? Isn't mail and/or SSH good enough for you?' to which the glib answer is 'No it isn't.'

A longer answer involves spotting the really obvious problem in my last post. That is 'Ok, so you've bodged in some code that'll auto-update your live puppetmaster tree on a commit to that repository. What are the chances that said commit is b0rked and causes an epic cake-fail?'

Well, in theory you're committing to a develop branch which only one or two of your machines are following, because that's the entire point of having the dynamic branch rig in the first place. However, PEBCAK happens and perhaps you should have a Jenkins instance that sanity-checks (as much as is possible, anyway) the puppet code that's just been committed.

Hurrah! Problem solved! Let's have one of those!

I don't know how you'd plumb something like that into another environment, but this is how it works here:

* Configure the stomp-jenkins daemon to listen for _puppet-environments_ commits on _future.git.commits_ (c0dez available in the usual place)
* [Configure Jenkins for Puppet and puppet-lint](http://vstone.eu/puppet-modules-in-jenkins/)
* Configure Jenkins to emit a message on (say) _future.jenkins.success_ if the tests pass. (c0dez for that available ditto)
* Configure the stomp-git daemon on your puppetmaster to listen on _future.jenkins.success_
* Er, _profit._

Obviously enough, chaining in extra bits or constructing side-chains is relentlessly trivial.

You could probably do it with a massive make or rake file, too. 
