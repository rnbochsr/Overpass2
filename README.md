# Overpass 2 - Hacked

*Overpass has been hacked! Can you analyse the attacker's actions and hack back in?*

> Bradley Lubow | rnbochsr, September 2022 

*My notes and solutions for the TryHackMe.com's Overpass 2 room.*

## Task 1 - Forensics - Analyze the PCAP

Overpass has been hacked! The SOC team (Paradox, congratulations on the promotion) noticed suspicious activity on a late night shift while looking at shibes, and managed to capture packets as the attack happened.

Can you work out how the attacker got in, and hack your way back into Overpass' production server?

Note: Although this room is a walkthrough, it expects familiarity with tools and Linux. I recommend learning basic Wireshark and completing (CC: Pentesting)[https://tryhackme.com/room/ccpentesting] and (Linux Fundamentals)[https://tryhackme.com/module/linux-fundamentals] as a bare minimum.

md5sum of PCAP file: 11c3b2e9221865580295bc662c35c6dc

I downloaded the task file and verified the md5sum. 
```bash
──(bradley㉿kali)-[~] md5sum overpass2.pcapng 
11c3b2e9221865580295bc662c35c6dc  overpass2.pcapng
```

Looks good. Now to see what they did, I'll look at the file in Wireshark. This should ber fun. I haven't done a lot in Wireshark, so this will be a good room to increase my skills with this program. 


### Wireshark Analysis

Looking thru the data in the PCAP file to find out how the attacker got in. The first thing I tried was using an `http` filter on the data. There was only one `POST` entry which made identifying the attacker's entry vector, and the answer to the first question pretty straightforward. 

*What was the URL of the page they used to upload a reverse shell?* 
`/d[REDACTED]t/`

Continuing the analysis of the PCAP file allowed me to locate lots of useful information. It took me a little longer that it should as I was having some trouble employing the best filters for the job. That is only because I haven't used Wireshark alot. 

I had to look at a number of uninteresting entries, but I was finally able to see the progression of the attackers. It was really quite interesting. You can see the methodolgy used, and even the typos that tripped them up now and again. I was able to find:
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

