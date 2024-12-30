# Summary
[Passive BGP Sessions « BGP Labs](https://bgplabs.net/session/8-passive/)
Configure "passive" BGP session on one of the sides.
## Lab customization
No. FRR image is used, it works in WSL.
## Actions & Discussion
In this lab we'll configure BGP feature - "passive" BGP session. If BGP router is configured as "passive", it means it doesn't try to establish BGP session, but will respond if other side initiates the session. At least one of two sides must be "active", otherwise two "passive" router can't start BGP session.

Ask question:
- When does it make sense to configure BGP sessions on router as "passive"?
	- Router has a lot of BGP sessions: route reflector with many iBGP clients, route server with many eBGP clients, etc.
	- Router has BGP sessions with servers, which may be turned off from time to time.
	- Hub routers in large VPN "hub-and-spoke" deployments.
	- etc.

At start let's check BGP sessions on `hub` router.
![[hub - show ip bgp - before.jpg]]

Now let's configure all BGP sessions on `hub` router as "passive". As you can see all BGP sessions became "Established" except with `s3`, because `s3` is also configured to be "passive".
![[hub - show ip bgp - after.jpg]]

