---
layout: post
title: "Java-based diversionary tactics"
date: 2012-04-01 16:54
comments: true
categories: [Alanis Morrisette, pro-celebrity swearing, XML is the work of Satan]
author: JHR
---

The other week, I stood in front of a room filled with my notional peers and allowed as how 'ActiveMQ setup was a bit of a pig, but once you'd got it working it was pretty simple.'

[FX: Pause for hollow laughter]

In retrospect, that statement was obviously going to come back and bite me just as soon as it had located its special big false teeth. Thus it came to pass on Wednesday night that two of the brokers had a meltdown, which in turn broke some experimental Nagios-over-Stomp code and so kept the poor sod who was on call in a state of near panic as _everything_ reportedly failed.

Oops.

So when I pitch up on Thursday AM, I am welcomed by the whole team merrily grousing about 'your effing brokers' (Some software objects are like children and pets. When they're well behaved and/or looking cute for visitors, they're _our_ children or pets. When they've just left a deposit behind the telly, they're _your_ children or pets). Indeed it was a right mess. One broker had used all the memory allocated and as if for spite had run out of filehandles too. It was all a bit odd. Some poking about revealed [AMQ-jira ticket] which more-or-less explains itself and the fact that it looked an awful lot like an experimental client wasn't responding to messages as quickly as one might hope.

<!-- more -->

There's a page or two on the Apache-AMQ site about the problem with 'slow consumers'. The upshot seems to be that it is not yet a solved problem. 

Anyway. This is going to be a hand-waving description of the AMQ config we use. There may or may not be better ways of doing it, but this one (mostly) works for us.

