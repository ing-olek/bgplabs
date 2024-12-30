# Summary
[Dynamic BGP Peers « BGP Labs](https://bgplabs.net/session/9-dynamic/)
Configure "dynamic" BGP sessions on one of the sides.
## Lab customization
No. FRR image is used, it works in WSL.
## Actions & Discussion
In the previous lab we configured "passive" BGP sessions, which can be useful on routers with many sessions. 

In this lab we'll configure "dynamic" BGP sessions, when a new BGP session is dynamically created based on *peer-groups* and some whitelist of allowed neighbor's IPs.

Ask question:
- When does it make sense to configure BGP sessions on router as "passive"?
	- Router has a lot of BGP sessions: route reflector with many iBGP clients, route server with many eBGP clients, etc.
	- Router has BGP sessions with servers, which may be turned off from time to time.
	- Hub routers in large VPN "hub-and-spoke" deployments.
	- etc.

>[!WARNING] Be careful when using "dynamic" BGP session, because potentially open list of source IPs is a security breach. Use TCP authentication or TTL security to limit number of hosts who can establish BGP session with your router. Another good approach is to use automation to reduce administrative burden.

### Lab task
In this lab we'll configure two peer-groups: "internal" for iBGP neighbors (core routers) and "external" for eBGP neighbors. Then with two source IP ranges we'll configure "dynamic" BGP sessions.

At start let's make sure there are no BGP sessions on `hub` router.
![[hub - show ip bgp - before 1.jpg]]

Let's configure iBGP peer-group first.
>[!TIP] Note, after configuration of peer-group we didn't add members to it manually, but with command `bgp listen range <net/mask> peer-group <pg-name>` configured dynamic BGP peers!

Once peer-group was configured and `bgp listen` command applied to the peer-group, iBGP sessions were established dynamically.
![[hub - iBGP peer-group & dynamic peers.jpg]]

The same way we configure eBGP peer-groups and set `hub` router to dynamically listen to BGP neighbors in the peer-group.
![[hub - eBGP peer-group & dynamic peers.jpg]]

If we disable BGP session on one of the routers (e.g. `c1`), `hub` router doesn't show any information about it, like the session didn't exist.
![[c1 - BGP shutdown.jpg]]

![[hub - dynamic peer shutdown.jpg]]

That's it!