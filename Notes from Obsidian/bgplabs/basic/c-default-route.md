# Summary
[Advertise Default Route in BGP « BGP Labs](https://bgplabs.net/basic/c-default-route/)
Advertise default route to BGP.
## Lab customization
No. FRR image is used, it works in WSL.
## Actions & Discussion
In this lab `x1` represents customer router and `r1` is ISP router (usually they are vice versa). 
The goal of the lab is to advertise only default route to customers (because Internet "Full view" with > 900 000 prefixes is too much for small customer router).

Let's check what routes `x1` receives at start. It receives 192.168.42.0/24 prefix, which represents all Internet routes.
![[Customer x1 - before.png]]

Let's advertise default route to `x1` from `r1`. In some OS you may have default in routing table to advertise it in BGP, or you may have to use command like `default-originate always`. 
In BGP we configure `default-originate` on per-neighbor basis (in contrast with OSPF).
![[r1 - advertise default.png]]

Now we see customer router receives default. But it still receives all other routes (represented by 192.168.42.0/24).
![[Customer x1 - receive default and other routes.png]]

Let's filter other routes!
![[r1 - filter all except default.png]]

Now customer receives only default route. The task is done!
![[Customer x1 - receive default only.png]]