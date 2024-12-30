# Summary
[Filter Advertised Prefixes « BGP Labs](https://bgplabs.net/policy/3-prefix/) 
Double-protect you from advertising bogus prefixes. Make sure you advertise only your aggregated prefixes.
## Lab customization
No. FRR image is used, it works in WSL.
## Actions & Discussion
Discuss topics:
- It is best practice always configure inbound and outbound route-maps to double-protect your AS from wrong routes
- Scenario: 
	- You have AS-Path access-list on "out", but you still advertise some wrong routes (/32 from loopbacks) to ISPs
	- If AS-Path access-list is not pre-configured, configure it again.
