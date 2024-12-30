# Summary
[Attach BGP Communities to Outgoing BGP Updates « BGP Labs](https://bgplabs.net/policy/8-community-attach/)
Use BGP communities provided by ISP to its customers.
## Lab customization
No. FRR image is used, it works in WSL.
## Actions & Discussion
In this lab we want to force ISP-2 (`x2` router) to forward traffic to 192.168.42.0/24 via ISP-1 (`x1` router).

First, let's try to use AS-Path prepends. It won't work, because `x2` has higher LocalPreference to routes received from `rtr` than `x1`. This is like ISP normally have higher LocalPref to their customers than to peers and uplinks
![[x2 - AS-Path prepend doesn't work.png]]

Hmm, what can we do? Fortunately for us, ISP-2 published BGP communities for its customers, which can take the following actions:
- Change LocalPref to N
- Add X prepends when advertising to peers Z
- Don't advertise
- Blackhole

Not all ISPs support BGP communities for customer traffic engineering. You can see BGP communities on ISP's website or RIPE or BGP information aggregators, like [Hurricane Electric BGP Toolkit](https://bgp.he.net/). 

For example, see BGP communities of Lumen (former Level3, AS3356) one of the biggest ISPs in the world: [Webupdates — RIPE Network Coordination Centre](https://apps.db.ripe.net/db-web-ui/lookup?source=ripe-nonauth&key=AS3356&type=aut-num).
![[RIPE - Level3 communities.png]]

In our scenario ISP2 supports the following BGP communities:
![[Pasted image 20241127202436.png]]

Ask questions:
- "Which community should we use?"
	- 65304:100, to lower LocalPref < 100, which is set to transit routes from other ISPs, like ISP1 (`x1` router)
- "When should we use community 65304:102?" 
	- When we have 2 links to ISP2 and want second one to be backup (it will have lower LocalPref 190 on ISP2 side)

Okay, let's set BGP community on `rtr` when advertising our routes to `x2`.
![[BGP community set.png]]

But for some reason it doesn't work. We don't see AS-Path prepends on `x2` (we removed them from our route-map), but LocalPref is still 200. Why? What's wrong? 
![[BGP community not working.png]]

Ask question:
- "What's missing in BGP community configuration?"
	- On some platform (for example, Cisco) BGP communities by default are stripped from BGP message. We should enable sending BGP communities on peer basis.

![[Enable send-community.png]]

After we enabled sending BGP communities to `x2` it works! Now `x2` has lower LocalPref on 192.168.42.0/24 route.
![[LocalPref lowered.png]]

Ask questions:
- "How does it work?"
	- ISP router has route-map applied on "in" from customers. That route-map matches BGP communities and takes appropriate actions.
- "What does first rule of route-map customer on `x2` do?"
	- It has double "deny": in route-map and in AS-path access-list. Route-map "deny" blocks route matching AS-Path access-list. 
	- The first rule of AS-Path access-list matches routes which have only one AS in AS-Path, maybe repeated one or more times (\1 in regex mean "previous match"). As these routes are "denied" in AS-Path access-list, they are actually "allowed" in the route-map! These rule represents routes from customers, because only one AS in AS-Path is allowed.
	- The second rule in AS-Path access-list blocks all other routes.
![[x2 - BGP config.png]]

