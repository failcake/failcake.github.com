---
layout: post
title: "Short commercial break"
date: 2013-10-28 23:24
comments: true
author: JHR
categories: [SysV,root exploit of the month club,four cats and wireless]
---
_Trigger warning: contains talk of horrible old Unix kit running horrible old Unix_

There's a good chance I'm probably a massive arsehole and I get paid for it. Which, I dunno, maybe I'm supposed to be
pleased with myself about it because being _disruptive_ is seen as a good thing these days. The last time I came across
people being described as such, they were the hyperactive (or just sugared-up) kids at junior school who seemed to be
convinced that it was all about them and if it wasn't they'd throw a massive strop and wander round with a lower lip
hung out like a soup-plate. I'm assuming a _disruptive technology_ doesn't have a howling fit in the middle of the
organic vegetable section of the supermarket if it doesn't get its own way, but then I wouldn't be surprised if it did.

The thing about the arseholedom isn't malevolent in the slightest, it's more a case of going up to someone and asking
them why they're nailing their legs to the table. You get this very weird selection of looks when you do things like
that. As if they're expecting you to come up with something sarcastic about using the contact adhesive on the shelf.
Then they'll say something like 'Well we've _always_ nailed our legs to the table in this department because it keeps
the bees from flying to Winchcombe.' Which, um, _okay..._ 

I mean, there's no answer to that. Especially when some manager piles out of the end office going "D'you want the bees
to go to Winchcombe? Do you? Because that's what's going to happen if you don't buck your ideas up and crack on with
that hammering."

But you have to try. You point to one of the chairs in the corner and suggest that using those would be much less
unpleasant. That's when the trouble _really_ starts. The manager goes pop-eyed and kicks off about 'You smart buggers in
IT think you know everything coming down here with your _ideas_ I don't have time for _ideas_ there's barely enough time
to send Bob here down to the hardware shop for more nails, what with the bleeding and the Tetanus jabs and now you want
us to cross-train to _chairs_ I'm glad you think we all sit around with glue-guns like you wasters someone should sort
you lot out once and for all.'

So you pull a chair over and they look at you like you just shat out a railway station.

What this is really about is that years ago (HP-UX 10.20 ago, in fact) I was given a HP9000 to look after. In poking
around the filesystem to see what dreadful sort of albatross I'd been handed, I found a whole pile of cron-jobs that ran
scripts to monitor sendmail and some more scripts that re-started sendmail and further scripts that tested the state of
earlier scripts. It all seemed a bit pointless because even then sendmail could more or less be left alone to generate
remote root exploits and sometimes deliver mail.

I asked one of the longer-serving chaps and he came over all leg-nailing. Apparently it wasn't to be touched because
sendmail was dreadfully unreliable and crashed every half hour.

I nodded, smiled and went off to throw away all the junk and upgrade the sendmail install to $latest.

It didn't crash.

The point being that writing long-lived daemon processes is really very well known science and instead of mucking about
with multiple layers of monitoring and backup, you're much better off making the daemon work right.
