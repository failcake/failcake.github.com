---
layout: post
title: "I had that Janov bloke in the back of my cab once"
date: 2013-02-24 15:47
comments: true
author: JHR
categories: [having it average,proving a point for spite,arm-waving and speculation]
---
Here's a non-technical thing that's been wandering round my head: brogrammers are more or less exactly what you can
expect in an environment run by old-school Unix admins. Or rather, they emerged as a species in reaction to an
environment which itself was a reaction.

I guess I'd better unpack that and provide some material so people can go TL;DR.

#### Brogrammers.
So (i). Brogrammer. You can go look that up on the internet, because that's what sensible people do. If they come across
a term or statement they're not sure about, they can poke about the internet for a bit, gather information from several
sources and perhaps come to a useful conclusion. It's not, y'know, _required_, but it's nice when it happens and makes
them look much less like dicks than the sort of people who'll just stand there going 'No! _Tell_ me what you mean!'

I would also ask you to go read this: [what your culture really says](http://blog.prettylittlestatemachine.com/blog/2013/02/20/what-your-culture-really-says), because it crystallised (or
began the process of precipitation or whatever) a lot of what this ramble may or may not be about. I have no particular
axe to grind with that piece because I am a white English bloke  in my mid forties, and if I've been a participant in
any of the scenarios listed I've not had the wit to realise it. It rings true, though. True enough that I suspect the
'if' in that preceding sentence is a 'when'.

<!-- more -->

So (ii). Old-school unix admin. There are a set of people out there who have failed to realise that the stories about
the 'Bastard operater from hell' are _parody,_  and that the Dilbert comics are a stark and existentialist warning from
a parallel universe where everything is terrible. More or less exactly like this one, in fact. Worse, there are a
younger set of people who've found the Usenet groups where the alleged greybeards let off steam by making up stories,
decided they want to be Unix Admins, but have cargo-culted precisely the wrong set of attributes in order for that to
happen. More or less like bad Chaos Magick through a wonky mirror.

#### Dickheads.
The end result of this is that you end up with a class of people who cultivate a surly attitude and firmly hold on to
the idea that they are the only ones in the business who know how to do anything computer-related correctly, and
because of that no-one should be allowed to fiddle with the servers (or the routers, as we have seen before) because
through the magic of circular logic they'll do something stupid and this is why we can't have nice things. However,
since one or other part of the business seems to make money via the act of repeatedly changing the things that are on
the servers, they will huffily and with much tutting allow carefully vetted changes to be made.

This is not to say that vetting changes is a _bad thing_, it's just that making it hard for people to change things is
not a useful substitute for having tested those changes in the first place.

Trying to work with martyrs like that is about as soul-destroying as trying to work with the sort of change-control
procedure which is merely the same attitude rendered as A Process.

The other thing that happens is that if you treat people like children, they'll act like children. Put in a
pr0n-blocking web-proxy at work and people will work out how to get around it so they can look at pr0n. It's an applied
version of 'Well it doesn't say _don't_ put washing-up liquid in the video-recorder'. The point that no-one with an
interesting and fulfilling job would want to look at pr0n during work hours is lost in sulky justification and 'I'll
show _them_' rationalisation.

[ Yes I have worked in shops where the NT blokes had topless-totty screensavers. It's, I don't know, not something that
sits well with me. ]

So now you've a set of people cast in the role of children, the old-school unix admin falls into the role of
long-suffering parent: "This server won't manage itself", "Don't leave your logs there", "I'll just clean up /tmp
myself, shall I?"

For anyone still reading, that is _massively_ dysfunctional. 

As an aside, a behaviour pattern that I have noticed _a lot_ is one where J. Random Dev will ask the admin(s) for
something-or-other (Shiny new webscale tech as mentioned on Hacker News perhaps) and will be told that no they can't
have that because no-one knows how to support it and had they thought about this other tech that looks a bit more
scalable? At this point J R Dev strops off to their Project Manager and explains that must have (shiny new webscale
tech) and that the admins were mean and said no. So the PM soon heaves to in the admin's cube and starts on about
business-critical functionality and unacceptable delays and if you recognise that as 'If your mum says no, go ask your
dad' then you win a rather grubby and worn Internets.

#### What is to be done?
Recognising what's going on is a really good start. I'm bound to say that the notion of DevOps culture is as good a
countermeasure as I have found so far. Other than that, patches welcome. I'm not convinced it's a solvable problem
without a reasonable cultural shift, and given the state of your organisation you may well not have that opportunity.

