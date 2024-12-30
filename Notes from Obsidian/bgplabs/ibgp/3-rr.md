# Summary
[Use BGP Route Reflectors « BGP Labs](https://bgplabs.net/ibgp/3-rr/)
Configure Route Reflectors in big iBGP topology.
## Lab customization
No. FRR image is used, it works in WSL.
## Actions & Discussion
In the previous lab we configured "full-mesh" iBGP sessions between PE and core routers in Service Provider network. 

With N routers in the network, number of iBGP sessions in "full mesh" topology increases very quickly - it is N * (N-1) / 2. Therefore, there is a couple of workarounds:
- Route Reflectors
- BGP confederations
## Route Reflectors
Route reflectors break iBGP "split horizon" rule: "iBGP does not advertise prefixes from one iBGP neighbor to another iBGP neighbor". 

It means, in typical scenario, there is one or more route reflectors forming "cluster" with a set of routers, called *route-reflector clients*. All clients in the cluster have iBGP sessions with route reflectors, therefore any two clients can exchange routes via route reflector. No direct iBGP sessions between clients needed.

Some facts about RRs:
- Route reflectors and clients might have iBGP sessions with other routers, which are not part of the cluster. 
- There maybe more than one RR cluster in the network. Route reflector can be a route-reflector client itself (in another cluster).
- Route reflectors may have iBGP session with each other, if there are more than one RR in cluster.
- RR clients are not aware if they have BGP session with route reflector or "normal" router, for clients it is usual iBGP session
- Route reflector doesn't change any attribute of iBGP routes, sending BGP updates "as is".
- Route reflector sends only one the best route. I.e. first route reflector selects best path to a given prefix from a few possibly available. And then sends it to BGP peers.
- Route reflectors may or may not forward traffic in data plane. It depends on network design, both options are viable.

### Route advertising rules of route reflectors
Route reflectors advertise routes based on the following rules:
- Route reflector advertise to its clients routes learnt from other clients, form iBGP peers or eBGP peers
- Route reflector advertise to its iBGP peers (non-clients) routes learnt from clients or eBGP peers, but not from iBGP peers (other non-clients)
- Route reflector advertise to its eBGP peers routes from clients or other iBGP peers (non-clients)

### New BGP attributes of Route reflectors. Protection from routing loops with RRs
Ask question:
- We said that route reflectors break "split horizon rule", but, then, how do they protect against routing loops?
	- Routing loop occurs when router receives its own advertised route and uses it because of better metric
	- To prevent from routing loops RR use two new BGP attributes: ORIGINATOR_ID and CLUSTER_LIST

BGP attribute ORIGINATOR_ID is set by route-reflectors and used by RR clients: if router sees its RouterID as ORIGINATOR_ID, the BGP route is discarded. When RR receives route from a client, it sets client's RouterID in ORIGINATOR_ID attribute.

BGP attribute CLUSTER_LIST is set by route-reflectors and used by other route-reflectors. When RR advertises route via iBGP it adds its RouterID to CLUSTER_LIST BGP attribute. When RR receives route via iBGP it checks CLUSTER_LIST attribute and discards route if it sees its RouterID in the list.

### Configure RR
To make a router route-reflector for a particular iBGP neighbor we use `neighbor <peer IP> route-reflector-client` command.
### Read more about RRs
- [BGP Route Reflector Details « ipSpace.net blog](https://blog.ipspace.net/2008/08/bgp-route-reflector-details/)

## Lab task
Our task in this lab is to re-configure "full mesh" iBGP sessions between leaf and spine routers to route reflectors. Two leaves will be RRs and four spines will be RR clients.

At start let's examine BGP neighbors on leaf and spine switches.
![[l1 - at start - iBGP full-mesh.jpg]]

We see there is iBGP "full mesh", i.e. every router has iBGP sessions with all other routers.![[s1- at start - iBGP full-mesh.jpg]]

We'll do these tasks:
- Delete iBGP sessions on leaves with other leaves
- Configure spines to be route reflectors for leaves
- Other configuration will remain (i.e. leaf-spine iBGP session on leaves side and iBGP session between spines)
>[!NOTE] We'll keep iBGP session between spines, because besides being route reflectors for clients they are "normal" iBGP routers and need to exchange routes.

![[l4 - deleted iBGP with leaves.jpg]]

Now let's configure spines as route reflectors. We just need to use `route-reflector-client` sub-command under `neighbor` configuration.
![[s2 - configure Route Reflectors 1.jpg]]

>[!NOTE] We configured BGP cluster-id parameter. We may set it to the same value or different ones, there are "pros" and "cons" of both options, see more on the link: [Mixed Feelings about BGP Route Reflector Cluster ID « ipSpace.net blog](https://blog.ipspace.net/2022/02/bgp-rr-cluster-myths/) 

Let's check if leaves see routes of each other. As you can see `l1` has routes to all leaves (192.168.41-44.0/24). If we check BGP route details, we can see new BGP attributes: ORIGINATOR_ID and CLUSTER_LIST - set by route reflectors.
![[l1 - show ip bgp - RR attributes.jpg]]

We can also see these attributes in BGP messages in traffic capture. When `s1` (IP 10.0.0.10) sends `l1` BGP update about 192.168.42.0/24 network (route from `l2`) it adds ORIGINATOR_ID and CLUSTER_LIST attributes.
![[s1 - tcpdump - RR attributes.jpg]]

When `l2` sent BGP update about its network 192.168.42.0/24 to `s1` there was no these attributes. It means route reflector clients are not aware if they have BGP session with RRs or "regular" iBGP session. 
>[!NOTE] We didn't change anything in BGP sessions between leaves and spines on leaves side!

![[s1 - tcpdump - RR attributes 2.jpg]] 

In the end we see that spines still have BGP sessions with all routers, but every leaf has only two BGP sessions with spines! And leaves have all routes.

Route reflectors is a great tool to make iBGP sessions scalable!
![[l2 - show ip bgp - in the end.jpg]]

## BGP confederations
Another way to reduce large number of iBGP sessions are so-called "BGP confederations".

In short, BGP confederations means that large AS N is divided to few sub-AS X, Y, Z, etc. Sub-ASes have eBGP sessions with each other, iBGP is used only inside sub-ASes, thus decreasing number size of iBGP "full-mesh" networks.

See more: [Examine Border Gateway Protocol Case Studies - Cisco](https://www.cisco.com/c/en/us/support/docs/ip/border-gateway-protocol-bgp/26634-bgp-toc.html#toc-hId-1434789637) 

Key facts about BGP confederations:
- External routers don't know about confederation, they establish BGP session with "public" AS N and see only AS N prepend in AS-Path;
- AS-Path inside confederation contains AS_CONFED_SEQ or AS_CONFED_SET attributes (normally AS-Path uses only AS_SEQ and AS_SET). 
	- These attributes allow to avoid routing loops between eBGP peers inside confederation.
	- However, AS_CONFED_SEQ and AS_CONFED_SET are not used in best path calculation!
- Routers in BGP confederation are configured with two additional parameters: **bgp confederation identifier N** and **bgp confederation peers X Y Z**
- eBGP session between confederation peer is treated equal as iBGP in best path selection process.

>[!CAUTION] When using BGP confederations, routers are running BGP process with BGP ASN = Sub-AS number. It means, to implement BGP confederation you have to change BGP ASN on your routers, which requires downtime.

Because of this fact, BGP confederations are rarely used in practice. Router reflectors are used much more often.

