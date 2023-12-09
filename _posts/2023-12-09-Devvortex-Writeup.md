---
layout: post
title: "HTB - Devvortex Writeup"
categories: writeup
tags: linux ctf
---

This writeup for the challenge Devvortex on Hackthebox is meant to give an overview of the challenge's solution without spoiling too much of the key details so you can still have fun while following it !

# 1. Initial enumeration
First and foremost, as usual for any challenge we can run a simple port scan using nmap:
![](/assets/writeups/Devvortex/Selection_146.png)

Given the port 80 is opened we can try to access this address from our browser. It is trying to redirect to devvortex.htb, so after adding it to our hosts file we land on the main page:
![](/assets/writeups/Devvortex/Selection_147.png)

This site doesn't provide much functionnality that might be exploited to gain access to a protected account, so we should continue the enumeration process using gobuster to discover subdomains if any is available:
![](/assets/writeups/Devvortex/Selection_114.png)

We found dev.devvortex.htb which lands us on another site:
![](/assets/writeups/Devvortex/Selection_112.png)

# 2. Foothold
Using gobuster in directory mode we discover some interesting pages, especially the /administrator which is a Joomla login page:
![](/assets/writeups/Devvortex/Selection_148.png)

![](/assets/writeups/Devvortex/Selection_149.png)

Also, trying to access the default README.txt allows us to retrieve the version of Joomla running on the site:
![](/assets/writeups/Devvortex/Selection_150.png)

Checking the known vulnerabilities for this version we find this CVE which happens to have a metasploit module for it:
![](/assets/writeups/Devvortex/Selection_118.png)

Using the metaploit module we get credentials to login on the admin pannel:
![](/assets/writeups/Devvortex/Selection_119.png)


# 3. Gaining access
Now that we have access to the admin panel of the site we can try to find a way to upload a PHP reverse shell on it. An easy way to do that is to find a file we are allowed to edit:
![](/assets/writeups/Devvortex/Selection_130.png)

This gives us access to a remote shell:

![](/assets/writeups/Devvortex/Selection_131.png)

From there we can attempt to login to the mysql database on localhost on try to find some relevent information to access an account with higher privileges. This gives us another username and a password hash:
![](/assets/writeups/Devvortex/Selection_135.png)

Using John we can crack this hash with the rockyou wordlist:
![](/assets/writeups/Devvortex/Selection_137.png)

This allows us to login on this account using ssh:
![](/assets/writeups/Devvortex/Selection_138.png)

We now have access to the user flag in the home directory.


# 4. Privilege escalation
The first step is to check if this user has access to commands with root privilege:
![](/assets/writeups/Devvortex/Selection_140.png)

We see that this user can run apport-cli with root privilege, we can retrive its version with -v:
![](/assets/writeups/Devvortex/Selection_142.png)

Searching online for known vulnerabilities we find that CVE-2023-1326 can be used for privilege escalation. Following the instruction to achieve this exploit we obtain root access on the machine:
![](/assets/writeups/Devvortex/Selection_145.png)

We can now change to root with the su command and get the flag from /root/root.txt.
Congratulations !