# Summary
[Use BGP Communities in Routing Policies « BGP Labs](https://bgplabs.net/policy/9-community-use/) 
Implement BGP policy based on communities - on ISP side.
## Lab customization
No. FRR image is used, it works in WSL.
## Actions & Discussion
In one of the previous labs we used BGP communities provided by ISP to force ISP's router to set lower Local Preference on ISP side, thus making that route backup. 

Now we'll implement that policy on ISP side.

Let's look at BGP neighbors and BGP RIB on `isp` router at start.
![[isp - show BGP - at start.jpg]]

We can achieve this goal with using `bgp community-list standard`  command and `route-map` where we match BGP community 65000:50.
![[isp - BGP communit-list and route-map.jpg]]

After that `isp` router sees best route to 172.17.207.0/24 via `x` router (another ISP), not `c` router (customer).
![[isp - show BGP - after.jpg]]

Congratulations!

Note: to pass check of `network validate` command you have to configure another route-map and attach it to `x` neighbor on in, setting Local Preference to 200.

