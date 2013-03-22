---
layout: post
title: "Finding places to put things"
date: 2012-12-28 23:31
comments: true
author: JHR
categories: [epic handwave,post-hack rationalisation,difficult second version]
---

I suspect this blog-thing will just contain sporadic apologies for lack of content for most of its lifetime. 

Anyway.

This time the excuses have been brought to you by the words 'fail', 'power', 'generator', 'contactor', '250A supply',
'melted' and the phrases 'boot that filer from a different Vol0', 'can you smell smoke?' and 'Oh hell not again'.

As you might imagine, it's been busy and the DR plan has been tested and found interesting.

We're still Barberising and Hiera-ing up our shonky collection of Puppet modules. I'd say that they're getting less
shonky by the day, but it's taking longer than that. I hesitate to talk about 'patterns', because... Actually, I think
that's an example of self-taught-hacker anti-intellectualism, which is an equal amount of rubbish as its opposite.

So. The Barberis(ing|ed) pattern is a fine thing and, when used in combination with the wonder that is Hiera, allows us
to do more things in simpler code.

However. One of the modules that I'd been putting off refactoring (so 'patterns' are suspect but 'refactoring' is fine,
eh?) was the one that manages our NSD install and thus the DNS for quite a number of domains, some of which contain
rather popular websites.

[NSD is the authoritative-only nameserver daemon written by NLNet Labs](http://www.nlnetlabs.nl/projects/nsd/), who are a top bunch of chaps. We abandoned Bind
after there were one too many vulnerability notices.

I'd been putting off the work because the v0.1 module just drops the entirety of the zone-files directory under
../files/ and lets Puppet do the work of synchronising the files across the nameservers. It's not as if it's a terrible
thing to do at first glance - Puppet's file-serving means you can stop faffing with hand-brewed rsync scripts for
managing the out-of-band DNS data, and if you've got your Puppet tree in a sensible SCM, you get version control 'for
free'.

However (again), great lumps of org-specific data like that shouldn't really, we are told, be held within the module
tree. It's not necessarily obvious where the data should go, though. Nor is it terribly obvious how you connect it back
to the Puppet module and have changes in the one signal the other to perform tasks. 

Well, it is if you look at the right corners of the Internet, but this thing is mostly me groping around and trying
stuff out as a warning to others.

NSD installation and management goes in the now-Barberised NSD module.

This also deposits code that rebuilds the NSD config file when a domain is added or removed. And indeed the out-of-band
master list of domains, which semi-obviously has to travel separately from the zonefiles for $reasons. 

(It's about this time that someone-who-is-not-me would be going 'Why isn't all this domain gubbins in a nice database
somewhere, then all zone maintenance would be a simple 'SELECT mumble FROM yinglebart WHERE tewkesbury ISNT something'',
which would be very shortly before I hauled out the sarcasm throwing machine.)

The zonefiles live in a git repo of their own. That repo is cloned down onto the master DNS server(s) and kept current via the
magic of post-commit hooks. Meanwhile, there's a file resource in the NSD module which looks like this:

    file { '/var/lib/nsd3/.git/HEAD':
      audit   => content,
      notify  => Exec['rebuild'],
    }

    exec { 'rebuild':
      command     => '/etc/nsd3/code/refresh.sh',
      refreshonly => true,
    }


... Which is lifted wholesale from
[here.](https://github.com/jordansissel/puppet-examples/tree/master/unmanaged-file-notify) Either we've found one of the non-terrible use cases for this hack, or I'll be
writing another rambling post in a few months when I've had a better idea.

