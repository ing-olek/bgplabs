# Summary
[Resolve BGP Wedgies « BGP Labs](https://bgplabs.net/policy/e-wedgies/)
Use BGP communities and AS-Path prepends to resolve issue with BGP "wedgies".
## Lab customization
No. FRR image is used, it works in WSL.
## Actions & Discussion
Some time ago we did labs when we managed Internet traffic with BGP communities and AS-Path prepends. In this lab we'll look at scenario of non-optimal routing (so-called "BGP wedgies") and will try to fix it.

### Lab scenario
In this lab we manage router `c1` in AS65000 which has 2 ISPs: ISP1 with router `p1` (AS 65207) and ISP2 with router `p2` (AS65304). Each of those ISPs has upstream provider, routers `u1` and `u2`, connected with each other.

In the lab scenario we'd like to receive inbound traffic via ISP1 (router `p1`). However, ISP2 (router `p2`) prefers directly connected link to customer, no matter how many AS-Path prepends we used when advertising our network 192.168.42.0/24. 
If you check `p2` router you see it has higher LocalPref = 200 to customer routes.

Let's add AS-Path prepends to route-map "out" on `c1`.
![[c1 - AS-Path prepends outbound.jpg]]

Then `p2` sees longer AS-Path via `c1` than via `u2`, but still prefers directly connected link because of higher local preference = 200.
![[p2 - show ip bgp.jpg]]

Fortunately, ISP2 implemented routing policy allowing its customer to tag their routes with BGP community 65304:100 and lower local preference to 50. We use that BGP community and tag network 192.168.42.0/24 with it. And we'll remove AS-Path prepends, as they didn't work.
![[c1 - set community for p2 & remove prepends.jpg]]

However, `p2` still prefers directly connected link to `c1` to reach 192.168.42.0/24 network! What's the problem?
![[p2 - BGP community & lower local preference.jpg]]
### The problem
The problem is that `p2` router doesn't receive alternative route from `u2` router! `u2` router prefers path via `p2 - c1` than via `u1 - p1 - c1` because of shorter AS-Path. And because it receives BGP best route from `p2` it doesn't advertise that route back to `p2`.
This scenario is called "BGP wedgies".
### Other stable scenario
Let's make an experiment. 
1. Stop advertising 192.168.42.0/24 network from `c1` to `p2` for a moment (for example, clear BGP session with `p2`).
![[c1 - clear ip bgp.jpg]]
2. Then `u2` will stop learning the network from `p2` and will prefer route from `u1` as the only one best route.
3. And `p2` will receive the route from `u2` now!
4. Then start advertising the network from `c1` to `p2` again, with community setting lower local preference. Now `p2` will accept the route from `c1`, but will have the best route via `u2`, because of local preference.
![[p2 - two routes.jpg]]
5. `u2` will have the best route via `u1`, because it will be the only route for `u2`. Remember, `u2` didn't receive the route from `p2`, because `p2` gets the best route from `u2`.
![[u2 - best via u1.jpg]]
Now we have desired state and it is stable.

However, let's make one more experiment.
1. Let's stop advertising 192.168.42.0/24 network from `c1` to `p1` (for example, clear BGP session with `p1`).
![[c1 - clear ip bgp with p1.jpg]]
2. Then `u2` will lose best route via `u1`.
3. Then `p2` will lose route via `u2` and will use route with low local preference received from `c1`.
4. Then `p2` will advertise the route from `c1` to `u2` and `u2` will use it, because it will be the only route available.
5. Now let's start advertising 192.168.42.0/24 network from `c1` to `p1` again. 
6. Then `u2` will have 2 routes to 192.168.42.0/24 - from `p2` and `u1`, but it will prefer route via `p2` because of shorter AS-Path. And therefore `u2` won't advertise the route to `p2`.
![[u2 - show ip bgp.jpg]]
7. And `p2` uses non-optimal route via directly connected link to `c1` despite of another route via `u1` available and despite we lowered local preference with BGP community.
![[p2 - show ip bgp - BGP wedgies.jpg]]
We are again in situation of "BGP wedgies".

You can see that there are two stable BGP states, one is desired for us and another one is not. And whether we're in one state or another depends on previous events in BGP convergence.

Ask question:
- How can we fix this? In other words, how can we enforce `u2` advertise the route received from `u1` to `p2`?
	- We must make for `u2` route from `u1` better than route from `p2`. We can do that by adding AS-path prepends.

Let's add AS-Path prepends to 192.168.42.0/24 network when advertising it to `p2`.
![[c1 - new route-map.jpg]]

Now `u2` has only one route to 192.168.42.0/24 learnt from `u1`.
![[u2 - new - show ip bgp.jpg]]

And now `p2` has the best route via `u2` because of combination of AS-Path prepends and local preference!
![[p2 - new - show ip bgp.jpg]]

Ask question:
- Is it possible that in some case `u2` will still prefer route via `p2`? I.e. our trick with AS-Path prepends won't work?
	- Yes, for example, `u2` may have higher LocalPref set for `p2`, then AS-Path prepends won't work and we will face again scenario of "BGP wedgies".

>[!HINT] Conclusion from this lab scenario is that in Internet with BGP we have full control of outbound traffic, but we can't fully manage inbound traffic, because our inbound traffic is somebody else's outbound traffic. 





