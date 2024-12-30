# Summary
[BGP Local Preference in a Complex Routing Policy « BGP Labs](https://bgplabs.net/policy/a-locpref-route-map/)
Implement more complex routing policy preferring all routes via ISP1 and some other routes via ISP2.
## Lab customization
No. FRR image is used, it works in WSL.
## Actions & Discussion
In this lab we'll implement more complex routing policy in scenario when we have 2 ISPs and 2 edge routers.

We'd like to prefer all routes via ISP1, but also use second Internet link - only for routes originated in ISP2.

Ask question:
- "What BGP attribute should we use?"
	- We should use LocalPref, because we have 2 edge routers
- "What criteria should we use to match only routes originated by ISP2"
	- AS-Path access-list will be the most efficient way to match ISP2 routes.

Let's check BGP RIBs on `c1` and `c2` before starting the lab.
![[c1 - show BGP - before.jpg]]

![[c2 - show BGP - before.jpg]]

Now let's implement routing policy on `c2`.

>[!TIP] We should use "$" sign of end of line in AS-Path access-list to avoid matching transit routes going through AS65101!

![[c2 - BGP route-map.jpg]]

And now we can see that all routes are preferred from `c1` except local network 192.168.42.0/24 and 192.168.101.0/24 prefix originated by AS65101 (ISP2).
![[c1 - show BGP - after.jpg]]

![[c2 - show BGP - after.jpg]]