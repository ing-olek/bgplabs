# Summary
[Reuse a BGP AS Number Across Multiple Sites « BGP Labs](https://bgplabs.net/session/1-allowas_in/)
Use "allowas-in" BGP feature.
## Lab customization
No. FRR image is used, it works in WSL.
## Actions & Discussion
In this lab we have scenario, when our company has many sites connected via MPLS network and Service Provider provides us only one AS number for all sites. Anyway, private AS numbers are from 64512 to 65535, i.e. there are only 1024 ASNs.

Ask question:
- How to deal with the issue, that BGP updates on CE router are rejected, because they have the same AS number in AS-Path?
	- Most popular vendors support `neighbor allowas-in` feature which disables that check. 
>[!CAUTION] Be cautious, because this disables BGP loop detection mechanism!

At start let's check routes on `ce1` router - we don't see 10.0.0.2/32 route from `ce2`. Why?
![[ce1 - show ip bgp - no route - before.jpg]]

Let's run debug. 
>[!CAUTION] Use **debug ip bgp** command with ACL or prefix list filter to limit debugging scope. Otherwise, there is risk to overload CPU & memory of your router.

We see the problem - BGP update is denied, because it contains router's own AS number 65000.
![[ce1 - debug ip bgp - BGP update denied.jpg]]

Let's fix it with `neighbor allowas-in` BGP feature and check results. Here we go! Now `ce1` sees 10.0.0.2/32 route from `ce2`!
![[ce1 - show ip bgp - route exists - after.jpg]]

The same command should be configured on `ce2` to accept routes from `ce1`.


![[pe1 - show ip bgp.jpg]]
