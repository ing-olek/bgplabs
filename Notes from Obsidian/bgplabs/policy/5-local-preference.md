# Summary
[Select Preferred Uplink with BGP Local Preference « BGP Labs](https://bgplabs.net/policy/5-local-preference/) 
Select best path for outgoing traffic with LocalPref attribute on two routers with one ISP.
## Lab customization
No. FRR image is used, it works in WSL.
## Actions & Discussion
Ask questions:
- "Is LocalPref attribute sent in BGP messages to eBGP peer?"
	- No, therefore LocalPref is used for outgoing traffic inside an AS
- "What is default value of LocalPref?"
	- Many vendors set default LocalPref = 100, however RFC doesn't specify that.

Check `show ip bgp` before and after configuration.


