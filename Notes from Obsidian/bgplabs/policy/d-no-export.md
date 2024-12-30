# Summary
[Using No-Export Community to Filter Transit Routes « BGP Labs](https://bgplabs.net/policy/d-no-export/) 
Use well-known "NO_EXPORT" community to stop being transit AS.
## Lab customization
No. FRR image is used, it works in WSL.
## Actions & Discussion
Some time ago we already did lab to prevent our AS being transit for ISPs, using AS-Path access-lists (of course, we assume we are end customer, not ISP!).

In this lab we will do the same - prevent AS from being transit, using BGP well-known community "NO_EXPORT".

We used custom communities (or communities defined by ISPs) in the previous labs. 
There are few well-known BGP communities, which are defined in [RFC 1997: BGP Communities Attribute](https://www.rfc-editor.org/rfc/rfc1997.html) and [RFC 8642: Policy Behavior for Well-Known BGP Communities](https://www.rfc-editor.org/rfc/rfc8642) and are recognized by all BGP routers.

```        
			+-------------+-----------------------------------+
            | Numeric     | Common Name                       |
            +-------------+-----------------------------------+
            | 0:0         | internet                          |
            | 65535:0     | graceful-shutdown                 |
            | 65535:1     | accept-own [rfc7611](https://www.rfc-editor.org/rfc/rfc7611)                |
            | 65535:65281 | NO_EXPORT                         |
            | 65535:65282 | NO_ADVERTISE                      |
            | 65535:65283 | NO_EXPORT_SUBCONFED (or local-AS) |
            +-------------+-----------------------------------+
```

- "NO_EXPORT" means "don't advertise route outside your AS (e.g. via eBGP)"
- "NO_ADVERTISE" means "don't advertise route to any BGP peer"

We'll set "NO_EXPORT community" to routes receives from ISPs. That guarantees we don't advertise them to another ISP!


Ask questions:
- "Why being transit AS is bad for end customer?"
	- Being transit AS means that you forward traffic from one ISP to another ISP. That may overload your Internet links + you will pay ISPs for all that traffic!

Let's start.
We can see that ISP1 `x1` advertises 10.42.100.0/24 network to our AS65000. 
![[x1 - advertise route X.jpg]]

And we see that ISP2 `x2` receives that network 10.42.100.0/24 from us, i.e. our AS65000 is transit!
![[x2 - receive route X.jpg]]

Okay, we configure route-map setting BGP community "NO_EXPORT". We can use its name in BGP configuration instead of XX:XX format, because it is well-known.
![[c1 - NO_EXPORT 1.jpg]]

However, we still see that `c2` advertises 10.42.100.0/24 to `x2`. It turns out, `c2` doesn't have BGP community attached to that route.
![[c2 - no community.jpg]]

Ask question:
- "What can be the problem?"
	- On some platforms we need to enable sending BGP communities between peers.

Okay, let's fix it. And apply the same config on `c2` for routes received from ISP2 (`x2`).
![[c1 - send-community both.jpg]]

Now we see that both `c1` and `c2` advertise ISPs only local 192.168.42.0/24 prefix!
![[c1 - advertise ISP only local.jpg]]

![[c2 - advertise ISP only local.jpg]]

>[!NOTE] Note that we didn't configure route-maps on `c1` and `c2` to match BGP community "NO_EXPORT"! There is no need, because it is well-known BGP community and all modern BGP implementations recognize it!

