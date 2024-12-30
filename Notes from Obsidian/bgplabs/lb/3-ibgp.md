# Summary
[IBGP Load Balancing with BGP Link Bandwidth « BGP Labs](https://bgplabs.net/lb/3-ibgp/) 
Send "link bandwidth" extended BGP community to other routers for weighted ECMP load balancing.
## Lab customization
In this lab we'll use Arista vEOS image for `rtr` router, because FRR image doesn't use weights based on BGP "lbw" extcommunities, when installing routes to IP RIB.
![[Lab topology - using vEOS.jpg]]
## Actions & Discussion
In the previous lab we configured UCMP load-balancing with "link bandwidth" extended communities, using them on one router.

In this lab we'll expand using "lbw" extended communities to many routers in the network.

### Lab task
Configure `we1` and `we2` routers to advertise routes with "lbw" extended communities. By default in the lab `we1` must advertise 20G and `we2` - 4G of bandwidth, but we may use different values.
If possible, let's configure bandwidth on interfaces and automatically convert it to "lbw" extended communities. However, not all platforms support it.
`core` router must receive routes from `we1` and `we2` with extended BGP communities.

At start let's check BGP RIB on `we1`, `we2` and `core` routers.

As you can see, ECMP is not configured yet on all internal routers. There is only one best route in BGP RIB.
![[we1 - BGP RIB - at start.jpg]]

`core` router also has only one best route.
![[core - BGP RIB - at start.jpg]]

Let's configure WAN edge routers to send "link-bandwidth" extended communities.

We configured `neighbor <ip> link-bandwidth auto` command for external BGP neighbors to automatically calculate "link-bandwidth" extended community values based on interface bandwidth.
But Arista vEOS doesn't support `bandwidth` command on interfaces, therefore we used default interface bandwidth values 1G instead of 20G on `we1` and 4G on `we2`.
![[we1 - link-bandwidth auto + send extcommunity.jpg]]

Now we see `we1` router tags its external routes learnt from `x1` with "link-bandwidth" extended community.
![[we1 - show ip bgp.jpg]]

And the same for `we2` router.
![[we2 - link-bandwidth auto + send extcommunity.jpg]]

![[we2 - show ip bgp.jpg]]

Now let's look at routes on `core` router. `core` router sees "lbw" extended communities, however it still has only one best route in its BGP RIB! Why? Because we have to enable ECMP and UCMP on `core` router.

>[!TIP] Note, that `core` router sees 10.1.3.0/24 route from `we1` (IP 10.0.0.2) in one route with bandwidth 250Mbps! While `we1` has two routes each with bandwidth 125Mbps. That's because we used command `neighbor <ip> send-community link-bandwidth aggregate` with **aggregate** flag.

![[core - show ip bgp - first.jpg]]

Let's configure ECMP and UCMP on `core` router.
![[core - configure UCMP + ECMP.jpg]]

And now `core` router has many ECMP routes o 10.1.3.0 and it installed it with appropriate weights to IP routing table!
![[core - show ip bgp - after + weights in RIB.jpg]]

