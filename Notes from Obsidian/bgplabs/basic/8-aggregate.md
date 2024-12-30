# Summary
[BGP Route Aggregation « BGP Labs](https://bgplabs.net/basic/8-aggregate/)
Redistribute OSPF routes to BGP with dynamic aggregation (`aggregate` command).
## Lab customization
No. FRR image is used, it works in WSL.
## Actions & Discussion
Ask questions:
- "When will you use dynamic aggregation with `aggregate` command in BGP, instead of "static" aggregation with `network` command?"
	- When it may happen that aggregated routes disappear from the network, e.g. when you aggregate prefixes of small site which maybe down. For example, because of electricity problems.

At start let's check OSPF and BGP neighbors at `ce1` router and its routing table. We see, there is no BGP routes yet.
![[ce1 - routes - before.png]]

After redistribution from OSPF to BGP we see many prefixes in BGP RIB on `ce1` and all of them are advertised to `ce2`. 
![[ce2 - redistribute OSPF to BGP.png]]

![[ce2 - all BGP prefixes.png]]

And we want to advertise only 10.42.42.0/24 aggregated prefix. Let's use BGP `aggregate-address` command for that.
![[ce1 - aggregate.png]]

Now we see on `ce2` aggregated prefix 10.42.42.0/24 but also see all other BGP prefixes.
![[ce2 - all BGP prefixes + aggregated.png]]

We would like to get rid of all other prefixes, we can do it with `summary-only` option of `aggregate-address` command.

We still see subnets of 10.42.42.0/24 in BGP RIB of `ce1`, but now they are labeled "suppressed" and not advertised to `ce2`.
![[ce1 - suppressed routes.png]]

However, we still see "garbage" route 10.0.0.3/32, because it's not part of 10.42.42.0/24 network. 
![[ce2 - aggregate + bogus.png]]

Let's filter on `ce1` all prefixes which are not part of 10.42.42.0/24 network, to not advertise to `ce2`.
![[ce1 - route-map - out filter.png]]

![[ce2 - aggregate only.png]]

Let's verify that `aggregate-address` command advertises routes only when there is at least one subnet of aggregated network in RIB.

Let's disable `ce1` interface with OSPF neighbor.
![[ce1 - Disable OSPF.jpg]]

Now we don't see any BGP routes on `ce2`.
![[ce2 - BGP - route not exists.jpg]]
