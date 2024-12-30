# BGP labs

BGP labs is a free course on BGP configuration: [Labs Overview « BGP Labs](https://bgplabs.net/).

This course contains many scenarios on how to configure BGP and use BGP features. Each scenario has lab for practice. 

Labs have prepared basic configuration (nodes created, links between nodes added, IP addresses are configured on interfaces, etc.).

Netlab ([netlab: a Virtual Networking Labbing Tool — netlab documentation](https://netlab.tools/) ) is used as simulator for labs. It uses `containerlab`, `libvirt` or `VirtualBox` to run OS images of network devices. Netlab supports many vendors and platforms, like FRR, Cumulus Linux, Arista, Cisco, etc.

As it may be difficult to install and run Ubuntu VM with nested virtualization on corporate laptops, we will run **netlab** in WSL.

There is also option to run labs in GitHub Codespaces:  [Use GitHub Codespaces « BGP Labs](https://bgplabs.net/4-codespaces/). It is for free, you need to register on GitHub and you have 120 "code hours" per month for free.

> [!CAUTION]
> If you use GitHub Codespaces, don't put any sensitive information there! 
# WSL installation
If you don't have WSL you need to install if following instructions below.
## Install WSL
![[Pasted image 20241118102933.png]]

## Create new user
After installation you'll be asked to create new user and set password for him/her. You can use any username.
![[Pasted image 20241118103123.png]]

## Verify 'sudo' command working
In WSL you'll use `sudo` command to execute commands as `root`. By default WSL adds new user you created step before to `sudoers` group.
![[Install WSL 3 - sudo.jpg]]

## Verify DNS resolution and systemd running
Verify if you can resolve DNS names from WSL. If necessary, edit `/etc/resolv.conf` file.

> [!TIP]
> In the latest version Ubuntu VM is installed in WSL already with `systemd` (i.e. you can run services with `systemctl`).

![[Install WSL 4 - resolv_conf & systemd.jpg]]
# Netlab installation
To install **netlab** in WSL follow instructions for [Ubuntu Server Installation — netlab documentation](https://netlab.tools/install/ubuntu/) .

First, install `networklab` Python package.
![[Install netlab.jpg]]

Second, using `netlab` install `ansible, libvirt, containerlab`. Answer "yes" to questions during installation.
![[Install netlab 2.jpg]]

There maybe conflicts with versions of existing packages and installation may fail (for example, you already have `ansible` installed or you have newer version of `vagrant`). Then, delete existing packages with `sudo apt remove` and run installation process again.

After successful installation of **netlab** you will see message like below.
![[Install netlab 6.jpg]]

Run `netlab test clab` command to test **netlab**.
![[Test Clab 1.jpg]]

Some output omitted.
![[Test Clab 2 1.jpg]]

# Clone BGP labs
Once **netlab** is successfully installed, it's time to actually start labs! 

Clone BGP labs from the repo:
`git clone https://github.com/bgplab/bgplab.git`

![[BGP labs - clone repository.jpg]]

# Customising netlab labs
Netlab has limitations when working in WSL (because it is not native Ubuntu/Linux in VM or bare metal server), therefore we'll have to change some settings in configuration files.

If we need to change settings for a specific lab, we can do it in `topology.yml` file in lab directory. 

See documentation on how to override default settings: [Topology Defaults — netlab documentation](https://netlab.tools/defaults/).

# Using netlab
Let's get familiar on how to use **netlab**!
## Lab topology & scenario
As we use pre-defined BGP labs, lab topology, scenario, devices and their IP addresses are described on https://bgplabs.net/.

For example, see [Configure a Single EBGP Session « BGP Labs](https://bgplabs.net/basic/1-session/).
### Lab topology
![[bgplabs - example topology.jpg]]
### Lab scenario
![[bgplabs - example scenario.jpg]]
### Info about devices and IP addresses
![[bgplabs - example devices.jpg]]
## Start lab
To start lab, go to the labs directory and run `netlab up` command.
![[Running lab 1.jpg]]

If lab is running successfully, in the end you'll see message like this.
![[Running lab 3 1.jpg]]
## Show lab status
You can see if your lab is running by executing `netlab status` command from the lab directory.
That command is also useful to see device names in the lab. You'll need device name to connect to it in the next step.

If you are in another directory, you can use that command with `--all` flag: `netlab status --all`.
![[Lab status 1.jpg]]
## Connect to network device
You can connect to device in the lab with `netlab connect <device-name>` command. 

Then you can configure the device and verify routing with `show` commands!
You can also save configuration with `write` command. To exit use `exit` command.

![[Lab - connect to device.jpg]]
## Verify configuration
Once you configured all network devices in the lab, you can check your configuration. 
Of course, you can login to lab devices and verify config with `show` commands. 

Also you can verify your solution with `netlab validate` command.
![[Lab - validate config.jpg]]
## Collect configuration
After completing the lab you may export device configuration to the lab directory with `netlab collect` command.
![[Lab - collect config.jpg]]

Device configuration is saved to `config` sub-directory in the lab directory.
![[Lab - collect config 2.jpg]]

## Load saved configuration
**Netlab** can load device config previously saved with `netlab collect`. I.e. you can continue lab from where you stopped last time.

>[!IMPORTANT] Note, **netlab** doesn't replace default initial config with saved config, it MERGES initial config with saved config.

>[!TIP] You can load any custom config to network devices as well. Only it should be understood by **netlab** and network OS images. See details in documentation: [Custom Configuration Templates — netlab documentation](https://netlab.tools/custom-config-templates/) 

>[!TIP] It doesn't matter, if you saved configuration on network device with `write memory` (or similar command) or not. To save your config for future use it is important to export it with `netlab collect` command.

See more in **netlab** documentation: [Deploying Custom Device Configurations — netlab documentation](https://netlab.tools/netlab/config/).

There are two options how to load saved config.
### Load config when starting lab
Use `netlab up --reload-config <config directory>` command when starting **netlab**.
![[Load config - way 1.jpg]]

After lab is started you can login to network device and make sure saved config was loaded.
![[Load config - way 1 - config loaded.jpg]]
### Load config after lab started
Another way to load saved config is to start lab with `netlab up` command as usual and later use `netlab config --reload <config directory` command.
![[Load config - way 2.jpg]]

After loading config you can verify it.
![[Load config - way 2 - config loaded.jpg]]

## Traffic capture
You can capture traffic when logged in to network devices, but not all devices support this option (e.g. `tcpdump` works in Arista vEOS).

And you can also capture traffic with `netlab capture` command, providing device and interface name. To see device's interface names use `netlab report --node <device> addressing` command.
![[Netlab - capture.jpg]]

It's possible to save packet capture in file with `-w <output_file>` of **tcpdump** utility. The file will be saved in the lab directory. Then you can open it with **Wireshark**.
![[Netlab - capture 2.jpg]]
### Wireshark profiles
**Wireshark** supports configuration profiles, which shows packet diagrams or use custom color scheme.

You can try various profiles downloading them from GitHub: [https://github.com/amwalding/wireshark_profiles](https://github.com/amwalding/wireshark_profiles)

Below is example of **Wireshark** packet capture with "Better Default" profile.
![[Personal/ALPE/bgplabs/0. Installation & setup/imgs/TCP MD5 - after.jpg]]

### Traffic capture with libvirt
When you use **libvirt** as VM provider, packet capture doesn't work on p2p interfaces. 
To capture traffic on **libvirt** nodes you can change interface type from UDP tunnels to Linux bridge, see: [Using libvirt/KVM with Vagrant — netlab documentation](https://netlab.tools/labs/libvirt/#libvirt-capture).

You can do it in lab topology file or globally. For example, to change user-wide settings, create `.netlab.yml` file in your user's home directory.
![[Libvirt - LAN bridges.jpg]]
## Stop lab
Once you are done with the lab, you should stop it with `netlab down` command. This command destroys containers where OS images were running.
![[Stop lab.jpg]]

> [!IMPORTANT]
You can run only one lab at a time! If one lab is already running and you'll start another one, **netlab** will raise an error.
You can find lab running with `netlab status` command. Then go to its directory and stop it with `netlab down` command.



