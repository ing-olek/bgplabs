# Summary
[TCP Autentication Option (TCP-AO) « BGP Labs](https://bgplabs.net/basic/9-ao/) 
Configure TCP Authentication Option (AO) for BGP, which is next generation of securing TCP session (after MD5 hashes).
## Lab customization
TCP MD5 authentication doesn't work in FRR (TTL Security works). Therefore we have to use another network OS or provider. 
For example, it works with Arista vEOS and **libvirt**. Before using Arista vEOS and libvirt we have to download vEOS image and build **vagrant** box for it.
## Actions & Discussion
TCP AO (like TCP MD5) is used not only in BGP, but this is general mechanism of digital signature of TCP sessions. 

TCP AO is better than TCP MD5 because:
- It supports stronger cryptographic algorithms than MD5 (SHA-1, SHA-256, AES-128-CMAC)
- It has protection against replaying for long-lived TCP sessions (like BGP session between peers)

See example how to configure on Cisco: [IP Routing: Protocol-Independent Configuration Guide, Cisco IOS XE Gibraltar 16.12.x - TCP Authentication Option [Cisco IOS XE 16] - Cisco](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/iproute_pi/configuration/xe-16-12/iri-xe-16-12-book/tcp-ao.html)

Configuration example on Arista EOS: [Configuration-examples/Arista EOS.md at master · TCP-AO/Configuration-examples · GitHub](https://github.com/TCP-AO/Configuration-examples/blob/master/Arista%20EOS.md)

In the beginning let's check that both BGP sessions are down.
And let's check existing security settings with `show running-config | sec management security` command.

We need to create key chains (in Cisco) or shared-secret profiles (in Arista). 
>[!TIP] In TCP AO keys have 'send-id' and 'receive-id' which must match on sender and receiver side (accordingly).

Therefore, it makes sense to login to `x1` and `x2` and check what IDs are there (they = "0", therefore we must create two shared-secret profiles each having key with ID = 0).

```
rtr(config)#management security
rtr(config-mgmt-security)#session shared-secret profile BGP-1
rtr(config-mgmt-sec-sh-sec-profile-BGP-1)#secret 0 BigSecret infinite
rtr(config-mgmt-sec-sh-sec-profile-BGP-1)#ex
rtr(config-mgmt-security)#session shared-secret profile BGP-2
rtr(config-mgmt-sec-sh-sec-profile-BGP-2)#secret 0 GuessWhat infinite

rtr(config)#router bgp 65000
rtr(config-router-bgp)#neighbor 10.1.0.2 password shared-secret profile BGP-1 algorithm hmac-sha1-96
rtr(config-router-bgp)#neighbor 10.1.0.6 password shared-secret profile BGP-2 algorithm hmac-sha1-96
rtr(config-router-bgp)#end
```

After config is applied we see that both BGP sessions are up.
In **tcpdump** we see that TCP AO digest is added to TCP packets.
![[Tcpdump - TCP AO - after.jpg]]

And this is how it looks like in **Wireshark**.
![[Wireshark - TCP AO 1.jpg]]
