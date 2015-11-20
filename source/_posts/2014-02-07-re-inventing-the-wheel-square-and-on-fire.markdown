---
layout: post
title: "re-inventing the wheel - square and on fire"
date: 2014-02-07 20:27
comments: true
categories: [proving a point for spite,post-hack rationalisation,old people are tiresome]
author: JHR
---

I am incompetent and I can't make Vagrant work. 

At least, that's the excuse I've been making for not joining in with the
rest of the floor in using that for Puppet (among other things) development. Instead, my dev-rig is a VM running as a
puppetmaster that's tracking the changes I make to a given branch in our Git repo via the magic of post-commit hooks and
another VM in which I can run 'puppet agent --debug --blah --server (first VM)'. Once in a while I remember to blow away
the second VM so I can make sure everything builds in the right order. However, even with snapshots it's just slightly
too painful to happen regularly.

Meanwhile, quite a lot of the recent developments at Future have involved rigs of between eight and twelve boxes.
Generating a worthwhile test/dev version of one of those is rather tiresome because even if you've got the spare
horsepower lying about, you have to spend yea-long wiring it all together, sanitising the static and/or test data and
it all quickly gets completely _oh god why did i even get out of bed i should have been a farmer like my dad mind you if
i had done that i'd have been out of bed at six to go feed the sheep on long barrow bank perhaps not after all..._ 

So when a mildly broken Dell R620 arrived back from one of the DCs coincidentally with me wanting to have a play with
this Docker business, it all seemed a bit convenient.

I am incompetent and I can't make Docker work. On Debian.

[LXC](http://linuxcontainers.org/), on the other hand, was slightly simpler than falling off a log.

Given that it's simple to build a puppetmaster that's the same as one of the live ones, and that all the machine config
I currently care about is in a manifest, it should be pretty easy to generate a container and have it puppet itself up
tolerably quickly.

This indeed turned out to be the case. However, having to hand-allocate IP addresses and fiddle about with container
naming such that they picked up the configs in use on the live rig was all a bit too hands-on and really not what I
wanted.

[DNSMasq](http://www.thekelleys.org.uk/dnsmasq/doc.html) fixed the first problem. It is a surprisingly useful tool.

A rakefile which read a list of made-up machine names, generated softlinks to the actual hiera node configs and then
instantiated the relevant containers fixed the second.

I also spent _quite some time_ building a Wheezy 'image' that minimised apt-get as much as possible.

Result - fully puppeted containers come up in circa a minute. Somewhat longer if you have to install PHP.
If I didn't have quite such a rational hatred of golden images and all who sail in them, it would likely be faster
still.

The next part is a bit fiddly.

The example problem I now have is that some parts of my collection of yea-many VMs want to connect to other parts. For
instance if I have a redis slave, I need to know what the master's IP address might be during the puppet run. At Future,
we generate a location fact and use that in our hiera, er, hierarchy to configure things like message brokers,
smarthosts and DNS ordering. I could just add yet another location - testbox, or something - allocate a block of IP
space and then add some extra indirection. And then I could do that again for each person and/or project that wanted to
run up a test-rig. At which point one has just run into a behaviour pattern that should probably be named 'It's OK, I
can fix that for you.'

I first came across this in, er, 1991 when doing some NHS-related coding. One of the other chaps had written a thing
which had to deal with, oh I don't know, ten items or something. Because he was a forward-thinking sort, he allocated
sixteen slots in his array and beetled off all smug for a coffee and a corned beef sandwich. As you might expect, a few
months later one site or other had a list of seventeen items and a bug report. 'It's OK, I can fix that for you!' went
our chap and expanded the array to the clearly ludicrous value of some twenty-three slots...

There's scope for an Eric Berne knockoff book of tiresome technical behaviour antipatterns, isn't there?

Anyway, I'm using DHCP, and I wanted the entire edifice to work with little or no extra typing.

CoreOS's [ETCD](https://github.com/coreos/etcd) looked like a good fit. Emit salient facts to etcd database when bringing up (say) redis master, then
query same via [Garethr's hiera-etcd](https://github.com/garethr/hiera-etcd) when bringing up the slave. Profit!

That bit did take a little tinkering to get right.

It seems to me that the notion of a reactive puppet configuration is really rather interesting. Other people may well be
screaming in terror and jabbering about things like 'deadly embrace' and 'terrible feedback loops are fine for the Jesus
and Mary Chain (Or A Place to Bury Strangers if that's too retro for you) but have no place in a theoretically stable
configuration.' However, just as a top-down decision process enforced by rigid hierarchy is a hateful idea for a
workplace environment, so it is for a machine environment.

TL;DR - code in [Github](https://github.com/FuturePublishing/model-village), patches welcome.

