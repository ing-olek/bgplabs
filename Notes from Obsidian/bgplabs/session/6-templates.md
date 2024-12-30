# Summary
[BGP Session Templates « BGP Labs](https://bgplabs.net/session/6-templates/). 
Configure peer-groups and use them in BGP neighbor configuration.
## Lab customization
No. FRR image is used, it works in WSL. Both FRR and Arista vEOS support only `peer-group` commands (or I didn't find `templates` in configuration).
## Actions & Discussion
In the previous lab we configured route reflectors and you might have noticed in BGP configuration many commands the same for many BGP neighbors.

Network device vendors support BGP config templates to make configuration simpler.
Cisco supports:
- peer groups
- session templates
- policy templates
### BGP peer groups
Other platforms (like FRR, which we'll use in this lab) use peer groups.

Peer groups were introduced in Cisco to resolve problem with generating BGP updates in old IOS versions. In IOS 11.x and before when router needed to send prefix in BGP update to neighbors it generated individual BGP update for each neighbor. Even if set of prefixes and attributes was absolutely the same! This causes high memory & CPU consumption on routers. 

Then Cisco introduced peer groups - you can create a group of peers with the same routing policies (i.e. route-maps) and only one update was generated per peer-group.

Later on peer group functionality for BGP updates become built-in. Now you don't need to create `peer-group` to generate one update for many routers with the same routing policies. However, `peer-group` command is still available and allows to configure parameters common for many routers.

See more: [BGP peer group vs. BGP templates - Michael's personal Blog](https://www.mvankleij.nl/post/bgp-peer-groups-vs-templates/) 

### BGP templates
BGP templates accomplish that second part of peer-group functionality, i.e. allowing to define template of BGP session or routing policy and then assign it to all neighbors sharing the same settings.

In Cisco BGP templates are more flexible in configuration, because you can inherit one template (parent) from another (child).
### Lab tasks
In this lab we have 2 spines and 2 leaves, working as route reflectors and RR clients, accordingly.
We'll configure two peer groups on spines - one general "ibgp" session and another one - for "rr-clients".

Let's look at `s1` configuration at start. You can see the same commands are applied to all iBGP neighbors (`remote-as` and `update-source`) and also to `route-reflector-client` to all clients.
![[s1 - BGP config - before 2.jpg]]

Let's configure peer groups and put repeating config in them.
![[s2 - peer-group configured.jpg]]

After configuring peer-groups BGP config is much shorter. Imagine config with 100 BGP neighbors?
![[s2 - BGP config - after.jpg]]

