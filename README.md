# Overpass 2 - Hacked

*Overpass has been hacked! Can you analyse the attacker's actions and hack back in?*

>
> Bradley Lubow | rnbochsr, September 2022 
> My notes and solutions for the TryHackMe.com's Overpass 2 room. 
>

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


