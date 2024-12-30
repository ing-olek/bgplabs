# Summary
[Use Multiple AS Numbers on the Same Router « BGP Labs](https://bgplabs.net/session/3-localas/)
Use `local-as` BGP feature to represent your AS as another ASN.
## Lab customization
No. FRR image is used, it works in WSL.
## Actions & Discussion
In this scenario we'll explore `local-as` BGP feature. A router can have only one BGP AS number, but sometimes it is useful to represent itself to BGP neighbors as another AS.

On BGP labs portal one scenario is provided - when you have a router with private AS number and the router has two eBGP sessions with ISPs. Each ISP wants you to use different AS number, and you have to deal with it somehow.

Ask question:
- Can you think about scenario when `local-as` BGP feature can be useful?
	- Another scenario of using `local-as` is merging two autonomous systems. For example, company A having AS X acquires company B having AS Y. It means, you will have to change AS number on all routers of company B from Y to X. However, peers of company B must also change BGP configuration on their side when you change your AS number or eBGP sessions with them will go down. To not have complex change to coordinate and to not be dependent on peers, you can use `local-as` BGP feature, then company B router will represent itself as AS number Y, while actually running BGP process with AS number X.
- How does local-AS feature work, can you guess? What does it change in normal BGP neighborship?
	- It replaces local AS number in BGP "Open" message with new ASN, when establishing BGP session
	- Local-AS feature also adds new ASN to AS-Path when sending BGP updates to neighbor. As there is new AS between real local AS and neighbor AS, we'll see it in packet capture in a moment. 
	- It works in both directions, i.e. router with Local-AS feature enabled also adds new ASN as prepend to AS-Path when receiving BGP updates from the neighbor.

At start let's verify BGP sessions on `rtr`. We see that BGP session with `x2` is down and in logs we see hint - our neighbor reports it sees bad ASN from us.
![[rtr - BGP sessions - before.jpg]]

Let's configure `local-as` feature on `rtr` router.
![[rtr - Configure local-as.jpg]]

As you can see, BGP session with `x2` is up (you may need to clear BGP session to bring it up not waiting for timers). 
>[!NOTE] Note, that in BGP RIB of `rtr` route 192.168.101.0/24 from `x2` has AS 65007 as prepend to AS 65100. It means that Local-AS feature by default adds new AS as prepend to BGP Updates in both directions, "in" and "out".

![[rtr - BGP sessions after.jpg]]

Let's check how `x2` see BGP routes and neighborship with `rtr`. We see that `x2` sees `rtr` as AS 65007 and AS 65007 is added to AS-Path, while original AS 65000 is still there.
![[x2 - show ip bgp - after.jpg]]

The same we can see in packet capture of BGP messages.

In BGP "Open" message we see `rtr` one says: "Hello, I am router from AS 65007".
![[packet capture - BGP Open.jpg]]

And in BGP Update `rtr` prepends original AS 65000 with AS 65007.
![[packet capture - BGP Update.jpg]]

To pass checks with **netlab validate** successfully we need to remove AS 65000 from AS-Path of routes received by `x2`. I.e. we want to pretend, there is no AS 65000 at all, like AS 65007 really replaced AS 65000 for `x2` BGP neighbor.

We can do that with additional "nerd knob": `neighbor local-as no-prepend replace-as`.
First part, `no-prepend` removes new AS 65007 from AS-Path prepends inbound. And second part `replace-as` removes old AS 65000 from BGP updates sent to peer. You can use them separately.

See `local-as` with `no-prepend` option. We see that new AS 65007 was removed from AS-Path on `rtr`.
![[rtr - local-as with no-prepend.jpg]]

Now let's use `local-as no-prepend replace-as` option. We see that AS 65000 disappeared from AS-Path on `x2`.
![[x2 - local-as replace-as.jpg]]

