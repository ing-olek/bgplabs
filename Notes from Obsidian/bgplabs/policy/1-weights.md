# Summary
[Select Preferred EBGP Peer with Weights « BGP Labs](https://bgplabs.net/policy/1-weights/) 
Select best path for outgoing traffic with "weight" attribute on one router with 2 ISPs.
## Lab customization
No. FRR image is used, it works in WSL.
## Actions & Discussion
Ask questions:
- "When does it make sense to use weight attribute in BGP?"
	- You have only one router, then weight works as good as LocalPref
	- You prefer to use only one ISP, while second ISP is backup. The second ISP may be too slow or too expensive.
- "What are drawbacks of using weight?
	- It is local for router, i.e. if you have two or more routers, you need to use LocalPref
	- It is not described in any RFC, it is supported by Cisco and some other vendors.

