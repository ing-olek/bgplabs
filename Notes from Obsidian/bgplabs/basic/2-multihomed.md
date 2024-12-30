# Summary
[Configure Multiple EBGP Sessions « BGP Labs](https://bgplabs.net/basic/2-multihomed/) 
Configure two eBGP sessions (multi-homing).
## Lab customization
No. FRR image is used, it works in WSL.
## Actions & Discussion
Topics to discuss:
- Check received routes with `show ip bgp neighbor routes` and note that the router receives prefixes from ISPs
- Check advertised routes with `show ip bgp neighbor advertised-routes` and note that the router doesn't advertise anything to ISPs, i.e. traffic goes in one direction now. Services won't work

Ask questions:
- "What do you configure when you have two eBGP sessions with ISPs (multi-homing)?"
	- Neighbor & remote-AS
	- Advertising your prefixes
	- Inbound filter: private networks, your networks, etc.
	- Outbound filter: AS-path filter (to not be transit), prefix-list (only your prefixes)
	- Advertising your prefixes
	- BGP metrics (LocalPref, AS-Path, MED, etc) for load balancing inbound and outbound traffic
	- Etc.