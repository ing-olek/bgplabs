# Summary
[BGP Route Server in an Internet Exchange Point « BGP Labs](https://bgplabs.net/session/5-routeserver/)
Configure BGP Route Server in IX (Internet eXchange).
## Lab customization
No. FRR image is used, it works in WSL.
## Actions & Discussion
In this lab we'll act as small IX (Internet eXchange) point, providing public peering for many ISPs. In data plane it means ISP routers are connected in one shared VLAN and in control plane we provide "Route server", i.e. router running BGP with all ISPs, thus allowing them to exchange routing information. 

Ask questions:
- What concept does Route server remind you?
	- It is like Route reflector. However, RRs are used in one AS with iBGP, while route server runs eBGP sessions with ISP routers.
- What is important in data plane to avoid route server be a bottleneck?
	- Data traffic shouldn't go through route server, its role is solely control plane. Thus, Route server MUST NOT change next-hop IP to itself (as eBGP does by default). However, if few routers share LAN segment and have eBGP sessions over it, BGP doesn't change next-hop to self when re-advertising routes from one peer connected to that LAN to another, see RFC 4271: [ietf.org/rfc/rfc4271.txt](https://www.ietf.org/rfc/rfc4271.txt)

Let's check BGP RIB on `isp1` router at start. You see, that `isp1` has eBGP session only with `rs` Route server, but it has routes from all ISP routers. Note, that ISP routes have correct next-hops (pointing to `isp2` and `isp3` IPs, not `rs`!). However, Route server added its AS 65000 to AS-Path. And we'd like to avoid it, having "clear" eBGP routes from other ISPs.
![[isp1 - show ip bgp - before.jpg]]

This can be done with `neighbor route-server-client` BGP feature. It removes router's own AS from AS-Path when advertising routes to eBGP peers.
![[rs - configure route-server-client.jpg]]

Now let's check routes on `isp1`. Ooops! All route disappeared, what happened?
![[isp1 - show ip bgp - no routes.jpg]]

Let's run **debug bgp updates**. Oh, we see that BGP updates from Route server were discarded, because of error: "Incorrect first AS"! 
![[isp1 - debug ip bgp - incorrect first AS.jpg]]

Indeed, now `rs` sends BGP update without prepending them with its AS 65000 and by default BGP expects to see neighbor's AS first in AS-Path in eBGP sessions.

Let's fix this on ISP routers! Yes, there is another "nerd knob" to change default BGP behavior.

After clearing BGP session softly to enforce "route-refresh" we see that `isp1` now has ISP routes without Route server's AS 65000.
![[isp1 - configure no enforce-first-as & show ip bgp - after.jpg]]

The same configuration should be done on `isp2` and `isp3` (but in BGP labs it's already done).



[IP Routing: BGP Configuration Guide, Cisco IOS XE Release 3S - Configuring a BGP Route Server [Cisco IOS XE 3S] - Cisco](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/iproute_bgp/configuration/xe-3s/irg-xe-3s-book/irg-route-server.html)