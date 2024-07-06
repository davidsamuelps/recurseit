---
title: "IPv4 to IPv6: from 32 to 128 bits"
date: "2016-08-07"
categories: 
  - "ccna"
tags: 
  - "128"
  - "32"
  - "address"
  - "adoption"
  - "bit"
  - "bits"
  - "colon"
  - "compress"
  - "compresses"
  - "exhaustion"
  - "format"
  - "header"
  - "hextet"
  - "ipsec"
  - "ipv4"
  - "ipv6"
  - "nat"
  - "networking"
  - "networks"
  - "new"
  - "octec"
  - "packet"
  - "private"
  - "public"
  - "reachable"
  - "rules"
  - "security"
  - "transition"
coverImage: "recurseit3.jpg"
---

Here we are again, people! This time, i decided to write a post about IPv6, being this protocol part of our present and future _(even past, because it is being used since years ago)_. As most of you must have IPv4 _(because IPv6 adoption is going SO slow)_, maybe IPv6 could be an alien protocol for you. Some people say its difficult, so, i want to explain a little bit here to make it clear. I do not think its difficult, i think it is different :) If you are used to IPv4, which is likely, then it can be shocking at the beginning, but,we can make cool examples here, right? :D

# **The facts**

As yo may know, IPv4 addresses are exhausted years ago, it looked inexhaustible 20 years ago, but now, its over! Why? well, now we have smartphones, tablets, laptops, and several types of devices, all of them needing internet connection. The internet changed the way we live, learn, transport (Thank God, Google Maps!), work, and even flirt! (you were not waiting that, do you? xD). So, its everywhere AND we are using it a lot. This exhaustion was foreseen by experts many years ago, that's why IPv6 was developed. Of course, it was in its early stages and not ready, but a temporal solution for this exhaustion was needed as fast as possible. That's the moment when NAT made its appearance.

With NAT we could deplete IPv4 exhaustion's speed and make a little longer our agony :) In no time we were able to share one or several IPv4 public addresses with hundreds/thousand of users in the same private network _(yikes!)._ Imagine the amount of people in China and India, that they need 4-5 layers of NAT! D:

So, enough history and funny facts. What about the differences? Are we going to find the same type of addresses in IPv6? Its true that broadcast doesn't exist there? how so? what about IPv4 and IPv6 in parallel? did i turn of the oven before writing this? :D

These questions and many others are going to be answered here! Same hour, same channel, don't miss it! :D

IPv6 and IPv4 are very different, not just the format, also the way we write them, hierarchy and so on. We will see here :)

# **The bits**

We were used to 32 bits and dotted decimal notation. In IPv4 world, 32 bits were enough for 4.3 billion of addresses _(sounds like a lot, but NO)_ and it seemed endless in that moment. Guess what? IPv6 went a way far from that, and address format is 128 bits! So, the magic number of public addresses is something like **340 trillion trillion trillion** and i am not stuttering! :D

