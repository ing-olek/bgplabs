# Summary
[Configure a Single EBGP Session « BGP Labs](https://bgplabs.net/basic/1-session/) 
Configure a eBGP session.
## Lab customization
No. FRR image is used, it works in WSL.
## Actions & Discussion
Configure additionally:
- `bgp log-neighbor-changes`
- `neighbor description`
- `bgp router-id` - discuss that it can be any IP address, but good practice is to use "Loopback0" as router ID in routing protocols
- Try `netlab validate`
- Try `netlab collect`

Ask questions:
- "What do you configure when you have an eBGP session with ISP?"
	- Neighbor & remote-AS
	- Advertising your prefixes
	- Inbound filter: private networks, your networks, etc.
	- Outbound filter: AS-path filter (to not be transit)
	- Advertising your prefixes
	- Etc.