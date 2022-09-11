# Overpass 2 - Hacked

*Overpass has been hacked! Can you analyse the attacker's actions and hack back in?*

> Bradley Lubow | rnbochsr, September 2022 

*My notes and solutions for the TryHackMe.com's Overpass 2 room.*

## Task 1 - Forensics - Analyze the PCAP

Overpass has been hacked! The SOC team (Paradox, congratulations on the promotion) noticed suspicious activity on a late night shift while looking at shibes, and managed to capture packets as the attack happened.

Can you work out how the attacker got in, and hack your way back into Overpass' production server?

Note: Although this room is a walkthrough, it expects familiarity with tools and Linux. I recommend learning basic Wireshark and completing [CC: Pentesting](https://tryhackme.com/room/ccpentesting) and [Linux Fundamentals](https://tryhackme.com/module/linux-fundamentals) as a bare minimum.

`md5sum` of PCAP file: `11c3b2e9221865580295bc662c35c6dc`

I downloaded the task file and verified the md5sum. 
```bash
──(bradley㉿kali)-[~] md5sum overpass2.pcapng 
11c3b2e9221865580295bc662c35c6dc  overpass2.pcapng
```

Looks good. Now to see what they did, I'll look at the file in Wireshark. This should be fun. I haven't done a lot in Wireshark, so this will be a good room to increase my skills with this program. 


### Wireshark Analysis

Looking thru the data in the PCAP file to find out how the attacker got in. The first thing I tried was using an `http` filter on the data. There was only one `POST` entry which made identifying the attacker's entry vector, and the answer to the first question pretty straightforward. 

*What was the URL of the page they used to upload a reverse shell?* 
`/d[REDACTED]t/`

Continuing the analysis of the PCAP file allowed me to locate lots of useful information. It took me a little longer that it should as I was having some trouble employing the best filters for the job. That is only because I haven't used Wireshark alot. 

I had to look at a number of uninteresting entries, but I was finally able to see the progression of the attackers. It was really quite interesting. You can see the methodolgy used, and even the typos that tripped them up now and again. Along the way I was able to get a better idea of how best to search for what I needed. The filters that proved most helpful were:
* `http`
* `tcp`
* Search terms for `PSH, ACK`

I was able to find:
* The uploading of the initial exploit
* The exploit contents
* The privilege escalation vector
* Files the attacker viewed
* The method of persistence
* And more

I haven't used Wireshark that much and I really enjoyed the process of following along after the fact to see what and how they hacked the system. 

*What payload did the attacker use to gain access?*
`<?php[REDACTED]?>`

*What password did the attacker use to privesc?*
`wh[REDACTED]nt`

*How did the attacker establish persistence?*
`ht[REDACTED]or`

To solve the last question in task 1 we have to try and crack the hashed passwords from the `etc/shadow` file. It is also buried in the PCAP file. Pulling out the hashes and feeding them thru Hashcat should tell us which ones are crackable. 

*Using the fasttrack wordlist, how many of the system passwords were crackable?*
[REDACTED] - The answer is 1 character long, so I redacted the entire thing. 


## Task 2 - Research - Analyze the code 

Now that you've found the code for the backdoor, it's time to analyse it.

The PCAP file gave us a lot of information. It included the location from where the backdoor was downloaded. Navigating a browser the the URL and you can see the source code for the hack. Everything is in cleartype, so there isn't any guesswork going on here. Reviewing it gives the answers to the following: 

__Answer the questions below__
*What's the default hash for the backdoor?*
`bd[REDACTED]e3`

*What's the hardcoded salt for the backdoor?*
`1c[REDACTED]05`

*What was the hash that the attacker used? - go back to the PCAP for this!*
`6d[REDACTED]ed`

This is buried in the PCAP file. Using the `tcp` filter and then searching for `PSH, ACK` narrowed things enough to find the entry where the atacker sent the hash they used. 

*Crack the hash using rockyou and a cracking tool of your choice. What's the password?*
`no[REDACTED]16`

I used hashcat again and easily cracked the hash. Now comes the real fun, getting back in and retaking the machine. I started the victim machine and now we'll see what we have. 


## Task 3 - Attack - Get back in! 

Using the information you've found previously, hack your way back in!

Trying to ssh into the target machine using the password for `james` didn't work. I found that surprising, but not to worry yet. Let's begin with some initial recon to see if they left any holes we can slip through. 

### Recon

*The attacker defaced the website. What message did they leave as a heading?*
`H4ck3d by CooctusClan`

#### NMAP Scan
```bash
# Nmap 7.80 scan initiated Sat Sep 10 20:25:55 2022 as: nmap -p- -T4 -v -sC -sV -oN nmap.scan overpass2
Nmap scan report for overpass2 (10.10.97.212)
Host is up (0.00070s latency).
Not shown: 65532 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 e4:3a:be:ed:ff:a7:02:d2:6a:d6:d0:bb:7f:38:5e:cb (RSA)
|   256 fc:6f:22:c2:13:4f:9c:62:4f:90:c9:3a:7e:77:d6:d4 (ECDSA)
|_  256 15:fd:40:0a:65:59:a9:b5:0e:57:1b:23:0a:96:63:05 (ED25519)
80/tcp   open  http    Apache httpd 2.4.29 ((Ubuntu))
| http-methods: 
|_  Supported Methods: HEAD GET POST OPTIONS
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: LOL Hacked
2222/tcp open  ssh     OpenSSH 8.2p1 Debian 4 (protocol 2.0)
| ssh-hostkey: 
|_  2048 a2:a6:d2:18:79:e3:b0:20:a2:4f:aa:b6:ac:2e:6b:f2 (RSA)
MAC Address: 02:D5:39:B3:58:55 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Sep 10 20:26:35 2022 -- 1 IP address (1 host up) scanned in 40.27 seconds
```

They have `OpenSSH` running on port 2222. Let's see if any of the creds I have so far will work on that port. 

Using the `james` ID, I first tried the initial password we had for him. That didn't work. I then tried the cracked backdoor password and I got in. Moving into his home directory reveals the user flag. 

```bash
root@kali:~# ssh -p 2222 james@overpass2
The authenticity of host '[overpass2]:2222 ([10.10.97.212]:2222)' can't be established.
RSA key fingerprint is SHA256:z0OyQNW5sa3rr6mR7yDMo1avzRRPcapaYwOxjttuZ58.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[overpass2]:2222,[10.10.97.212]:2222' (RSA) to the list of known hosts.
james@overpass2's password: 
Permission denied, please try again.
james@overpass2's password: 
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

james@overpass-production:/home/james/ssh-backdoor$ ls
README.md  backdoor.service  cooctus.png  id_rsa.pub  main.go
backdoor   build.sh          id_rsa       index.html  setup.sh
james@overpass-production:/home/james/ssh-backdoor$ cd ..
james@overpass-production:/home/james$ ls
ssh-backdoor  user.txt  www
james@overpass-production:/home/james$ cat user.txt
thm{d1[REDACTED]67}
```

*What's the user flag?*
`thm{d1[REDACTED]67}`

### Privilege Escalation

Now we have to find the escalation vector they used, or some other exploit, to retake the machine. 

First let's see if james is still in the sudo group. He is, but none of the credentials I have allow me to switch to root. None of the other login credentials will work either. I need to look around further. 

The hint made reference to the hackers leaving something behind so that they can easily escalate without a password. I looked back in that backdoor directory, but nothing. I found it in James' home directory as a hidden file. A script that used an `suid` bit to retain the root privileges. running it and I was root. moving into the root user home directory and getting the `root.txt` flag. 

*What's the root flag?*
`thm{d5[REDACTED]4d}`


## Final Thoughts
This was a fun challenge. I got to work on my Wireshark skills and a very entry level king of the hill type thing. 
