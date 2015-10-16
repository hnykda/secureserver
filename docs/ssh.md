The most common way how to connect to your server is using [SSH](). 

We will open RPi to world and in that case we need to secure it a bit. Service, which takes care about SSH is called `sshd`. *Where* is it? It is run by `systemd`, so `systemctl status sshd` will show you more info. To reduce the brutal force attacks on root account (if was thousands of attempts every day on my little unimportant server), we will disable root login through SSH. 

## Tweaking SSH

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

and again restart `ssh`. We just disabled the ability to connect to your server with password. But how else to login in? Using [keys]()!

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
