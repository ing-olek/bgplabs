# Summary
[Running EBGP Across a Firewall « BGP Labs](https://bgplabs.net/basic/e-ebgp-multihop/)
Configure eBGP multihop session.
## Lab customization
No. FRR and Linux images are used, they work in WSL.
## Actions & Discussion
In this scenario we have internal and external routers separated by firewall, which doesn't run BGP, but uses static routes instead.

We will configure eBGP-multihop session between routers. When traffic goes through firewall, it will be forwarded based on static routes on the firewall.

Ask questions:
- "What are potential problems with eBGP multihop session? Note: usually eBGP session is configured with directly connected peer!"
	- TTL = 1 by default in eBGP messages. Therefore packets in eBGP multihop session will be dropped by intermediate L3 devices
	- Peer routers need to have a route (either static or dynamic) to reach neighbor!


At start there is no eBGP session between `int` and `ext` routers.

We configured eBGP session and activated it in address-family IPv4, but the session is in "Active" state.
![[int - BGP neighbor configured.jpg]]

Let's add static routes to /32 neighbor IP address on both sides. But BGP session is still "Active".
![[int - static route.jpg]]

In output of `show ip bgp neighbor` command we see that eBGP hop can be only 1 hop away.
![[int - BGP neighbor.jpg]]

Let's configure "eBGP-multihop" feature (on both sides) - and we see the BGP session came up!
![[int - eBGP-multihop.jpg]]

To pass `netlab validate` check we also need to advertise default route 0.0.0.0/0 from `ext` router to `int`.