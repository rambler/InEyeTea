+++
date = "2012-05-25T10:23:32-06:00"
title = "UUID Uniqueness"

+++

A little while ago, someone asked me about collisions in [UUIDs](https://datatracker.ietf.org/doc/rfc4122/):

> Any idea what kind of guarantees of uniqueness these tools argue/provide? For example if I use them to generate `$ 10^{15} $` IDs over 10 years, what is the probability that I will get a collision? Do they have a reference to the algorithm they use?

Well, in terms of financial guarantees, none.  If you use this free software and it doesn't do what you expect, you get back what you paid for it.

However, the proposed standard does "guarantee" uniqueness; this is clearly part of the design.[^1] An implementation which follows this specification has a component which is derived from the MAC address of the generating host for what is called "the spatially unique node identifier".  Other components include the time with 100 ns resolution, and a counter, which is incremented each time a UUID is generated and set to a random value when the counter becomes unavailable.

[^1]: From [the RFC]((https://datatracker.ietf.org/doc/rfc4122/): "A UUID is 128 bits long, and can guarantee uniqueness across space and time."

My envelope here shows that to generate `$ 10^{15} $` UUIDs over ten years (on a single machine), you'd have to generate another one every 315 ns or so (I've got `$ 315.36*10^{15} $` ns in ten years).  Since this is close to the clock resolution, you might be relying on the counter mechanism, perhaps quite a bit at times depending on the characteristics of the UUID request arrival process.  You'd have to have a pretty big bit bucket - you'd be generating nearly 50 MB/s for ten years, over 14 petabytes total, at 128 bits per identifier.

If the machine crashes, the 14 bit counter gets set to a random value. Suppose the clock also gets set backwards (overlapping a period where UUIDs were generated).  If the implementation followed the spec, the chance of the counter repeating itself at the exact time the clock repeats itself is pretty small (i.e. effectively 0).  One could calculate a meaningless, ridiculously small positive probability by making assumptions about the rate at which clock resets occur, and the rate at which requests occur, using standard approximations of the [general birthday problem](http://mathworld.wolfram.com/BirthdayProblem.html).  Or not.

Frankly, the most likely source of failure would be an implementation interaction error such as: the source for the random number that the counter uses isn't very random (i.e. 2 bits instead of 14) or the clock really has a resolution of 1 second, but the implementation relies on it being much finer.  The implementation is simple and designed to be fast; it's not so likely that the implementation would be wrong as that assumptions made in the implementation are invalidated as the underlying system changes.

There's a political / privacy issue also with the use of standard UUIDs; the MAC address of the generating host is essentially included verbatim. So, if you drop a UUID in the course of committing a crime, this may be used in the investigation or case [against you](http://news.com.com/2100-1023_3-223857.html).[^2] That's the privacy part; the political part is the [public outcry](http://www.efc.ca/pages/media/nytimes.07mar99.html) regarding the privacy part.  The specification addresses these concerns by proposing alternatives to the MAC address based UUID.

[^2]: I'm not sure if Microsoft's implementation is/was exactly the same as the UUID standard, but the issues are similar.

Thinking ahead, the UUID clock field will rollover around 3400 A.D.