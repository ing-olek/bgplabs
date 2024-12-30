# Summary
[Configuring Cumulus Linux and FRRouting « BGP Labs](https://bgplabs.net/basic/0-frrouting/) 
Try **netlab**.
## Lab customization
Cumulux Linux doesn't work in WSL, therefore it is replaced with FRR.

Another workaround is change provider from **clab** to **libvirt**. Then **vagrant** will automatically download image of Cumulux Linux and deploy VMs.
## Actions & Discussion
Check if FRR process is running with `ps -ef | grep frr` command.
![[Check if FRR process running.jpg]]


# My notes
## Cumulux Linux doesn't work in WSL in clab
The lab uses Cumulus Linux in containerlab with docker image networkop/cx4.4.0 - it doesn't work in WSL. But it works on my Ubuntu laptop.

```
TASK [run /tmp/config.sh to deploy initial config from /home/ig70ju/.local/lib/python3.12/site-packages/netsim/ansible/templates/initial/cumulus.j2] ***
fatal: [rtr]: FAILED! => changed=true
  cmd:
  - bash
  - /tmp/config.sh
  delta: '0:00:03.001291'
  end: '2024-11-19 13:20:38.712889'
  msg: non-zero return code
  rc: 1
  start: '2024-11-19 13:20:35.711598'
  stderr: |-
    ***  switchd is not licensed
    ***
    error: Another instance of this program is already running.
    ***  switchd is not licensed
    ***
    ***  switchd is not licensed
    ***
    Synchronizing state of frr.service with SysV service script with /lib/systemd/systemd-sysv-install.
    Executing: /lib/systemd/systemd-sysv-install enable frr
    Job for frr.service failed.
    See "systemctl status frr.service" and "journalctl -xe" for details.
  stderr_lines: <omitted>
  stdout: |-
    INIT: setting hostname
    INIT: creating loopback interface
    INIT: creating other interface
    INIT: executing ifreload
  stdout_lines: <omitted>
```

Therefore I'll replace `cumulus` with `frr` in `topology.yml` file and also updated text message.
```
ig70ju@WPO1L5CG2149XXW:~/bgplab/basic/0-frrouting$ cat topology.yml
# Configure a BGP session with an external BGP speaker

defaults.sources.extra: [ ../../defaults.yml ]
plugin: [ fix_arm ]

# Make BGP a valid node attribute so we can configure bgp.as on the node not running BGP
defaults.attributes.node.bgp:

name: frrouting
defaults.device: frr

nodes:
  rtr:
    bgp.as: 65000
  x1:
    module: [ bgp ]
    bgp.as: 65100

links:
- rtr:
  x1:

message: |
  The "Configuring FRRouting" lab is ready. Connect to the lab
  devices with the "netlab connect" command.

  Good luck!
ig70ju@WPO1L5CG2149XXW:~/bgplab/basic/0-frrouting$
```

## Running Cumulus Linux with libvirt & vagrant
Set **libvirt** as default provider in `topology.yml` file.
![[Set libvirt as provider.jpg]]

![[Cumulus linux with libvirt.jpg]]