# Summary
[EBGP Sessions over IPv6 LLA Interfaces « BGP Labs](https://bgplabs.net/basic/d-interface/) 
Configure IPv4 (!) eBGP session over IPv6 LLA (link-local addresses), using "interface" instead of IP in `neighbor` command.
## Lab customization
No. FRR image is used, it works in WSL.
## Actions & Discussion
In this lab we'll use a trick with configuring eBGP session with LLA (IPv6 link-local addresses). 
Neighbor IPv6 LLA will be automatically discovered by ICMPv6, therefore we don't need to know neighbor IPv6 address, but can put "interface name" in `neighbor` command.
Moreover, we'll configure BGP session for IPv4 address-family over IPv6 link!
### Start - Enable IPv6
To start, let's check if `rtr` has IPv6 addresses configured on interfaces.

As we can see IPv6 addresses are not configured on `eth1` and `eth2`.
![[Start - no IPv6 interfaces.png]]

In this lab we use FRR images, to enable IPv6 on interfaces in FRR we need to use Linux commands in `bash`.

To enable IPv6 in Linux we need to change kernel tunable parameters (located in `/proc/sys/net`) with `sysctl` tool. 
See:
- [Kernel Administration Guide | Red Hat Product Documentation](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html-single/kernel_administration_guide/index?extIdCarryOver=true&sc_cid=701f2000001Css5AAC#network_interface_tunables)
- [1.6. Using Network Kernel Tunables with sysctl | Red Hat Product Documentation](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/networking_guide/sec-using_network_kernel_tunables_with_sysctl#sec-Using_Network_Kernel_Tunables_with_sysctl) 
![[FRR - Linux kernel parameters proc_sys_net.png]]

Before configuration there is no IPv6 address on Linux interfaces.
![[Linux - IPv6 - before.png]]

Let's enable IPv6 with `sysctl` on Linux interfaces.
![[sysctl - enable IPv6 addresses.png]]

And now let's check IPv6 addresses on interfaces. Now we see IPv6 link-local addresses on interfaces.
![[Linux - IPv6 - after.png]]

And the same inside FRR.
![[FRR - IPv6 enabled.png]]

### Configure IPv6 neighbor discovery RA
Next mandatory step is to configure IPv6 NDP (Neighbor discovery protocol) to send RA (Router Advertisement) messages on interfaces. 
See more: [Neighbor Discovery Protocol - Wikipedia](https://en.wikipedia.org/wiki/Neighbor_Discovery_Protocol)
>[!TIP] Neighbor discovery protocol is like ARP for IPv6 (there is no ARP for IPv6). RA in ND is typically used by IPv6 clients (end hosts) to get default gateway (router's IPv6 link-local address) when clients have IPv6 addresses auto-configured with SLAAC.

![[IPv6 - ND - enable RA.png]]

And let's set lower RA advertisment interval (e.g. 30 seconds instead of default 600 seconds).
![[IPv6 - ND - set lower RA interval 1.png]]
### Configure IPv6 BGP with LLA
We should use `neighbor <intf> interface remote-as <remote-AS>` command to configure BGP neighbors.

#### Address-family IPv4 
And don't forget to activate BGP sessions in IPv4 address-family!
>[!NOTE]BGP is capable to exchange various routing information over one TCP session. These kinds of routing information are called Address-families (AFI) or Sub-Address-families (SAFI). See [RFC 4760 - Multiprotocol Extensions for BGP-4](https://datatracker.ietf.org/doc/html/rfc4760).

Ask questions:
- "What address-families do you know?"
	- IPv4 unicast
	- IPv6 unicast
	- VPNv4 unicast
	- VPNv6 unicast
	- L2VPN vpls
	- L2VPN evpn
	- IPv4 multicast
	- IPv4 mdt 
	- etc.

We see, that BGP session with `x2` is now up, but with `x1` it is still down.
![[BGP - configure over IPv6 LLA.png]]

#### Removing /30 IP address from eth1
The problem is that interface `eth1` has mask /30 and in this case some network OS (including FRR) try to establish BGP session via IPv4.
![[Interface eth1 - M30.png]]

When we removed /30 IP address from `eth1` and configured /32 instead, both BGP sessions are up.
![[Both BGP sessions up.png]]

#### ICMPv6 & BGP - packet capture
Let's look what happened under the hood in ICMPv6 and BGP over IPv6.

In another window let's start capture on `rtr eth1` interface.
![[Netlab - traffic capture.png]]

Then login to `x1` and flap `eth1` interface. It will re-start ICMPv6 Router Advertisement and re-establishes BGP session over IPv6.
![[x1 - IPv6 interface.png]]

Then, open tcpdump with Wireshark - indeed, we see ICMPv6 Router Advertisement and BGP session over IPv6.
![[ICMPv6 and BGP - tcpdump.png]]
