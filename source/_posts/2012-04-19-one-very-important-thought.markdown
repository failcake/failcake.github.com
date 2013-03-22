---
layout: post
title: "One very important thought"
date: 2012-04-19 12:24
comments: true
author: JHR
categories: [terrible excuses for Ruby,post-hack rationalisation,Boards of Canada]
---

I have been hacking on stomp-git over the past couple of days. Mostly because there may or may not be a lurking nasty with the way it sometimes stops listening on its, er listen topic, but
also because it has needed a deal of de-Futuring and general fettling so it doesn't make proper coders claw their own faces off in horror.

And because I had forgotten how I'd made it work when it came to the svn->git rollout of one of our major sites.

Thus it seems as good a time as any to explain how I think it should work and indeed why it works like that.

<!-- more -->

The elevator pitch (of sorts) is that if you've got a message-bus that connects all your servers together... 

Actually there wasn't an elevator pitch. I think it was a combination of wondering about how one might make deploys faster (and thus less painful), 
implementing [Hunter Haugen's dynamic git-branching for puppet](http://hunnur.com/blog/2010/10/dynamic-git-branch-puppet-environments/) and making the AMQ rig more reliable because we'd sold one of the dev teams on the idea of using it for inter-server comms.

What the stomp-git daemon does is subscribe to a topic, in our case named 'future.git.commits' and if it spots a commit message for a repository it cares about, issues a 'git fetch' command in order
to update its local copy of that repo. Thus when it comes time to deploy a site, the relevant tag or branch is already lurking in the git tree and the actual 'git checkout epic-bugfix-4.77' happens tolerably quickly.

This does involve a selection of scripts and daemons flying in close formation, so I shall attempt to describe the flow of events.

An assumption I made is that there's something like gitolite where most of your repositories live and it's been configured to emit post-commit messages.

If you're using GitHub, they have post-commit hooks that'll talk to all sorts of things, and their example Sinatra code is trivially hackable. I'll cover that in a later post.

A commit message looks a lot like this:

{% codeblock %}
[spog] refs/heads/master aa794ab6cc1a4a504fd908efc938c0f241557ad4
repo: spog
oldrev: dc01649eee870d1a9f051f046165890c4c5e197d
newrev: aa794ab6cc1a4a504fd908efc938c0f241557ad4
refname: refs/heads/master
 commit aa794ab6cc1a4a504fd908efc938c0f241557ad4
Author: John Hawkes-Reed <john.hawkes-reed@futurenet.com>
Date:   Wed Mar 14 11:55:15 2012 +0000

    bonkle

diff --git a/bonkle b/bonkle
index 1169e1c..48ee005 100644
--- a/bonkle
+++ b/bonkle
@@ -26,3 +26,4 @@ bonkle
 bonkle
 bonkle
 bonkle
+bonkle

{% endcodeblock %} 

... And if that looks an awful lot like the output from the example post-commit mail, that's because I cribbed it wholesale.

The only section that the stomp-git daemon is looking for is the 'repo: spog' line. The rest is syntactic sugar on the off-chance that someone wants to subscribe to that topic and (say) pass the output to an IRC channel.

Here's the YAML config for the stomp-git daemon:
{% codeblock %}
---
# Corp
#rserver: failover://(stomp://gituser:gitpass@amq1.domain.com:61613,stomp://gituser:gitpass@amq2.domain.com:61613)
# Co-lo
rserver: failover://(stomp://othergituser:othergitpass@amq3.otherdomain.net:61613,stomp://othergituser:othergitpass@amq4.otherdomain.net:61613)
#
# Topics:
listen-topic: future.git.commits
report-topic: future.events.stomp-git
#
# All repo modes badly explained:
# normal: - fetch $repo.
# trusting - fetch $repo, checkout origin/master to $target.
# dangerous - fetch $repo, checkout origin/master.
# puppetmaster - fetch $repo, checkout to $target/$branch. As above.
#
# Repos below here
#
repo-list:

#  example-repo:
#    user: rubyapps
#    repo: /data/repos/example
#    mode: normal
#
#  spog:
#    user: www-data
#    repo: /data/repos/octopress-site
#    target: /data/sites/www.my-octopress.co.uk
#    mode: trusting
#
#  puppet-hieradata:
#    user: puppet
#    repo: /etc/puppet/hieradata
#    mode: dangerous
#
#  puppet-environments:
#    user: puppet
#    repo: /etc/puppet/puppet-environments
#    target: /etc/puppet/environments
#    mode: puppetmaster

{% endcodeblock %}

... Where 'rserver' is a connection URL you can feed to the Ruby stomp library, listen- and report- topics should be moderately self-evident and the interesting bit is at the bottom - the repository names and locations on the target machine. So that turns into 'cd repodir && git fetch'. Sorted.

_Puppetmaster specials?_

The dynamic git-branch stuff above does its stuff with ssh pubkeys and a post-commit shellscript. This works fine for our one puppetmaster that's on the same VLAN as our gitolite box. However, our other puppetmaster is on the inside of the corporate firewall where inbound connections are _verboten._ Like don't even ask because a chorus of derision from the rest of the ops team can sometimes offend the sorts of people who become wedded to single ways of doing things. 

However, the AMQ rig is Allowed, so the stomp-git daemon has a puppetmaster mode. (Which would probably be better being split off into a different program, but I wrote it in a temper and and...)

In puppetmaster mode, the daemon will also checkout the update it has just fetched. You really don't want want this sort of behaviour under most circumstances.

Referencing the above config, 'puppet-environments' is the location of the cloned copy of your puppetmaster tree and 'environments' is the checked-out version that features in the linked description of dynamic branching even further above.
