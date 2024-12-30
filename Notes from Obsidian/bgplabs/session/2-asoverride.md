# Summary
[Fix AS-Path in Environments Reusing BGP AS Numbers « BGP Labs](https://bgplabs.net/session/2-asoverride/)
Use `asoverride` BGP feature to replace AS number in AS-Path with another AS.

## Lab customization
No. FRR image is used, it works in WSL.
## Actions & Discussion
In the previous lab we had scenario when our company has many sites connected with MPLS L3VPN provider and the service provider told us to use the same AS number on each site.
The workaround was to use `allowas-in` BGP feature on our CE routers. Then we could accept routes from other sites, in spite of they have the same AS in AS-Path as ours.

Okay, we used that nerd knob `allowas-in` as workaround? But wouldn't it be nice, if we could have it fixed on provider side? Maybe our routers don't support `allowas-in` command or it makes config too complex for our admins.

In this lab we'll accomplish the same goal but using another BGP feature, but this time on Service Provider side.

At start we see `ce1` doesn't have routes from `ce2`.
![[ce1 - no route - before.jpg]]

While `pe1` has BGP routes from both `ce1` and `ce2`.
![[pe1 - BGP routes - at start.jpg]]

We'll use `neighbor as-override` BGP feature to replace neighbor AS number in AS-Path with providers ASN when sending BGP updates to that neighbor. See more: [Understanding BGP AS-Override Feature - Cisco Community](https://community.cisco.com/t5/networking-knowledge-base/understanding-bgp-as-override-feature/ta-p/3111967)

On `pe1` router configure `as-override` on `ce1` neighbor.
![[pe1 - AS-Override.jpg]]

And in a moment we see `ce2` routes on `ce1`!
![[ce1 - show ip bgp - after.jpg]]

And we have to configure the same on `pe2` on BGP session with `ce2`.


