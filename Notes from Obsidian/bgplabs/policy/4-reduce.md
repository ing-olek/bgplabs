# Summary
[Minimize the Size of Your BGP Table « BGP Labs](https://bgplabs.net/policy/4-reduce/) 
Accept only default route from ISP1 and backup default + local routes from ISP2.
## Lab customization
No. FRR image is used, it works in WSL.
## Actions & Discussion
In this lab we simulate situation when we have small edge router and two ISPs advertise "Full views" of BGP routes ( > 900 000) to it.

For example, Cisco 9300 switches support only 32 000 of routes, see:
[Cisco Catalyst 9300 Series Switches Data Sheet - Cisco](https://www.cisco.com/c/en/us/products/collateral/switches/catalyst-9300-series-switches/nb-06-cat9300-ser-data-sheet-cte-en.html) - they have limited TCAM resources (special memory, where FIB is stored).

Cisco routers in contrast with switches usually don't have hardware limit on TCAM resources, but their capabilities depend on available RAM and CPU.

In our case two "Full views" is overkill, it will be enough to have a default route (from ISP1) and backup default (from ISP2). However, it would be shame to not use ISP2 link, therefore we can accept prefixes of ISP2 itself over the second link.


At start let's check what routes `rtr` receives from `x1` and `x2`.
![[Show BGP at start.jpg]]

Here is one of possible solutions.
![[BGP - prefix list and route-maps.jpg]]

And let's verify it.
![[Show BGP in the end.jpg]]

Note: in this lab command `netlab validate` returns error, but it is false - it can't correctly check second default route from `x2`.
![[Netlab validate - fail - error.jpg]]

