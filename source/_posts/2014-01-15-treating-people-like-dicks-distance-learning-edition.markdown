---
layout: post
title: "Treating people like dicks (distance learning edition)"
date: 2014-01-15 20:58
comments: true
categories: [proving a point for spite,arm-waving and speculation,all hardware is terrible]
author: JHR
---
Today one of the old Solaris boxes expired. Well, I say 'box' and 'expired'. I mean '1U
Solaris-X86-what-were-we-thinking machine' and 'fell into maintenance mode while I was eating breakfast'. And, in a
truly extraordinary amount of digression and rambling, when I say 'what were we thinking' I probably mean 'The kit had
actually managed to serve MySQL tolerably reliably for some 1500 days.'

I don't know if you lot remember the uptime wars, but they were medium sized in the late nineties. Rather like Sleeper
or Menswear, but with fewer annoying tunes and rather more waiting around. We learned better as soon as someone equated
long uptimes with being an obvious target for some bollix with a copy of Metasploit.

Anyway. A machine that hadn't been restarted since it was shoved in a rack, that was host to a pile of Solaris zones.
What could possibly have gone wrong with that?

It transpired that one set of binary logs or another had experienced a Jolly Interesting Time and had managed to confuse
the zpool enough that the alleged hypervisor had thrown a strop and gone into maintenance mode. Which, um, _okay..._

Thankfully there are no beard-fondling Solaris types around to tell me that the next move was a Bad Idea, but mucking
out the disks, clearing maintenance mode and restarting the beast looked to be the least-worst option.

That is until we discover that the running network config had never been written back to various bits of /etc and indeed
there were no build notes or valid excuses on either the deceased wiki or the somewhat shiny new one.

There now followed a swearing competition.

I suspect that what happened in 2009 was pretty much like what happened this morning. After the eighth or twelfth reboot, the
people wanting the databases back won over 'I would like to make this network config survive a reboot surely the
combined wit of the Sun/Oracle doc and three dozen assorted blogs and HOWTOs can't all be missing the vital something
or other that we can't spot either...'

It's still an unpleasant trick, though. 
