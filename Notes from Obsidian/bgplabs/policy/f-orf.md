# Summary
[Use Outbound Route Filters (ORF) for IP Prefixes « BGP Labs](https://bgplabs.net/policy/f-orf/)
Use ORF (outbound route filters) to filter not needed BGP prefixes on sender side (ISP router).
## Lab customization
No. FRR image is used, it works in WSL.
## Actions & Discussion
In this lab we'll see ORF feature, when your router tells BGP neighbor (e.g. ISP router) to apply route filter on its side. I.e. ISP router filters routes according to your filter before sending BGP updates to you.

Ask questions:
- "When does it make sense to use this feature?"
	- When you have small router and want to save CPU resources, therefore filter is applied on peer side.
	- When you have limited bandwidth and don't want to waste it with unnecessary BGP updates.

Obvious limitation of ORF is that BGP neighbor must support this feature. Two BGP neighbors agree on using ORF when they establish BGP session, because ORF is one of "BGP capabilities", i.e. BGP features supported by router.

Our task is to configure ORF filter and ORF BGP capabilities on `rtr` which will be applied on `x1` router in order to receive only default route + subnets of 172.16.0.0/16 network with mask length 24 or less.

Let's check BGP RIB at start.
![[Show BGP - before.jpg]]

Let's configure only prefix-list filter on "in".
![[Prefix-list filter only.jpg]]

Now let's see how prefix-list filter "in" filters BGP updates. Use `terminal monitor` and `debug bgp updates` command and reset BGP session with `clear ip bgp *`. 

You can see that some routes were "DENIED", i.e. `rtr` spent upstream bandwidth and CPU cycles on processing these updates.
![[Debug BGP updates - denied.jpg]]

Now let's enable ORF capabilities on `rtr` ("send") and `x1` ("receive"). 
>[!IMPORTANT] Normally, we don't configure external devices like `x1`, however in this lab we need to enable ORF on "receive" on `x1`.

![[rtr - enable ORF send.jpg]]
![[x1 - enable ORF receive.jpg]]

Now let's reset BGP session again and check if unnecessary routes were filtered by ORF.

On `rtr` we see much few routes - only those which are accepted: 0.0.0.0/0, 172.16.8.0/22.
![[rtr - debug - ORF applied.jpg]]

And on `x1` we see that some routes were suppressed by ORF filter!
![[x1 - debug - ORF applied.jpg]]