You just arrived to the site maintained by Daniel Hnyk. This site should provide practical example how to set up your Linux server from scratch. No deep knowledge is necessary and you shouldn't need to know anything more except typing commands into bash shell.

## Purpose
I enjoy programming and about two years ago it was obvious that I could get a lot from having my own server used for various uses like:

1. Web server - hosting webpages which I build in Django, Rails, Nodejs or plain HTML
2. Proxy - sometimes it's quite handy to have your own proxy for security or getting to national pages while abroad
3. Backups - ever needed to push quickly some confidential file and didn't want to let Dropbox admins to look at it?
4. Sharing files - quickly and securely share files with your friends, family or just different computers.
5. Opening sessions - using SSH tunneling you could let your connection open even if you are behind several routing points
6. Long term jobs - periodically running some script (e.g. scrapping info from some website) or automating some repetitive task (e.g. backup).

Even though today out-of-box Linux distributions have quite good security, using all these technologies necessary brings some risk of attack against your server. Hence, it's crucial to know about vulnerabilities and try to defend yourself against possible attacks. I will try to give you examples of such cases and reason why you should apply according policy.

## Used software
As you could read in the header, as an operational system is chosen Linux. My working station is going to be Raspberry Pi with Arch Linux ARM distribution. Arch Linux is bleeding edge distribution which in nutshell means that we are going to use the most up-to-date technologies to this date (late 2015). Even though or examples are given for this specific Linux distribution it's good to mention that most of commands should be easily done even on other distributions. For example, package `systemd` is [used in all major distributions](https://en.wikipedia.org/wiki/Systemd#Adoption_and_reception) and hence you should be able to use it any where.

I am not going to go into much details around Linux environment, since I've already done so in [other tutorial](http://tutos.readthedocs.org/en/latest/). Some parts of this tutorial are going to overlap a bit (like domains and IPs) and in that case I will simply copy the text here with necessary updates. 

## What you should have before 
* installed Linux distribution of your choice with root access to the shell (direct by connecting peripherals or indirect using e.g. SSH)
* Internet connection. In this guide we are going to assume the most common setup for servers - you are connected by Ethernet to your router which is connected to wall socket.
* approximately 2 hours of time

## What are we going to cover
This guide is going to provide step-by-step solutions which cover:

1. Users, sandboxing their environment from others (and mainly from root's), storing passwords
2. General recommendations about installed software, controlling services, network settings, port forwarding
3. SSH and keys, SSH guard service, firewall using `iptables`, SSL
4. Entropy used for encrypting and its sources
5. Installing and securing web server `nginx`, sandboxing websites folders
