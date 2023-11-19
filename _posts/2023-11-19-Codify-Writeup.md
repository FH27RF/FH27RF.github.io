---
layout: post
title: "HTB - Codify Writeup"
categories: writeup
tags: linux easy
---

This writeup for the challenge Codify on Hackthebox is meant to give an overview of the challenge's solution without spoiling too much of the key details so you can still have fun while following it !

# 1. Initial enumeration
First and foremost, as usual for any challenge we can run a simple port scan using nmap:
![](/assets/writeups/Codify/Selection_110.png)

Given the port 80 is opened we can try to access this address from our browser. It is trying to redirect to codify.htb, so after adding it to our hosts file we land on the main page:
![](/assets/writeups/Codify/Selection_111.png)

# 2. Foothold
This site mainly consists of a sandbox that will run javascript code. This functionnality relies on the vm2 module which is vulnerable. After looking for an exploit I found a piece of code that allows us to run arbitrary system commands in the sandbox:
![](/assets/writeups/Codify/Selection_079.png)

This vulnerability can now be exploited to add our own ssh public key to the authorized list in /home/svc/.ssh/ which allows us to open an ssh session:
![](/assets/writeups/Codify/Selection_082.png)

# 3. Gaining access
From there we can look around, and there is an interesting file in the /var/www/contacts directory. tickets.db is a mysql database that we can directly open from this shell:
![](/assets/writeups/Codify/Selection_084.png)

This gives us a username and a hash we can now try to crack using John or Hashcat:
![](/assets/writeups/Codify/Selection_085.png)

We can now try to log in as joshua using ssh and retrieve the user flag from /home/joshua/user.txt !

# 4. Privilege escalation
The first step is to check if this user has access to commands with root privilege:
![](/assets/writeups/Codify/Selection_088.png)

We see that this user can run the following script with root privilege:
![](/assets/writeups/Codify/Selection_090.png)

This script's comparison method is vulnerable given it doesn't compare strings but is matching two expression, we can check that by running it and entering '*' as the password which will backup data.
We can now use this to brute force the password one character at a time. For instance if the password is Qwerty123, running the script with Q\* will be successful, then with Qw\* etc.

Crafting a small script with python we then obtain the root password:
![](/assets/writeups/Codify/Selection_079e.png)

We can now change to root with the su command and get the flag from /root/root.txt.
Congratulations !