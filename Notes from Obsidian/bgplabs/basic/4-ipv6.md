# Summary
[Configure BGP for IPv6 « BGP Labs](https://bgplabs.net/basic/4-ipv6/) 
Configure eBGP session with ISP for IPv6 (dual stack).
## Lab customization
No. FRR image is used, it works in WSL.
## Actions & Discussion
In this lab we'll configure eBGP sessions with ISP via IPv6 and advertise our IPv6 prefix 2001:db8:1::/48.

### IPv6 addresses
IPv6 protocol itself is a large topic, we won't cover it in this lab in detail. Just a reminder of IPv6 address structure:
- it consists of 16 bytes, i.e. 128 bits. Bytes are written in hex format
- each group of 2 bytes is divided by colons ":"
- leading zeroes in group of 2 bytes can be omitted
- once in IP address group of all zero bits can be replaced by "::"

See more:
- [IPv6 address - Wikipedia](https://en.wikipedia.org/wiki/IPv6_address)
- [IPv6 - Wikipedia](https://en.wikipedia.org/wiki/IPv6)
![[Ipv6_address_leading_zeros.svg.png]]
### Configure IPv6 BGP session
Notes:
- Configure IPv6 BGP session as usual with `neighbor` command, use public IPv6 addresses of peers
- Verify IPv6 BGP session is up with `show bgp ipv6 summary` command
- Add IPv6 prefix to BGP routing (use `address-family ipv6`)
- Add static IPv6 route to Null0 
- Verify IPv6 prefix 2001:db8:1::/48 is advertised to peers

> [!NOTE] Note: in BGP table we see IPv6 routes with link-local address!

![[IPv6 routes - Link-local next-hops.png]]