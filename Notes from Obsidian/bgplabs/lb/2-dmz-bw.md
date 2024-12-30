# Summary
[EBGP Load Balancing with BGP Link Bandwidth « BGP Labs](https://bgplabs.net/lb/2-dmz-bw/) 
Use "link bandwidth" extended BGP community for weighted ECMP load balancing.
## Lab customization
In this lab we'll use Arista vEOS image for `rtr` router, because FRR image doesn't use weights based on BGP "lbw" extcommunities, when installing routes to IP RIB.
![[Lab topology - use EOS.jpg]]
## Actions & Discussion
In the previous lab we configured ECMP load-balancing in eBGP. In this lab we'll use "link bandwidth" extended BGP community for load-balancing across two links with unequal bandwidth. As the links have unequal bandwidth (2:1 ratio in our example) we'd like to use unequal cost load balancing.

### Extended communities
Ask questions:
- What is the difference between *standard* and *extended* BGP communities? What are they used for? See more: [BGP Zero to Hero Part 7, BGP Communities](https://learningnetwork.cisco.com/s/article/BGP-Zero-to-Hero-Part-7--BGP-Communities) 
	- *Standard* BGP communities are 32, e.g.: 16 bit ASN + 16 bit ASN
	- *Extended* BGP communities are 48 bits long, e.g.: 16 bit ASN + 32 bit IP address
	- *Large* BGP communities are 96 bits long, e.g.: 32 bits IP address * 3 times
- What are 4 types of BGP path attributes? See more: [RFC 4271 - A Border Gateway Protocol 4 (BGP-4)](https://datatracker.ietf.org/doc/html/rfc4271)
	- Well-known mandatory: NEXT_HOP, AS-PATH, ORIGIN - well-known attributes must be recognized by all BGP routers, mandatory attributes must exist in all BGP updates
	- Well-known discretionary: LOCAL_PREF, ATOMIC_AGGREGATE - discretionary attributes might be omitted in BGP updates
	- Optional transitive: COMMUNITY, EXTENDED_COMMUNITY, AGGREGATOR - optional attributes might be not recognized by BGP routers, transitive attributes, however, must be send further
	- Optional non-transitive: MED, ORIGINATOR_ID, CLUSTER_LIST - optional non-transitive attributes might be discarded, if not recognized by BGP routers
- What types of BGP *Extended* communities do you know? [RFC 4360 - BGP Extended Communities Attribute](https://datatracker.ietf.org/doc/html/rfc4360)
	- RT: route-targets - they are used in MPLS L3VPNs to import / export VPNv4/VPNv6 routes between VRFs
	- SoO: site-of-origin - they are used by PE routes to prevent routing loops when CE routers are multi-homed, see: [BGP Site of Origin (SoO) - Concepts & Configuration](https://learningnetwork.cisco.com/s/article/bgp-site-of-origin-soo-concepts-amp-configuration) 
	- LBW: link bandwidth - we'll see in this lab.
	- etc.
### "Link bandwidth" extended community
"Link bandwidth" extended community works this way:
- A router sets "link bandwidth" extended community for various routes based on bandwidth of external links 
- Another routers receive BGP updates with "link bandwidth" extended communities and then selects best path according to usual BGP algorithm. 
	- If there are many best ECMP routes (with equal metrics), they are installed to IP routing table with different weights, according to "link bandwidth" extended communities
	- If some of ECMP routes don't have the extended communities, router may act differently: discard those routes, set default weight for them or treat all routes as ECMP only, ignoring "link bandwidth" extended communities.

### Lab task
In this task we should reset MEDs on routes received from `x1` and `x2` to make both routes active in ECMP. And then applied "link bandwidth" extended BGP communities to set weights in 2:1 ratio.

At start we see two routes 10.1.3.0/24 with different MEDs.
![[rtr - At start - show ip bgp.jpg]]

Let's create route-maps to reset MED to 0 and add extended communities to the routes.
To add extended communities we will create `ip extcommunity-list` first.

After configuring extended communities "link bandwidth" we still don't see many ECMP routes, only one best route.
![[rtr - extcommunity & BGP config.jpg]]

To make ECMP work we have to enable `maximum-paths <n>` in address-family IPv4 BGP configuration.
![[rtr - enable maximum-paths.jpg]]

Now ECMP works, there are two routes to 10.1.3.0/24. Let's see BGP routes with "link-bandwidth" extended community. However, in IP RIB we still see two routes with equal metrics.
![[rtr - show ip route - no weights.jpg]]

To make extended BGP community "lbw" work on Arista platform we need to enable UCMP (unequal-cost multi-path). Then in IP RIB you can see weights assigned to two routes 10.1.3.0/24. The weights are assigned according to bandwidth ratio (2:1) configured in BGP "link bandwidth" extended community!
![[rtr - ucmp & show ip route + weights.jpg]]

This labs shows example of usage of "link bandwidth" extended community. Then you can do unequal-cost load-balancing in BGP (or more specifically, weighted ECMP!).

