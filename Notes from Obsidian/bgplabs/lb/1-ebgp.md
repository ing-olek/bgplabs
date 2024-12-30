# Summary
[Load Balancing across External BGP Paths « BGP Labs](https://bgplabs.net/lb/1-ebgp/) 
Configure ECMP in eBGP.
## Lab customization
In this lab we use FRR image, but I have to change default BGP config, because I need to disable `bgp bestpath as-path multipath-relax` feature and set `maximum-paths 1` in address-family IPv4.
![[lab customization - disable multipath and bgp as-path relax.jpg]]
## Actions & Discussion
In this lab we'll configure ECMP (Equal-cost multi-path) on a router with many eBGP sessions.

Ask questions:
- What is the difference between load balancing and load sharing? Or are they synonyms?
	- Load balancing means sending traffic via various path to the same destination (destination prefix or IP address of host)
	- Load sharing means sending whole traffic via multiple paths (whole traffic is included, destination is not necessarily the same)
- When two eBGP routes are considered "equal"? What metrics should they have the same?
	- Weight
	- LocalPref
	- AS-Path (not only AS-Path length, but exact match of AS-Path!)
	- Origin
	- MED

### Lab task
In this lab we will configure `rtr` router to have eBGP multi-paths to prefixes 10.1.3.0/24 (via 2 paths) and 10.7.5.0/24 (via 3 paths).

Let's look at BGP RIP and IP RIB of `rtr` at start. As you can see `rtr` receives many routes for 10.1.3.0/24 and 10.7.5.0/24 prefixes, but only one route is installed in the routing table.
![[rtr - At start - show ip bgp + show ip route.jpg]]

How to fix this? We need to enable eBGP multi-path (i.e. ECMP).
![[rtr - maximum-paths configured.jpg]]

Let's check BGP RIB and IP RIB now. We see that 10.1.3.0/24 and 10.7.5.0/24 prefixes now have two routes in RIB each.

Prefix 10.1.3.0/24 has 2 equal routes, both originated in the same AS 65100 with the same weight, local preference, AS-Path and MED.
![[rtr - show ip bgp - 2 routes now.jpg]]

However, 10.7.5.0/24 prefix has only 2 of 3 routes. Why? Because 3rd route has different AS-Path! Even despite it has the same AS-Path length!

Let's fix this by enabling appropriate BGP feature: `bgp bestpath as-path multipath-relax`.
![[rtr - bgp bestpath as-path multipath-relax.jpg]]

Now we see that all 3 routes for 10.7.5.0/24 prefix are installed in the routing table!
![[rtr - after - show ip bgp + show ip route.jpg]]