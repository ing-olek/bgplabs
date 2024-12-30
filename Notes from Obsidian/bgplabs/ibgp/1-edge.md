# Summary
[Establish an IBGP Session « BGP Labs](https://bgplabs.net/ibgp/1-edge/)
Use 2 edge routers and iBGP session between them.
## Lab customization
No. FRR image is used, it works in WSL.
## Actions & Discussion
In the previous labs we configured eBGP session between our router and ISPs, then applied basic route filtering mechanisms on inbound and outbound and tune BGP sessions (with BFD and MD5 / AO authentication).

Now it's time for the next step. Our organization wants to deploy redundant network topology, therefore we need second edge router with eBGP to ISP. 

And of course, we would like that two edge routers exchange routes with each other and have consistent routing policy. We can do that with redistribution from eBGP to OSPF (or another IGP), but it's not a good practice, because BGP "Full view" consists of > 900 000 prefixes (in 2024) and it will be too many LSAs to be flooded in OSPF, it would cause 100% CPU utilization on routers and neighbor flapping.
Therefore our choice is iBGP.

### iBGP features and differences with eBGP
Ask questions:
- "What are differences between iBGP and eBGP?"
	- iBGP - BGP session is established between two routers in the same AS, eBGP - two different AS
	- iBGP doesn't change any attribute of BGP route: neither AS-Path, nor next-hop, etc.
	- eBGP has AD = 20 and iBGP has AD = 200 (on Cisco routers)
	- iBGP sessions are typically configured between loopbacks of routers, while eBGP has TTL = 1 (i.e. only with directly connected routers)
### "Split Horizon" in iBGP
As iBGP doesn't have AS-Path protection against routing loops, split horizon rule is applied to BGP updates: router doesn't advertised routes via iBGP, if the routes were learnt by iBGP. 
That implies that routers in iBGP usually have "full mesh" of BGP sessions, to be sure that every router exchange routes with every other router.
### iBGP doesn't change any BGP attributes
The key difference of iBGP is that it doesn't change any BGP attribute when sending route to another router, neither AS-Path, nor next-hop, etc.
>[!TIP] You can memorize this fact, if you remember that back in the days BGP was used only for routing with ISP. And imagine you have two edge routers each having eBGP session with ISPs. You want them to select best external path consistently, i.e. they need to exchange BGP routes, like if every edge router had both external routes and compared them. Therefore iBGP is needed to send BGP route from one router to another, without any changes! This way both edge routers have exactly the same routes and can make the same decision in selection BGP best path.

Okay, let's start with the lab!

First, we configure iBGP session on both `r1` and `r2`. 
![[Enable iBGP - not activated.jpg]]

However, we don't see the BGP session in `show ip bgp su` output! Why? Because that command shows BGP sessions in IPv4 address family, and while on Cisco IPv4 address-family is enabled by default, on other platforms you might have to enable it with `neighbor <peer ip> activate` command in `address-family ipv4`.
![[Enable iBGP - still Active.jpg]]

However, the BGP session is still in "Active" state, not established. What's the problem?
In debug we see that router is trying to establish BGP session not from Loopback interface, but from physical Ethernet interface. That doesn't match with config on peer router.
![[Debug BGP - no loopback IP.jpg]]

Fix it with `neighbor <peer ip> update-source <interface>` command. 
>[!TIP] Best practice is to configure iBGP sessions from loopback interfaces. That guarantees, iBGP will remain up if any physical Ethernet interface goes down.

![[iBGP - configure update-source.jpg]]

Now we have iBGP session up, let's check BGP RIB. A strange thing - on `r2` we see best route to 192.168.100.0/24 is local, not via `r1`, despite of local route having longer AS-Path! Why?

If we look closer, we see there is no asterisk * near 192.168.100.0/24 route learned from `r1`, it means that route is considered as *invalid*.  
The problem is the route is advertised from `r1` with next-hop 10.1.0.2 pointing to external ISP router, and that next-hop is *inaccessible* from `r2`.
Remember, iBGP doesn't change any BGP attribute? :-) 
![[iBGP - route invalid.jpg]]

There are two solutions:
- One solution is to advertise external directly connected routes to internal OSPF;
- Another solution is to change next-hop of the route to `r1` router with `next-hop-self`. This is best practice for iBGP
![[iBGP - next-hop-self.jpg]]

Now we see 192.168.100.0/24 on `r2` has best route through `r1` with next-hop 10.0.0.1!
![[iBGP - next-hop-self 2.jpg]]