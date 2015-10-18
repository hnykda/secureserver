# Users on Linux
It's necessary to elaborate a bit how users and permissions work on Linux systems. 

The most important user present on every Linux distribution is the *root* user, sometimes called as a *superuser*. This guy is the master and 
can do basically everything. As a consequence of this
fact is extermaly important to not let anyone get root access to your server since then he can do everything. 
It's also quite dangerous work as a root all the time, since you can e.g. mistakenly erase your system partition
and so on. For that we are going to create new user which will get the permissions only when he asks for them. 
For this purpose we will use package called `sudo` (from *superuser do*).

Every file (and virtually *everything* is a file in Linux) specify permissions for owner, group and others.
There are three types of permissions: *read*, *write* and *execute*. More about permissions can be found out
e.g. [here](http://linuxcommand.org/lts0070.php). 

Before that, we need to explain some theory from elementary security and cryptography.

## Storing passwords
Users are authenticated by passwords. We've learned from past that storing passwords cannot be done in plain text,
since when someone steal this text, he automatically gets access to all accounts listed there. For that reason we
[hash](https://en.wikipedia.org/wiki/Hash_function) the passwords and store them in hashed form. Furthermore, the [salt](https://en.wikipedia.org/wiki/Salt_(cryptography)) is used while hashing. The file
with stored hashes is usually in `/etc/passwd`. 

For changing a password for a current user `passwd` is used. Type `passwd` and change the main *root* password to something reasonably strong.
You are not going to type this password a lot, since we are going to use other user for common system maintenance.

## Sandboxing
Here we just need to realize that you just don't want anyone to read some confidential file which don't belong to them
or execute some malicious script. For this purpose there is an idea of *sandboxing*. That means
that we separate environments of different users - he could touch only what he creates and cannot touch anything
which we don't explicitly allow. This scheme will be recurring once more when we will setting up e.g. web server.

# Creating a new user
Now we need to install first necessary package. You have to be logged as the root and have internet connection. Update
the system by `pacman -Syu` and then install `sudo` package by running `pacman -S sudo`. 

We now want to add regular user (think about it as a god who is creating humans). This can be done by: `useradd -m -G wheel Bob`. It will create user called “Bob”. There are also some other switches in command. `-m` is for creating bob’s *sandbox* for his files in `/home/bob` and `-G` adds him to the `wheel` group. 

Why? Remember that `sudo` can grant you superusers privileges. Every user who is in wheel group will have the ability to use `sudo`.  For that we need to 
edit `sudo` configuration file. Type: `visudo`, find this line: `# %wheel ALL=(ALL) ALL` and delete `#` character (for future reference, this means to “uncomment line”). 
It will look like this `%wheel ALL=(ALL) ALL`. Save and exit. We now need to set password for `bob`. Do it by typing `passwd bob`. From now you should be always working as a *bob* instead of *root*. When you'll want to do something what only *root* can do, e.g. `this_command`, you'll have to type `sudo this_command` and insert **bob**'s password (not the root one!). It's annoying, but save.

