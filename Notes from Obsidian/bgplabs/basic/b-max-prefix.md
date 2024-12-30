# Summary
[Limit the Number of Accepted BGP Prefixes « BGP Labs](https://bgplabs.net/basic/b-max-prefix/) 
Set max-prefix-limit for number of prefixes received from BGP peer.
## Lab customization
No. FRR image is used, it works in WSL.
## Actions & Discussion
Configure BGP neighbor logging and `terminal monitor` to see log messages in console.
In `address-family ipv4 unicast` configure max limit of prefixes from BGP peer with command `neighbor <peer ip> maximum-prefix <max prefix limit> <% of limit - warning threshold>`.

Use `sh start <prefix number>` and `sh stop` commands to start and stop generation of many prefixes by customer router.

Ask questions:
- "What is a reasonable number of max prefix limit for BGP session?"
	- I think it depends on "normal" amount of prefixes, let's say multiplied several times
	- It also shouldn't be higher than size of RIB / FIB on the network device
