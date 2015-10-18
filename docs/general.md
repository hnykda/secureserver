Now some general but serious recommendations and system administration tasks. 

# Maintaining the packages
Every Linux distribution comes with package manager. Arch Linux has `pacman` and we've already
used that in the previous chapter. This is used for managing for installing virtually all 
software you are going to use.  

## Keep number of installed packages low
Less software usually means less maintenance resources, less space used and lower risk of attack. 
When you get lost in your folder structure and lost track of installed software (what does what), how can you
be sure that you are not a *happy* user of some Trojan or malware? Programs might also have bugs which might
be exploited by attacker to get into your system. It get us to another aspect. What *low* means? Of course
it can be exactly specified and it doesn't depend on your specific usage - the magical number
using bleeding edge astrology approaches says...

...262 packages! That's exactly what I have on my system and that is still quite manageable.

## Up-to-date system 
Maintainers are usually hard working on patching their applications while the security
holes are reported. For that reason is absolutely necessary to use up-to-date software on your system.
On Arch Linux you use `pacman -Syu` to keep your system updated. 

Also try to know about development of security critical packages. Is a development active? If their 
github repository shows last commit 5 years ago and number of reported issues is increasing, run!

# Managing services 
Sometimes referred as daemons, these workhorses are usually running at background. For example in next chapter we will use SSH - it will run at background. Services are (on modern systems) managed by [controversial]() service manager  `systemd`.  `systemd` is controlled by `systemctl` (besides other, now irrelevant commands). To start some program, which is in this context called service (and we will stick to that), just run `systemctl start <unit>`. There are other useful (and that’s 90% of what you need to know about `systemctl`) commands (all starts with systemctl and ends with `desired_unit` - watch example):

* `enable` - this allow to run service after boot (but it will not start immediately)
* `disable` - this will make device not to start after boot
* `start` - this will immediately start a service (but will not enable it - it won’t be run after boot)
* `stop` - stop service immediately (but not disable)
* `status` - this will print out all information in pretty format - you can find if it is enabled, started, if there are any errors etc.

## Example
There is service, which takes care about connection to network. We will play with that for a minute now. It’s called `systemd-networkd`. Try to start it, enable it, disable it and then stop it and get status to see what every command does by trying these:

```
$ systemctl start systemd-networkd
$ systemctl status systemd-networkd
$ systemctl enable systemd-networkd
$ systemctl status systemd-networkd
$ systemctl disable systemd-networkd
$ systemctl status systemd-networkd
$ systemctl stop systemd-networkd
```

Last thing you need to know about `systemd` for our guide is where these services has it’s own configuration files. They are stored either in `/usr/lib/systemd/system/` or `/etc/systemd/system`. For example, I’ve noticed SSH service. Configuration file for this service is in `/usr/lib/systemd/system/sshd.service`. You can type `systemctl cat sshd` to see what is inside and of course it can be edited.

## Keep number of running services low
As in previous part with packages, try to keep number of running services low for the same reasons. It's easier to keep track what is 
now happening on your server, takes less resources and cannot be used for attacks and forgery. How many is *low number*? I have 14.

# Network settings
For following chapters is essential to have your network connection set up correctly. First of all we have to assure that you are going to have
still the same IP address. IP address isn't nothing else then just 4 numbers separated by dot and its purpose is same as the number of your room in your
house. Why room of the house and not the address of house itself? Well, your house has also IP which in general differs from the one your computer has inside the house. 
More about local and public IPs can be read [here](). 

Let's assume you are connected using ethernet connection. After connection RPi to the ethernet port in your router, the connection is usually established automatically and your router assigned your RPi by some pseudorandom IP address.To see what IP you are currently assigned can be done by `ip addr`. The output should be something like:
```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether b8:27:eb:2d:25:18 brd ff:ff:ff:ff:ff:ff
    inet 192.168.0.201/24 brd 192.168.0.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::ba27:ebff:fe2d:2518/64 scope link 
       valid_lft forever preferred_lft forever
```

`eth0` is ethernet connection and you can notice that my current IP is `192.168.0.201`.

We want to make this IP address static - it won't change if you unplug and plug again your device.  And how to choose static address? As you know your router is assigning IP address automatically (it is called DHCP). But not randomly in full range. It has some range of IP addresses which it can assign. Standard IP address of router is usually `192.168.0.1` and then assign addresses from `192.168.0.2` to `192.168.0.254`. Second standard is `10.0.0.138` for router and then it assigns addresses from `10.0.0.139` to `10.0.0.254`. But it can be anything else.

I suggest to set one the address on the end from this range. You can notice, that my `eth0` has IP address `192.168.0.201`.

Open or create a file `/etc/systemd/network/ethernet_static.network` and paste this:

```
[Match]
Name=eth0

[Network]
Address=the.static.address.rpi/24
Gateway=your.router.ip.address
```

my case:

```
[Match]
Name=eth0

[Network]
Address=192.168.0.111/24
Gateway=192.168.0.1
```

Now you need to remove old non-static default profile `/etc/systemd/network/eth0.network`. Move it to your home folder so you can revert changes if something doesn't work.
