For some labs we required network device OSes running as VMs in libvirt/vagrant environment, because some features are not supported in containerlab (example - basic/9-ao).

**Libvirt** and **vagrant** tools are installed during **netlab** installation. However, we'll need to download network OS images and build VMs from them.

# Arista vEOS with libvirt
## Build Vagrant box for Arista vEOS
Follow instructions from netlab site: [Building an Arista EOS Vagrant Libvirt Box — netlab documentation](https://netlab.tools/labs/eos/).

First, download Arista vEOS image from Arista portal: [Software Download - Arista](https://www.arista.com/en/support/software-download). It is free after registration.
> [!TIP] Registration with @gmail.com etc public domains doesn't work. You can use your @ing.com email.

We will use vEOS64-lab-4.33.0F.qcow2 image.
![[Arista images - portal 1.jpg]]

Create an empty directory and place there downloaded image file.
Start building vagrant box for vEOS file with `netlab libvirt package eos <image>` command.
![[Build Arista image.jpg]]

After booting the image login with `admin` username (and empty password) and disable Zero-touch provisioning with `zerotouch disable`.
![[Login admin & disable zerotouch.jpg]]

Login again as `admin` and apply initial config
```
Creating initial configuration for Arista EOS
=============================================

* Wait for the 'login' prompt
* Login as 'admin' (no password)
* Disable zero-touch with 'zerotouch disable'

===============
*** WARNING ***
===============
Disabling zero-touch (which will also cause a device reboot) is crucial. With
zero-touch enabled, the Vagrant box will acquire an IP address on the management
interface, but never start SSH, resulting in a stuck "vagrant up" process.

After the system reboot

* Login as 'admin'
* Go into enable mode, enter configuration mode
* Copy-paste the following configuration

NOTE: the management traffic is isolated in a dedicated management VRF (management).

=============================================
aaa authorization exec default local
!
no aaa root
!
service routing protocols model multi-agent
!
vrf instance management
!
username vagrant privilege 15 secret sha512 $6$3kgdKcJLJ3j/0N51$a0YshIzKL3xtdwP6XXXRlY9B8yHFK/tLdg0I95YUIaW7oHqLsgK9TxMg8/0bL6VDkImuWT.g7WRKTxi8nNPtA1
username vagrant ssh-key ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEA6NF8iallvQVp22WDkTkyrtvp9eWW6A8YVr+kz4TjGYe7gHzIw+niNltGEFHzD8+v1I2YJ6oXevct1YeS0o9HZyN1Q9qgCgzUFtdOKLv6IedplqoPkcmF0aYet2PkEDo3MlTBckFXPITAMzF8dJSIFo9D8HfdOV0IAdx4O7PtixWKn5y2hMNG0zQPyUecp4pzC6kivAIhyfHilFR61RGL+GPXQ2MWZWFYbAGjyiYJnAmCP3NOTd0jMZEnDkbUvxhMmBYSdETk1rRgm+R4LOzFUGaHqHDLKLX+FIPKcF96hrucXzcWyLbIbEgE98OHlnVYCzRdK8jlqm8tehUc9c9WhQ== vagrant insecure public key
!
interface Management1
   vrf management
   ip address dhcp
   dhcp client accept default-route
!
no ip routing
no ip routing vrf management
!
security pki key generate rsa 2048 default
security pki certificate generate self-signed default key default param common-name Arista
!
management api http-commands
   no shutdown
   !
   vrf management
      no shutdown
!
management api netconf
 transport ssh default
!
management api restconf
 transport https default
  ssl profile default
  port 6040
!
management security
 ssl profile default
  certificate default key default
!
management ssh
   vrf management
      no shutdown
!
no user vagrant shell
!
end
=============================================

* Save the configuration with 'wr mem'
* Poweroff the VM with 'bash sudo poweroff'
* If the device starts reloading instead of shutting down, disconnect
  from the console (ctrl-] usually works).
```

![[Create initial config.jpg]]

Save config and poweroff the VM. Press `Ctrl + ]` to stop constant rebooting.
![[Save config & Poweroff VM.jpg]]

Enter box version - 4.33.0F. You may see error message in the end, but we'll fix it soon.
![[Add box version.jpg]]

To run **Vagrant** in WSL we need to set environment variable `VAGRANT_WSL_ENABLE_WINDOWS_ACCESS=1`.
See: [Vagrant and Windows Subsystem for Linux | Vagrant | HashiCorp Developer](https://developer.hashicorp.com/vagrant/docs/other/wsl).

To make change in environment variable permanent, add that command to `~/.bashrc` file (this script is executed on every logon to WSL).
![[Pasted image 20241122124002.png]]
Now we are ready to use Arista vEOS devices in labs!
## Using Arista vEOS devices in labs
To use Arista vEOS devices with **libvirt** provider in labs, you may need to update `topology.yml` file of the lab.
See: [Topology Defaults — netlab documentation](https://netlab.tools/defaults/#changing-defaults-in-lab-topology).

For example, in lab `basic/9-ao` add this block after other defaults settings.
```
defaults:
  device: eos
  provider: libvirt
```

![[Use Arista vEOS in labs.jpg]]