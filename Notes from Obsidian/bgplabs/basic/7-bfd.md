# Summary
[Use BGP Timers and BFD to Speed Up BGP Convergence « BGP Labs](https://bgplabs.net/basic/7-bfd/) 
Adjust BGP keepalive and hold timers and use BFD in BGP session.
## Lab customization
No. FRR image is used, it works in WSL.
## Actions & Discussion
Ask questions:
- What are default BGP timers?
	- Keepalive - 60 sec, Hold time - 180 sec
- Must BGP keepalive and hold time match on both peers to establish BGP session?
	- No, smaller keepalive is used and hold time is 3 * keepalive

### Simulate BGP convergence with default timers
Let's simulate BGP convergence with default timers. 
>[!TIP] To do that, remove IP address from `r1 - x1` link. If you shutdown the interface, BGP will tear down session.

On `r1` enable logging to console: `terminal monitor` and `debug zebra rib` to track changes in routing table.
![[Simulate BGP convergence - default timers.jpg]]

After removing IP address from `r1 eth1` interface BGP session with `x1` remains up for almost 3 minutes.
![[Simulate BGP convergence - default timers 2.jpg]]

Accordingly, only 3 minutes after BGP update reached `r2`.
![[Simulate BGP convergence - default timers 3.jpg]]
As you can see, with default BGP timers convergence took about 3 minutes!
### Optimize BGP timers
Let's set BGP timers to: keepalive = 3, hold time = 9 and repeat experiment.

![[BGP - changed timers.jpg]]

Now convergence is much faster!
![[BGP timers changed.jpg]]
![[BGP - changed timers 2.jpg]]

![[BGP - changed timers 3.jpg]]

### Use BFD with BGP sessions
Now let's configure BFD on BGP session between `r1` and `x1`. First, we create BFD profile with timing settings, then apply it to BGP session.
![[Enable BFD.jpg]]

Verify BFD settings. We set BFD transmit-interval and receive-interval = 100ms, but BFD uses timers = 300ms. Why?
Because during  negotiation BFD selects max (local transmit, remote receive) timers! See: https://datatracker.ietf.org/doc/html/rfc5880#section-6.8.7.
![[BFD peers.jpg]]

![[BGP peer - BFD.jpg]]

In the end let's repeat our experiment with BGP convergence.
![[BGP convergence - BFD.jpg]]

Now convergence takes < 1 second!
![[BFD peers 2.jpg]]
