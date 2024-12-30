# Summary
[Advertise IPv4 Prefixes to BGP Neighbors « BGP Labs](https://bgplabs.net/basic/3-originate/) 
Advertise own /24 network to ISPs.
## Lab customization
No. FRR image is used, it works in WSL.
## Actions & Discussion
Configure prefix advertisement in BGP with `network` command in `ipv4 address-family` and static route to `Null0`.

Discuss topics:
- BGP doesn't advertise prefixes automatically. Nor it discovers neighbors automatically. That's the difference with OSPF and other IGPs
- Check advertised routes with `show ip bgp neighbor advertised-routes` and note that the router advertises not only its prefixes, but also ISPs' i.e. it's transit router (which is not desired)

Ask questions:
- "How can we inject routes to BGP RIB?"
	- `network` command
	- `aggregate` command
	- `default information originate` command
	- redistribute routes static, connected or from IGP
- "What are pros and cons of redistributing vs manual configuration?"
	- Redistribution:
		- Pros: saves time when many routes should be injected to BGP; can be effectively used when routes can't be aggregated
		- Cons: potentially a lot of routes can leak between routing protocols, causing troubles
		- Scenario: typically redistribute to iBGP in enterprise network
	- Manual:
		- Pros: better control on routes added to BGP
		- Cons: doesn't scale well, if you have many routes to be injected to BGP(unless they are aggregated or you use automation)
		- Scenario: typically manually configure `network` when advertising public IP network to ISPs
- "How we can add a new route in IP RIB (to be used by `network` command)?
	- Create Loopback interface and assign the network to it
	- Create static route to `Null0` interface