The most recent (and likely final) topology is a mesh (actually it looks like a wonky pentagram, but that's neither here nor there), in that each broker (there are seven) is directly connected to the other six. The theory is that since the individual brokers aren't guaranteed to stay up (more of that anon) each section of the network should have a pair of brokers available and all the clients should instantiate 'reliable' comms by connecting to a failover pair. In Ruby (as evidenced by MCollective itself) and PHP, this works rather well.

A tiresome thing to note is that the XML entities/objects/annoyances that make up the broker specification _have to be in alphabetical order_ which, um, riiight. 

{% codeblock %}  
  <broker xmlns="http://activemq.apache.org/schema/core" brokerName="Your-broker-1" useJmx="true" persistent="false">
{% endcodeblock %}
  

So far so good. Your various brokers should have different names.  The wrinkle here is that we turn off persistent messaging. The theory is that in the case of a broken consumer on a busy topic or queue - say you're moving metrics or logs - the broker won't get bogged down storing them to disk. As we have discovered, in that eventuality it'll try its best before getting wedged. I think we're in Linux OOM-killer territory here. There isn't a good answer so much as a least-worst one.

{% codeblock %}
  <destinationPolicy>
    <policyMap>
      <policyEntries>
        <policyEntry topic=">" advisoryForSlowConsumers="true" />
   			<policyEntry topic=">" advisoryWhenFull="true" />
   			<policyEntry queue="*.reply.>" gcInactiveDestinations="true" inactiveTimoutBeforeGC="300000" />
        <policyEntry topic="metrics.>">
          <pendingMessageLimitStrategy>
            <constantPendingMessageLimitStrategy limit="50000"/>
          </pendingMessageLimitStrategy>
        </policyEntry>
       	<policyEntry topic="future.events.>">
        	<subscriptionRecoveryPolicy>
        		<fixedCountSubscriptionRecoveryPolicy maximumSize="3"/>
       		</subscriptionRecoveryPolicy>
     		</policyEntry>
     	</policyEntries>
   	</PolicyMap>
  </destinationPolicy>
{% endcodeblock %}

Policy entries set per-topic/queue behaviours. For instance we want one of the ActiveMQ.Advisory topics that the brokers use for internal management to grass up slow consumers and another one to emit messages when any other topic fills up. The JSON they emit can be jolly handy for monitoring what's going on with a broker. The next-to-last entry sets a maximum message backlog for the metrics tree, and the final one allows anything consuming the future.events tree to see the last three messages in each subtopic. A kind of 'previously on future.events' if you will.

If you have more than one broker, you'll need to set up some network connectors. The below examples fold in the changes for MCollective 1.3 which requires separate connectors for queues and topics. You'll also note that the URIs are specified as SSL connections. Setting this up requires some faffery with Java Keytool and some XML that specifies where to find the keystores. This will turn up later because of the alphabetical ordering thing.

{% codeblock %}
  <networkConnectors>
    <networkConnector 
      name="broker1-broker2-topics" 
      uri="static:(ssl://an.ip.add.ress:6166)" 
      userName="YourUsername" 
      password="YourPW" 
      duplex="true" 
      networkTTL="1" 
      decreaseNetworkConsumerPriority="true">
      <excludedDestinations>
        <queue physicalName=">" />
      </excludedDestinations>
    </networkConnector>
    <networkConnector 
      name="broker1-broker2-queues" 
      uri="static:(ssl://an.ip.add.ress:6166)"
      userName="YourUsername" 
      password="YourPW" 
      duplex="true" 
      networkTTL="1" 
      decreaseNetworkConsumerPriority="true">
      <excludedDestinations>
        <topic physicalName=">" />
      </excludedDestinations>     
    </networkConnector>
  </networkConnectors>
{% endcodeblock %} 

{% codeblock %}
  <plugins>
    <authorizationPlugin>
      <map>
        <authorizationMap>
          <AuthorizationEntries>
            <authorizationEntry queue=">" write="admins" read="admins" admin="admins" />
            <authorizationEntry topic=">" write="admins" read="admins" admin="admins" />
            <authorizationEntry queue="mcollective.>" write="admins" read="admins" admin="admins" />
            <authorizationEntry topic="mcollective.>" write="admins" read="admins" admin="admins" />
          </authorizationEntries>
        </authorizationMap>
      </Map>
    </authorizationPlugin>

    <simpleAuthenticationPlugin>
      <users>
        <authenticationUser username="YourUsername" password="YourPW" groups="admins,everyone"/>
        <authenticationUser username="Joe-Sysadmin" password="JoesPW" groups="admins,everyone"/>
      </users>
    </SimpleAuthenticationPlugin>
  </plugins>
{% endcodeblock %}

Topics, queues and users. Allow admin users to read, write (and thus create, which is somewhat crucial) topics and queues. Also set up some of those admin users. Since the brokers don't create the topics or queues until something actually starts writing to one, one could envisage a situation where the only entries in the config file are those giving admin rights. Further, since the Java code is capable of doing its user-management via LDAP, one ought to be able to dispense with the hard-coded user entries. Since one has to restart the brokers each time the config files are changed, this would likely be a Good Thing. As it is, there are a pair of brokers in each VLAN and we take care to specify that clients should expect to do failover between them.

{% codeblock %}
  <sslContext>
    <sslContext keyStore="file:${activemq.base}/conf/broker.ks" keyStorePassword="TopSecret" trustStore="file:${activemq.base}/conf/client.ts" trustStorePassword="SecretTop"/>
  </SslContext>
{% endcodeblock %}

SSL. What I have taken to doing is generating a certificate for each broker and storing it in broker.ks. Then I export the certificates and generate a client.ts file containing all the 'other' certificates. There's probably a better way of doing it.

{%codeblock %}
  <transportConnectors>
    <transportConnector name="openwire" uri="ssl://0.0.0.0:6166?socketBufferSize=131072?ioBufferSize=16384"/>
    <transportConnector name="stomp" uri="stomp+nio://0.0.0.0:61613?socketBufferSize=131072?ioBufferSize=16384?transport.closeAsync=false"/>
  </transportConnectors>
{% endcodeblock %}

You will need some transports. Stomp for the MCollective rig and our Ruby/PHP/Perl clients, Openwire for the Java-native messaging used by the inter-broker links.

{% codeblock %}
  </broker>
{% endcodeblock %}

_Fin._

