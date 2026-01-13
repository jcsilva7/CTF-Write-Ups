# Bricks Heist Write Up

## Index

- [Room Info and Challenge Description](#thm-description-of-the-ctf)
- [Overview](#overview)
- [Challenge](#challenge)
  - [Enumeration](#enumeration)
  - [Web Enumeration](#web-enumeration)
  - [Vulnerability Identification](#vulnerability-identification)
  - [Initial Access](#initial-access)
  - [Post‑Exploitation & Investigation](#post-exploitation)
- [Conclusion](#conclusion)

## THM Description of the CTF

### [Room link](https://tryhackme.com/room/tryhack3mbricksheist)

Crack the code, command the exploit! Dive into the heart of the system with just an RCE CVE as your key.

Brick Press Media Co. was working on creating a brand-new web theme that represents a renowned wall using three million byte bricks. Agent Murphy comes with a streak of bad luck. And here we go again: the server is compromised, and they've lost access.

Can you hack back the server and identify what happened there?

## Overview

- Platform: TryHackMe  
- Room: Bricks Heist  
- Objective: Compromise the host, identify malicious activity, and trace attacker fingerprint  
- Attack vector: WordPress Bricks Builder RCE (CVE‑2024‑25600)

## Challenge

### Enumeration

Started by running a simple nmap command:

```bash
nmap -sC -Pn -T4 -vv bricks.thm
```

The command revealed 4 open ports:

```bash
Discovered open port 80/tcp on 10.82.156.161
956  Discovered open port 22/tcp on 10.82.156.161
957  Discovered open port 443/tcp on 10.82.156.161
958  Discovered open port 3306/tcp on 10.82.156.161
```

From these we can see http and https ports, which are expected, nothing irregular, also port 22 which is SSH, that could be interesting and the path to entering the system, and also 3306 which is not a well-known port, but after some quick research, it's easy to find that this is the default port for MySQL database management system, so, also some possibilities here, but for an easy ctf the others are more likely to help.

### Web enumeration

After opening the website, I investigated the HTML to check for any comments with important info, but found nothing of interest.

The website also had an image on the front page that I downloaded and used multiple tools (file, strings, binwalk, exiftool) to check if there is any hidden info on the image, but nothing was found.

Also used the gobuster tool to find any .txt or .php files of interest, but also found nothing.

### Vulnerability identification

Using Wappalizer, a Firefox extension that shows the technology behind websites, I noticed WordPress+PHP was found. The next step was to try and check the WordPress version, first in the headers, and then using a tool called webpack.
From its output I got WordPress version 6.5, I then searched online for RCE CVE related to that version and found nothing.

After that, with a bit more thought, the extension also showed that the website used a WordPress extension called `bricks` that, after searching, does have a RCE vulnerability for versions <=1.9.6 ([CVE-2024-25600](https://nvd.nist.gov/vuln/detail/CVE-2024-25600)).

### Initial access

Using an  [exploit available on GitHub](https://github.com/Chocapikk/CVE-2024-25600/tree/main), I got access to the system. A quick use of `whoami` and `id`, and I confirmed my user was `Apache`.

The first question of THM was the content of the hidden.txt in the web folder, which only required to use `ls -la` to check the current folder contents and then `cat` to reveal the flag.

### Post exploitation

After that, the goal was searching for a suspicious process.

I started with using `ps` but got nothing. Then using `systemctl | grep running` I got a rundown with the active system processes.

```bash
ubuntu.service loaded active running TRYHACK3M
```

This process stands out immediately. (the tryhackme mention helps).

After checking that service we can check there is a binary running with 2 processes:

```bash
/lib/NetworkManager/nm-inet-dialog
```

The binary name was the answer to the suspicious process answer.

The next question was the service name of this process which I already had. `ubuntu.service`.

Next the goal was to find the log file name of the miner instance (from this we can also deduce that this process is some sort of a miner for cryptocurrency).

I started with searching common folders like `/var/log` but found nothing.

Then I just kept it simple and search for the log in the folder of the binary. In the TryHackMe website it showed that the answer format was `****.****` and in that folder there was a file named `inet.conf`. After using `cat` on the file, the content aligned with a log file so that was indeed the correct answer.

The next step was searching for the wallet address of the miner instance. In the log file there was an ID of the process that seemed quite suspicious. After searching online and realizing that that ID was an hex of some string, I used cyberchef to try and decode the address. After using a recipe with HEX, the output was clearly a Base64 String (the `==` was the indicator). After adding another Base64 decode to the recipe, I got the final string.

The full string was not the correct answer, but looking at it more closely, the string seemed to have 2 similar parts starting with the same characters, after trying both as an answer, one of them was actually the wallet address.

Moving on to the final question, the goal was to discover the threat group associated with the transactions related to that wallet address (some OSINT, always nice). A quick search online revealed that that address was related to a cybercriminal group called _LockBit_, responsible for carrying out ransomware attacks since 2019 (and also inspecting the address we can see that that address alone moved 15.5 BTC, worth oer 1.4 million dollars at the time and 144M at the time of this write up).

## Conclusion

This was a simple CTF, the goal was to find the web stack of the web app in question, which was not hard since the name of the room pointed to what we are supposed to exploit, and use the exploit to gain access and find more info about the system.

Even though it is not the most complex challenge it is a decent challenge for beginners trying to enter the world of cybersecurity, more specifically pentesting/red teaming.
