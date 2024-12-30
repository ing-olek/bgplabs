# Summary
[Build a Transit Network with IBGP « BGP Labs](https://bgplabs.net/ibgp/2-transit/)
Configure iBGP "Full mesh" topology.
## Lab customization
No. FRR image is used, it works in WSL.
## Actions & Discussion
In one of the previous labs we configured iBGP session between two edge routers, it was used to share eBGP routes learnt from ISPs.

Now we'll configure iBGP in topology with more routers. Our company's network is getting bigger and we'll have more complex network topology.

### Task
- Configure iBGP session between `pe1` and `pe2`
- Configure iBGP sessions on `core` with `pe1` and `pe2` (if necessary)
- Ping `ext` router from `pe2` router

To start let's ping `ext` router (IP 172.16.42.42) from `pe2` router (IP 192.168.43.2).
>[!TIP] To set source interface or IP you should use Linux **ping -I** command from **bash** not from **vtysh** mode.

Ping doesn't work. Why? Let's check routing table - `pe2` doesn't have route to 172.16.42.0/24.
![[pe2 - at start - ping failed - no route 1.jpg]]

Then let's configure iBGP session between `pe1` and `pe2`.
>[!TIP] Remember to configure iBGP sessions to peer's loopback IP address and use own loopback as source! Also remember to activate the BGP session in address-family IPv4. And configure `next-hop-self` option.

![[pe2 - iBGP session configured.jpg]]

Now let's check routing table on `pe2` and try ping again. Ping to 172.16.42.42 still doesn't work. Now **traceroute** shows it stops on `core` router.
![[pe2 - no ping, traceroute.jpg]]

Let's login to `core` router and check its routing table. We see `core` router doesn't have route to external 172.16.42.0/24 and it's quite natural, as it doesn't have BGP session with `pe1`! 
By the way, `core` router doesn't have route to source network 192.168.43.0/24 either!
![[core - no routes.jpg]]

Ask questions:
- "How can we fix routing on `core` router?"
	- Redistribute BGP routes to OSPF - that causes high risks of large network downtime one day
	- Configure iBGP "full-mesh" sessions between all internal routers. We'll use this approach.
	- Configure "BGP-free core" with MPLS or Segment routing or other overlay technology. This is out of scope of this lab.

Let's configure iBGP "full-mesh" sessions on routers in AS65000.
![[core - BGP configured.jpg]]

Now let's try ping from `pe2` to `ext`. It works! :-)
![[pe2 - ping works.jpg]]

### Questions to discuss about iBGP
Ask questions:
- "Why iBGP "full-mesh" sessions must be configured between all internal routers?"
	- Because iBGP uses "split horizon" rule: routes learnt from an iBGP peer are not advertised to another iBGP peer
- "What are BGP features to eliminate large number of iBGP sessions?"
	- Route reflectors
	- BGP confederations
- To understand why iBGP "full-mesh" is needed, can you think about network topology which doesn't have iBGP "full-mesh" resulting in a routing loop?
	- Hint: typically routing loops are created when logical topology (BGP) doesn't match physical topology
