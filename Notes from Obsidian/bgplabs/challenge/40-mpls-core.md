# Summary
[BGP-Free Core in a Transit Network « BGP Labs](https://bgplabs.net/challenge/40-mpls-core/)
Configure MPLS L3VPN in ISP to have BGP-free core.
## Lab customization
FRR image requires installation of Linux kernel module to support MPLS, but it fails in WSL.
![[Install modprobe MPLS Linux kernel module - fail.png]]

There is manual in Internet how to make your own Linux kernel based on WSL and add necessary modules to it.
## Actions & Discussion