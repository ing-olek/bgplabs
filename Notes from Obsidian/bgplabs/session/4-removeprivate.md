# Summary
[Remove Private BGP AS Numbers from the AS Path « BGP Labs](https://bgplabs.net/session/4-removeprivate/)
Remove private AS numbers from AS-Path when advertising BGP routes to neighbors.
## Lab customization
No. FRR image is used, it works in WSL.
## Actions & Discussion
In this scenario we will act as ISP with customer with private AS number. We have eBGP session with customer which doesn't have its public ASN, but uses private AS. It's fine, but when advertising customer routes to other ISPs in Internet we must remove private AS from AS-Path.

At start let's see what our ISP router `rtr` advertises to another ISP's router `x2`. As you can see `x2` sees customer's private AS 65000 in AS-Path.

>[!TIP] AS 64500 is not private AS, by the way. Private AS range is from AS 64512 to AS 65535.

![[x2 - show ip bgp - before 1.jpg]]

Now let's configure `rtr` to remove private AS 65000 when advertising routes to `x2`.
![[rtr - remove-private-as.jpg]]

And now let's verify that `x2` sees 192.168.42.0/24 customer's network without AS 65000.
![[x2 - show ip bgp - after 1.jpg]]
