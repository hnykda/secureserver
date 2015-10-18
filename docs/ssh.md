The most common way how to connect to your server is using [SSH](https://en.wikipedia.org/wiki/Secure_Shell). 

We will open RPi to world and in that case we need to secure it a bit. Service, which takes care about SSH is called `sshd`. *Where* is it? It is run by `systemd`, so `systemctl status sshd` will show you more info. To reduce the brutal force attacks on root account (if was thousands of attempts every day on my little unimportant server), we will disable root login through SSH. 

# Getting a proper randomness
A lot of security algorithms relies on having a good source of (pseudo)randomness. Why? Take for example the password file from previous chapter. 
If you would generate the salt in some predictable way, an attacker would have much easier work to guess the passwords. 

There are two special Linux *devices* which you can use for getting random data. Those random data are mostly generated from user interaction with e.g. mouse, keyboard or screen. 
That is the problem on headless Linux server as we can have, since they do not have such interfaces. Because of that, relaying on the basic 
source of randomness is insufficient. These devices are in `/dev/random` and `/dev/urandom`. The latter differs from the former in that manner that it will generate
*random* data even though there is not enough entropy for ensuring sufficient randomness. Hence, use `/dev/random/` instead.

If you don't want to invest to additional hardware which provide very high and *true* randomness by buying e.g. [OneRNG](http://onerng.info/), there is another solution 
which will definitely be enough for 99% of users. That solution is `havege` (respective it's daemon `haveged`). This is a clever algorithm which generates randomness
based on time which processor spends during computing some instructions. 

Installing and activating `haveged` cannot be easier. Go for:
```
$ pacman -S haveged
$ systemctl enable haveged
$ systemctl start haveged
```
and your `/dev/random` will be fed by `haveged` data since now.

# SSH

If you haven't installed SSH yet, do it by `pacman -S ssh`.

Open the file `/etc/ssh/sshd_config` and edit/uncomment or add these lines as follows (left the rest untouched):

```
Port 1234
PermitRootLogin no
PubkeyAuthentication yes
```

Restart `ssh` service by `systemctl restart sshd`. Since now, you cannot login as a root by ssh and that's good. Also - we changed the port of ssh to `1234` (you are advised to change this number to something different of course). Think about *port* as a tunnel, which is used by some application. There are about 60 thousands of them and you can choose whichever you want. Default port for SSH is 22 and that is the port which is most widely abused by brutal force attacks. Changing it to something from this range just makes it several tenth thousands harder to guess the access... 

Since now, using only `ssh bob@ipadress` to connect to server is not enough. You will have to add port which should be used. `ssh -p 1234 bob@ip.address` will do that for you.

Improving the security of SSH can be done basically by two approaches depending on your needs.

## SSHguard
Shortly, `sshguard` is going to block recurring attempts of login and in case they are failing, it denies the access entirely for some specific time. Install it by `pacman -S sshguard`. For proper usage, we need to tweak Linux firewall app called `iptables`. 

### iptables
`iptables` are really just another layer of security often called as a firewall. It just specified what connections can come in and out. We need only simple configuration which can be done quite easily.

```
$ iptables -F
$ iptables -X
$ iptables -P INPUT ACCEPT
$ iptables -P FORWARD ACCEPT
$ iptables -P OUTPUT ACCEPT
$ iptables -N sshguard
$ iptables -A INPUT -j sshguard 
$ iptables-save > /etc/iptables/iptables.rules    
```

Personally I stopped to use `sshguard`, since just changing port what SSH use was enough to reduce uninvited connections. Furthermore, there is actually a safer method using only SSH keys.

## SSH keys
This is the best you can do from the point of security. Open `/etc/ssh/sshd_config` and edit these lines as follows:

```
ChallengeResponseAuthentication no
PasswordAuthentication no
UsePAM no
```

and again restart `ssh`. We just disabled the ability to connect to your server with password. But how else to login in? Using [keys](https://wiki.archlinux.org/index.php/SSH_keys)!

### Creating a key pair
We now need to create a key pair. Two files will with public and private key will be created. Let's do it by this command: `ssh-keygen -t ed25519 -f ~/.ssh/bobs_key` (left passphrase empty).  
One of created files is `~/.ssh/bobs_key.pub` and that is the one (public key) you want to transfer to client - the computer **from** you are going to connect to the server. 
The second one is private key and you should never ever show any of the keys to anyone under any circumstances. It would basically imply your vulnerability, since anyone who possess public key and the username can connect to your server.

Copy the content of the public key to the clients ssh folder. Now you need to specify this file to connect by `-i` switch following the filename, e.g. `ssh -p 1234 -i ~/.ssh/bobs_key.pub bob@yourrpiaddress`. 

## Convenient settings
It is annoying still typing same username, port and filepath to the key when we want to connect to our server. On PC from which we are connecting (no RPi), edit `~/.ssh/config` to this:

```
Host my_superpc
  HostName ipaddressofRPi
  IdentityFile /home/yourusername/.ssh/name_of_identityfile
  User bob
  port 1234
```

since now, when you want to connect to RPi, you can just type `ssh my_superpc` and it will take care about rest. Quite clever, isn't it?


# Opening the server to the world
Because we now already know what port is and we have an application which uses them, we can take leverage of using them for security purposes. You have already seen that when we changed the default port from SSH. In the classic scenario when you have your server plugged into your router you wouldn't have to worry anyway, because the router wouldn't allow anyone to get to your server. So how to achieve this?

## Port forwarding
You must first get your public IP address. That can be found e.g. [here](http://www.whatsmyip.org/). There is another caveat - you might not have your own public IP, but rather some local from your provider and your provider has the one which is public. There is nothing you can do in to make your server visible from outer world unfortunately, except trying to negotiate with your provider. 

In case you are not so unlucky, you have your own public IP which you can use to connect to your server. You will make a request to your IP on specific port, e.g. using `ssh` to connect on port `1234`. This will get into the router first and router will decide where should be the request sent next. If the port isn't explicitly mentioned in the router *lookup* table, the request is thrown away. That is good - we don't want let the attacker give opportunity to do anything with our inner network. So enable only those ports which are necessary. 

How to do a port forwarding differs based on the router and it's firmware. But you need to get into your router and change the settings. How to do that you might find e.g. [here](http://portforward.com/) or use google for your router type.

To get into your router you need to set up port forwarding from some chosen port to the port you have chosen before for your SSH daemon (`1234` in my case). You can choose the same even for the outer port, but you can choose different for that (let's say `8765`), so your setting should be something like `8765 -> server.local.ip.address on port 1234`. This will tell router to redirect requests on port `1234` to your server and specific port on that server. 

Now you should be able to connect to your server using the port `8765`, username `bob` and your public IP address. The syntax is exactly the same: `ssh -p 8765 -i ~/.ssh/bobs_key.pub bob@YOURPUBLICIPADDRESS`.
