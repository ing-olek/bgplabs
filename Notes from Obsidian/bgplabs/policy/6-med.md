# Summary
[Use MED to Influence Incoming Traffic Flow « BGP Labs](https://bgplabs.net/policy/6-med/)
In scenario with 2 ISPs select best path for inbound traffic with AS-Prepends.
## Lab customization
No. FRR image is used, it works in WSL.
## Actions & Discussion
Check routes advertised by `rtr` to `x1` and `x2` before and after configuration.

Ask questions:
- How many prepends you need to make sure `x2` will prefer routes to 192.168.42.0/24 via `x1` rather than `rtr`?
	- 2 prepends, 1 might be not enough
- "When adding AS-Path prepends will not work?"
	- When peer router uses routing policy with attributes with higher preference in BGP best path selection, for example, LocalPref or weight.
	- If you don't have direct link with destination AS (i.e. there are other AS between you), other AS may also add AS-Path prepends to this route, you don't have control on this.
