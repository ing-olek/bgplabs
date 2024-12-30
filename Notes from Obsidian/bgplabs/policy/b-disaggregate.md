# Summary
[Use Disaggregated Prefixes to Select the Primary Link « BGP Labs](https://bgplabs.net/policy/b-disaggregate/)
Advertise more-specific prefixes via ISP-1 and aggregate prefix via ISP-2.
## Lab customization
No. FRR image is used, it works in WSL.
## Actions & Discussion
In this lab we want to force ISP-2 (`x2` router) to forward traffic to 172.16.4.0/22 via ISP-1 (`x1` router).
- First, let's try to use AS-Path prepends
	- It won't work, because `x2` has higher LocalPreference to routes received from `rtr` than `x1`.
	- This is like ISP normally have higher LocalPref to their customers than to peers and uplinks
- And in our case ISP-2 doesn't publish BGP communities which can be used by customers or peers to manipulate LocalPref
- As last resort we can use route disaggregation. 
	- It will work only if we have bigger prefix than minimum /24
	- It will work 100%, because routers always prefer more-specific routes, no matter what metric they have

Notes:
- Create prefix-lists matching aggregated prefix 172.16.4.0/22 and two or more more-specific prefixes
- Don't forget to inject more-specific prefixes to IP RIB and BGP RIB! :-)

Ask questions:
- "What is disadvantage of using this approach (route disaggregate) globally?"
	- Routing tables in Internet are getting bigger (they are already > 900 000 routes)
