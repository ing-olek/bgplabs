# Summary
[BGP Policy Templates « BGP Labs](https://bgplabs.net/session/7-policy/).
Configure peer-groups and use them to apply route-map to BGP neighbors.
## Lab customization
No. FRR image is used, it works in WSL. Both FRR and Arista vEOS support only `peer-group` commands (or I didn't find `templates` in configuration).
## Actions & Discussion
In the previous lab we used peer groups to configure iBGP sessions. What parameters did we set in peer groups? 
We set `remote-as` and `update-source` and these parameters were used to establish BGP session.

Peer groups can also be used to configure templates of routing policies (i.e. route-maps). In this lab we'll use peer groups to configure route-maps to filter routes on eBGP sessions.
### Lab task
In this lab our task is:
- Filter prefixes with long masks (>24) received from eBGP peers
- Filter prefixes with long masks (>24) when advertising to eBGP peers
- Tag routes received from eBGP peers with BGP community 65000:200 (meaning "IP transit")
- Tag local routes advertised to eBGP peers with BGP community 65000:300 (meaning "AS 65000 local routes")

At start let's check routing table on `rtr` router. You see /32 prefixes received from `x1` and `x2` and also advertised to them. We should filter /32 routes and keep only /24 ones.
![[rtr - show ip bgp - before.jpg]]

Let's create prefix-lists and route-maps first. We'll also set BGP communities in route-maps on "in" and "out".
![[rtr - configure route-filters.jpg]]

Now let's create peer-group and apply route-map to it.
>[!TIP] You need to `activate` BGP session in IPv4 address-family. This command can also be included in peer group configuration.

![[rtr - configure eBGP peer-group.jpg]]

Let's check results - we see that /32 prefixes were filtered both on "in" and "out".
![[rtr - show ip bgp - after.jpg]]

Let's also verify BGP communities on routes on `x1` or `x2` - "transit" routes are tagged with 65000:200 and local `rtr` routes are tagged with 65000:300. 
![[x1 - various BGP communities.jpg]]

And of course, another solution how to scale configuration of N x 100 BGP sessions is network automation! :-)

