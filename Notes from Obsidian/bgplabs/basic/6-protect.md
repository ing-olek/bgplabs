# Summary
[MD5 Passwords and GTSM « BGP Labs](https://bgplabs.net/basic/6-protect/) 
Enable TTL Security and TCP MD5 authentication to protect BGP sessions.
## Lab customization
TCP MD5 authentication doesn't work in FRR (TTL Security works). Therefore we have to use another network OS or provider. 
For example, it works with Arista vEOS and **libvirt**. Before using Arista vEOS and libvirt we have to download vEOS image and build **vagrant** box for it.
## Actions & Discussion

### TTL Security
First, let's make sure both BGP sessions are down.
![[BGP session down - before.jpg]]
>[!NOTE] Sometimes IP addresses of nodes in lab are different from the ones in lab description on https://bgplabs.net/

As our topic is TTL security, let's check TTL in BGP messages between `rtr` and `x1`.

We can see that `rtr` sends BGP packets with TTL = 1, while `x1` sends BGP packets with TTL = 255. Probably, that's the issue.
![[tcpdump - TTL Security - before 1.jpg]]
After configuring TTL security with `neighbor 172.16.0.10 ttl maximum-hops 1` command we see that BGP session with `x1` is up.

### TCP MD5 authentication
In packet capture on `rtr` we can  see that BGP messages from `x1` are signed with MD5 hash, while messages from `rtr` are not. 
![[TCP MD5 - before.jpg]]

Let's configure TCP MD5 authentication with `neighbor 172.16.1.11 password GuessWhat` command. Then we see the BGP session is up.

Let's verify that TCP MD5 checksum is now present in TCP packets of BGP messages. 
Now we'll do packet capture with `netlab capture` command and save capture to file in lab directory.
![[netlab - packet capture.jpg]]

Then open the file with Wireshark. We can see that `rtr` sends "BGP Open" message and in TCP header there is TCP option - MD5 hash of the packet.
![[TCP MD5 - after 1.jpg]]
