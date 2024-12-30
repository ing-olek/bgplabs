# Summary
https://bgplabs.net/basic/5-redistribute/
Redistribute OSPF routes to BGP (and vice versa) in MPLS network (with L3VPN service from ISP).
## Lab customization
No. FRR image is used, it works in WSL.
## Actions & Discussion
In the previous lab (`3-originate`) we advertised own /24 prefix in BGP by using `network` command. 
In this lab we'll try another way how to add routes to BGP - route redistribution.

In simple scenario configure mutual redistribution between OSPF and BGP with `redistribute ospf` and `redistribute bgp` commands.

Discuss topics:
- BGP doesn't advertise prefixes automatically. Nor it discovers neighbors automatically. That's the difference with OSPF and other IGPs
- Check advertised routes with `show ip bgp neighbor advertised-routes` and note that the router advertises not only its prefixes, but also ISPs' i.e. it's transit router (which is not desired)

Ask questions:
- "What are advantages and disadvantages of route redistribution?"
	- Pros: saves time when many routes should be injected to BGP; can be effectively used when routes can't be aggregated
	- Cons: potentially a lot of routes can leak between routing protocols, causing troubles
- "When redistribution to BGP is often used?"
	- Scenario: typically redistribute to iBGP in enterprise network
	- Or on Customer Edge (CE) routers redistributing from OSPF / EIGRP / Static & connected to BGP when using MPLS L3VPN service
- "What is a typical problem with redistribution between routing protocols? For example, from BGP to OSPF?"
	- Information about metric and other attributes of the route is lost. Metrics of different routing protocols are different.
	- Workaround: set default metric when redistributing new route from another routing protocol.
- "What are best practices of route redistribution?"
	- Tag routes and filter by tags
	- Filter routes by prefixes (in route-maps)
	- Use redistribution in one way only and advertise default route in another direction
## Additional tasks
### Advertise default route to OSPF
On `c1` we will use only one-way redistribution: OSPF to BGP. In reverse direction (BGP to OSPF) we will advertise default route instead.

Task: On `c1` advertise default route to OSPF with metric 100. Check if default route is in RIB on `s1`.

Ask questions:
- "How to handle issue, that default route doesn't exist in RIB on `c1`?"
	- Use `default-information originate always` command

![[OSPF - originate default.jpg]]

![[S1 - OSPF routes.jpg]]

### Redistribute BGP routes to OSPF with filtering
On `c2` we will filter routes which are advertised from BGP to OSPF.

In `c2` BGP RIB we see not only 172.16.x.x routes, but also 10.x.x.x/32. Let's keep only 172.16.x.x.
![[C2 - BGP routes.jpg]]

![[S2 - OSPF routes - before.jpg]]

Task: On `c2` redistribute BGP routes to OSPF, keeping only 172.16.x.x - 172.31.x.x routes, set OSPF metric 50 to them. Check routes is in RIB on `s2`.

Ask questions:
- "How to check if redistributed route is advertised by OSPF?"
	- Use `show ip ospf database external` command
![[C2 - Redistribute to OSPF - filter.jpg]]

Now we see that 10.0.0.3/32 route was filtered during redistribution.
![[S2 - OSPF routes - after.jpg]]
