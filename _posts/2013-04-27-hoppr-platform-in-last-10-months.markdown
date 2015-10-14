---
layout: post
title: "Hoppr Platform in Last 10 Months"
date: 2013-04-27 20:45
comments: true
description: "hoppr architecture"
keywords: "hoppr,hoppr architecture, hoppr checkins , ddd , cqrs, Hoppr,Y2CF Digital Media,"
categories:
---

with 7.6 m users hoppr platform is now handling around 600k checkins and 3m+ events per day.

underline is a heavily distributed architecture spread across three data centers and aws vpc.
it support zero downtime deployment and minimalistic service availability even in case of wan link failure.

##Tech Stack : ##

* polyglot , 80% code base in ruby and 20 % in java , node and php
* mongo
* redis
* mysql (percona)
* elasticsearch
* torquebox and hornetq
* chef
* nagious
* apache and haproxy
* centos
* openvz

<!--more-->

##A big rewrite##

it all start with a big rewrite of the the existing platform , which
was a big fat jsp code base, repeated for 6 telcos , code was not unit tested and
not even close to mvc 1.


initial version of new platform is developed in 2 months with a core domain
layer and application layer for vendor and access medium specific code.
the idea was to develop a core domain layer serve as api and other as application layer for device/vendor specific code, now we have around 20 similar micro services (some RESTish and some json over http),
each service is highly cohesive to device , vendor or a context.
apart from acting as anti corruption layers these micro services can also scale out independently.


##vertical to horizontal##

for first few months we tried to scale up with torquebox in big box with jvm optimized for huge pages, but
for zero downtime deployment and HA, we moved to scale out with openvz
containers with apache/ mod_cluster(http) and haproxy(jms) as load balancers.

we automated box setup to one click deployment using chef, and now we
have around 50 nodes handling all load.

##More About Architecture##

ussd checkin in our system requires response in less then
200ms , so system is modeled in events and base event checkin is
processed in less then 50ms, a checkin further emits another max 5 events for
various post processing actions , each event is processed by one or all sites.


site is cluster of backend services with own db and cache deployed in different datacenter,
and can independently process basic checkin and generally optimized for specific access
mediums.

sites are connected by a vpn tunnel over lease line.
site can play one or multiple role and we control it by giving it
capability and capability is controlled by chef.

**distributed architecture over wan is not a big deal ,but problems starts if it is not a shared nothing** , and multiplies with type and intensity of sharing.

for some internal requirements we have to go with wan distribution and
hence to sync lots of data for mobile portability and
points/skipper/leaderboard calculations.

*multi site replication*

for muti site replication we are using application layer sync and
created a custom sync framework with fan out and repeat on failure with exponential backoff features,
data consistency is achieved by design by contract on sync endpoints.

sync framework's core is built with hornetq and netty connectors and it is working really great even with remote listeners.

*replicating single source of truth*

points and leadernoard is single source of truth and it is calculated by one site,
we also need to access this data syncronpusly and it is not possible
over vpn because of encryption and multiple connections overhead.
 
we tried various approaches to sync this, and finally followed cqrs type approach and now have different
request paths for read and writes. points are calculated in one site and synced to remote redis slave and read operation is always from redis slave.

redis sync is almost realtime and very fast even for master to slave
of slave over 2 hop vpn links separated datacenters.

*tuning centos*

system defaults not always works and cpu and memory sometimes are not
indicators of bottleneck. we tuned various kernel
parameters for network and connection related stuff.

more on this in following post


##Lesson Learned##

* dont trust your api users , they can stress your system in no time,  use timeouts and circuit breakers
* redis sync is very fast and it sync in millis from master to slave of
  slave even over 2 vpn link separated datacenters
* visualize every things , logstaligia is great for apache logs visualization and quick indicator of load
* adopt configuration management as soon as possible
