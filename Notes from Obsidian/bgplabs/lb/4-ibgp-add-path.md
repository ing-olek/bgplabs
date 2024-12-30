# Summary
[IBGP Load Balancing with BGP Additional Paths « BGP Labs](https://bgplabs.net/lb/4-ibgp-add-path/)
Use "Additional paths" BGP feature on Route reflectors.
## Lab customization
In this lab we'll use Arista vEOS image for `rtr` router, because FRR image doesn't use weights based on BGP "lbw" extcommunities, when installing routes to IP RIB.
![[Lab topology - using vEOS 1.jpg]]
## Actions & Discussion
In the previous lab we configured UCMP load-balancing when many routers exchanged iBGP routes with "link bandwidth" extended communities.

### Lab task
We should configure routers the way that `ac1` router has multiple routes to 192.168.42.0/24 and was able to do ECMP or UCMP load-balancing.

Note:  in this case `ac1` is route-reflector client of `rr`. 

At start let's check BGP tables at `rr` and `ac1`.
![[rr - show ip bgp - at start 1.jpg]]

You can see that `rr` has two routes 192.168.42.0/24 tagged with "link bandwidth" extended communities, but `ac1` has only one best route (with next-hop IP 10.0.0.1).
![[ac1 - show ip bgp - at start.jpg]]

Even if we configure ECMP & UCMP on `rr`, `ac1` still receives only one best route via BGP. 
![[rr - configure ECMP & UCMP.jpg]]

Even worse, all route reflector clients receive the same BGP route! Therefore load balancing doesn't work.
![[ac1 - show ip bgp - intermediate.jpg]]

To overcome this problem we'll use "BGP Additional path" feature on route reflectors.

First, let's configure ECMP & UCMP on `ac1`.
![[ac1 - configure ECMP & UCMP.jpg]]

Now let's configure "Additional path" BGP feature on route reflector `rr` with `neighbor <ip> additional-paths send any` command.
![[rr - Additional-paths capability.jpg]]

> [!IMPORTANT] "Additional paths" is BGP capability, therefore it is required to reset BGP session to negotiate it. Take it into account and plan maintenance window when deploy "Additional paths" feature in production! 

On `ac1` we see BGP session re-established and there are 2 routes to 192.168.42.0/24.
![[ac1 - BGP re-established & ECMP.jpg]]

In "BGP neighbors" output we can see BGP capabilities negotiated.
![[rr - BGP capabilities.jpg]]

![[ac1 - BGP capabilities.jpg]]

And now ECMP & UCMP work fine on `ac1`.
>[!TIP] Note, that BGP routes on `ac1` have additional attribute "Rx path id" - ID making multiple paths unique in BGP process.

![[ac1 - show ip bgp - after.jpg]]

And here is packet capture of BGP Update message containing "link bandwidth" extended community and ID "Rx path ID" added by "Additional path" BGP feature!
![[bgp - capture.jpg]]
