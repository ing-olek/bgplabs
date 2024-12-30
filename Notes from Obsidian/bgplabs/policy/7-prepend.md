# Summary
[Use AS-Path Prepending to Influence Incoming Traffic Flow « BGP Labs](https://bgplabs.net/policy/7-prepend/)
In scenario with 2 ISPs select best path for inbound traffic with AS-Prepends.
## Lab customization
No. FRR image is used, it works in WSL.
## Actions & Discussion
Check routes to 192.168.42.0/24 on `x1` and `x2` before and after configuration.

Ask questions:
- "What if you use AS-Path prepends instead of MED? Are there any side effects?"
	- AS-Path prepending will work to make routes via `x1` more preferred than via '`x2'
	- However, other AS will see your routes 192.168.42.0/24 via that ISP with additional prepends. It may be not desired, if you have two or more ISPs. Otherwise, MED will impact only your neighboring AS.
