+++
hasMath = false
date = "2017-03-30T20:01:49+02:00"
title = "How to safely SSH into your own computer"
draft = false
summary = "Allowing incoming SSH connections to your personal computer is something you want to avoid. If you have to do it, then do it in a safe way."
tags = [
  "tech"
]
showLogo = false
logo = ""
+++

I came across a situation where I had to `ssh` into my personal computer from a remote server. Most of the time, you don't want to do this. There's *botnets* and other *evil-doers* out there that try to break into easily accessible machines, and a password-protected SSH connection is easily broken into.   
*So don't do this unless you really need to!*

Okay, so you really need to. What do we need to do?

1. *Tell your router to forward incoming transmissions for your `SSH port` to your machine.*   
The specific procedure depends on your router and I'll leave it to you to find out how to do it. What's important, though, is that you should **change the default port**. Botnets, script kiddies and everybody else that wants to break into your system will try the default port `22` first. If that doesn't work, some might move on. Others might try more ports. Make them work for the right port, choose something random and large, in the `10000 to 64000` range. [This is a list of used ports, so pick one that's not on this list!](https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers)
+ *Start your SSH daemon*   
At least on my `arch linux` system, the daemon that listens for incoming connections is not enabled by default.   
You have two choices:   
1) `start` it with `sudo systemctl start sshd.service`. An instance will be running until you restart your PC. If you only occasionally want to access your machine from a remote server, then **this is the safer way**.   
2) `enable` it with `sudo systemctl enable sshd.service`. This will auto-start the daemon upon boot, too.
+ *Tell SSH to listen on your chosen port*   
Open up your **`/etc/ssh/sshd_config`** file. The idea here is that many options are present in the file and set to their default values, *and they're commented out*. So, to change them, *uncomment* them by removing the leading `#` and then you can change their default values.   
Fine the line for `#Port 22` and change it to `Port <your port number>`.
+ *Disable root login*   
For the sake of completeness, find the line that starts with `#PermitRootLogin` and change it to `PermitRootLogin no`.
+ *Disable password authentication*   
All passwords are evil if you have the possibility of using *public key authentication*. So find the line `#PasswordAuthentication yes` and change it to `PasswordAuthentication no`.
+ *Enable public key authentication*   
You kinda need one way of getting in, right? **This should be enabled by default** at the time of writing, but check anyway: `#PubkeyAuthentication yes` is the magic line. If you find it, leave it, it's the default. Or remove the `#`, whatever.
+ *Generate a public/private key pair*   
**On the remote server** generate a key pair with `ssh-keygen -t rsa -b 4096 -C "your_email@example.com"`. Follow the prompts, set a password.   
Now start an `ssh-agent`: `eval "$(ssh-agent -s)"`.   
And add the key to the agent, this way you don't have to type in your key's password every time you use it: `ssh-add ~/.ssh/<your private key file>`.
+ *Tell your system to allow this key*   
**On your home system** open **`~/.ssh/authorized_keys`** (create it if it doesn't exist) and add the content of your **remote system's `~/.ssh/<your public key file>`**. Just copy the entire content, it should be on one line and contain lots of random characters.
+ *Restrict this key to a particular IP address*   
Another nice security measure. In your **home system's `~/.ssh/authorized_keys`** (the one you just edited), prepend to the line from last step this: `from="1.2.3.4"` where you of course replace the IP with your remote server's IP.
+ *Install fail2ban*   
`fail2ban` is a cool tool that automatically blocks IP addresses that seem suspicious by trying too many times to get into your system unsuccessfully. This would be your last line of defense. All you need to do is to install and enable it. On `arch` it'd be: `sudo pacman -S fail2ban` and then `sudo systemctl enable fail2ban.service`.
+ *Create a new user with limited rights*   
Create a new group: `sudo groupadd sshusergroup`   
Add a new user: `sudo useradd -m -g sshusergroup -s /bin/bash sshuser`   
Restrict SSH to allow only this user: add the line `AllowUsers sshuser` to **`/etc/ssh/sshd_config`**
+ *Restart your daemon*   
To load all these settings, reload `sshd` with `sudo systemctl restart sshd.service`.

You could call this a very **paranoid approach**. Just the way I like it, because you reaaally don't want random people on your personal computer. I can't think of a way of breaking into this system via SSH anymore.   
Let's recap: The SSH daemon is not always running, only when you need it (if you didn't enable it). If it is, then it only allows key authentication, which is pretty safe. Also it only allows this if it comes from the single IP you specified. And if someone is still able to try different keys, then if he tries too many, he's banned by `fail2ban`. And if all security measures fail, then he can only login to the `sshuser` user, which is part of no groups to speak of, has *no sudo rights* and can't really do a thang outside of his home directory.

On the **remote server** you can now ssh into your machine with `ssh -p <your port> sshuser@<your IP>`.