IPv4 address format is singular, easy and short. 32 bits, separated in 4 groups of 8 bits _(called **[Octets](https://en.wikipedia.org/wiki/Octet_%28computing%29)**)_ each by a dot. That's why we say dotted decimal _(pretty obvious, huh?)._ Every group can have a number from 0 to 255. A common example that EVERYONE have seen in some moment of their life is something like the following:

> **192.168.1.0 /24**

I'm pretty sure i rang a bell in your head, because this is the private IP range configured by home routers. Depending of the vendor, third octet can have a different values, but logic is the same ;)

In the case of IPv6 we have 128 bits, divided in 8 groups of 16 bits each _(called **[Hextet](https://en.wikipedia.org/wiki/Hextet)**)_ separated by a colon. Every Hextet is written in hexadecimal and values can be between 0x0000 and 0xFFFF. This is an example of an iPv6 address:

> 2001:0000:0000:0000:DB8:800:200C:417A

Yeah, it doesn't look pretty and i don't think i would be able to memorize several of them. The cool part is that we can simplify it, following some rules. This is the first one:

> _The leading zeroes in any 16-bit segment do not have to be written; if any 16-bit segment has fewer than four hexadecimal digits, it is assumed that the missing digits are leading zeroes._

So, following that, we can write a single zero (0) in those hextets and simplify our address this way:

> 2001:0:0:0:DB8:800:200C:417A

Remember, this just can be done with leading zeroes! Else, we can enter into an ambiguous ground.

What about this address?

> FF01:0000:0000:0000:0000:0000:0000:0001

We can write it this way:

> FF01:0:0:0:0:0:0:1

BUT, then we reduce it even more! Making use of another rule:

> _Any single, contiguous string of one or more 16-bit segments consisting of all zeroes can be represented with a double colon._

In simpler words, if we see more than one hextet filled with zeroes, we can simplify them with a double colon (:)  So, our address will be now something like this:

> FF01::1

YES! See that? A couple of rules are handy! :D

But, read again! This can only be done ONCE! Why? Because we can indicate an ambiguous address. Take this example:

> 2001:0000:0000:0DB8:AC10:FE01:0000:0000

If you apply the double colon rule here twice, then you would write something wrong like this:

> 2001::DB8:AC10:FE01::

Why is this wrong? Because we don't know the length of the consecutive zeroes strings. So, it could be indicating several addresses like:

> - 2001:0000:0DB8:AC10:FE01:0000:0000:0000
> - 2001:0000:0000:0000:0DB8:AC10:FE01:0000

When the correct one is:

> 2001:0000:0000:0DB8:AC10:FE01:0000:0000

See the difference? So, we can use both rules together is we have several goups of hextets filled with zeroes and not contiguous. We can write the address in these ways:

> - 2001::DB8:AC10:FE01:0:0
> - 2001:0:0:DB8:AC10:FE01::

This way of simplifying IPv6 addresses by the application of these two rules mentioned earlier its called **_Compressed Format._**

Another interesting fact:

In IPv4 we could identify the network and host portion easily, using bitcount and subnet masks. In IPv6 its always done by bitcount. How do we do that? Easy peasy! Our address, followed by a forward slash and the number of bits of the prefix:

> 3ffe:1944:100:a::bc:2500:d0b/64

Also, you can set the host addresses to zero, as we did in IPv4 and write something like this:

> 3ffe:1944:100:a::/64

# Some advantages over IPv4

- Simplified configuration: literally "plug and play", as soon as you plug you copper cable to the PC, it creates by itself a local address (i will explain that later in detail) automatically. It uses your MAC address as part of the local address. _(cool, huh?)_
- Better end-to-end connectivity: We know that NAT was a huge help for IPv4, but for IPv6 we don't need it to communicate with other IPv6 hosts! You will get a public and reachable address thanks to the huge space that IPv6 has. No more NAT, baby! :D
- "Embedded Security": I put this between quotation marks. Why? Based on the [security](https://tools.ietf.org/html/rfc6434#section-11) section in  [RFC 6434](https://tools.ietf.org/html/rfc6434)

> Previously, IPv6 mandated implementation of IPsec and recommended the key management approach of IKE.  This document updates that recommendation by making support of the IPsec Architecture \[RFC4301\] a SHOULD for all IPv6 nodes.  Note that the IPsec Architecture requires (e.g., Section 4.5 of RFC 4301) the implementation of both manual and automatic key management.  Currently, the default automated key management protocol to implement is IKEv2 \[RFC5996\].
> 
> This document recognizes that there exists a range of device types and environments where approaches to security other than IPsec can be justified.  For example, special-purpose devices may support only a very limited number or type of applications, and an application- pecific security approach may be sufficient for limited management or configuration capabilities.  Alternatively, some devices may run on extremely constrained hardware (e.g., sensors) where the full IPsec Architecture is not justified

So, in few words, yeah, IPSec implementation is possible, and a SHOULD, but its not a MUST. A lot of people say that IPv6 is more secure because of IPSec, but, its not mandatory. It's secure, **IF** it is **ENABLED**. So, do not believe blindly in those comments.

Also, i would like to share with you the meaning of the words SHOULD, SHOULD NOT, MUST and MUST NOT in the RFCs.  [RFC 2119](https://tools.ietf.org/html/rfc2119) describes how those words must be interpreted.

Based on RFC 2119:

> ## SHOULD
> 
>  This word, or the adjective "RECOMMENDED", mean that there may 
>   exist valid reasons in particular circumstances to ignore a
>   particular item, but the full implications must be understood and
>   carefully weighed before choosing a different course.

Also, there is a RFC about security implications. [RFC 7123](https://tools.ietf.org/html/rfc7123) states:

> In general, networks should enforce on native IPv6 traffic the same
>    security policies currently enforced on IPv4 traffic.

So, even if its claimed that IPv6 has native security support _(which means your OS should support it)_, same policies should be enabled as its done in IPv4 networks (firewalls, filtering, etc).

- Better QoS support/More efficient IP packet: The IPv6 packet has a "flow label" field that gives better identification when its treated for QoS. Routers can use this field to assign the packet to an specific flow. Also, there is no need for checksum at IP Layer, so, it does not need to be calculated at every hop. This is now handled by upper layers protocols.
- Opportunity for new services: Now that NAT is gone, new services that were not possible easily because of the NAT layer, can be implemented. Everything based on end-to-end connectivity.

 

Phew! This was a lot of information, right? And there is still, too much out to discuss. So, here a new saga is born! IPv6 Saga! :D

In next posts we will talk about address types, uses, methods of address assignation and some transition mechanisms between IPv4 and IPv6. This is just getting started! :)

Thanks for reading!
