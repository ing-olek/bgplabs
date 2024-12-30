# Summary
[Filter Transit Routes « BGP Labs](https://bgplabs.net/policy/2-stop-transit/) 
Prevent AS from being transit. Advertise routes to ISPs with filtering - empty AS-path access-list.
## Lab customization
No. FRR image is used, it works in WSL.
## Actions & Discussion
Discuss topics:
- Why being transit AS is bad?
	- Because ISPs will forward traffic through your AS to other AS in Internet. It will overload your links + you'll pay ISPs for all that unnecessary traffic!

Note: on FRR command to configure AS-Path access-list is different from Cisco:
`bgp as-path access-list`

Ask questions:
- Check that **x2** router prefers path to **x1** via **rtr**. Why (despite of longer AS-Path 65000 65100)? 
	- Because of higher LocalPref = 200